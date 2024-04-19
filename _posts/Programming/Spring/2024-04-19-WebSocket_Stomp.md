---
title: "[Spring Boot] WebSocket + STOMP 실시간 채팅 간단하게 구현하기!"
excerpt: "[Spring Boot] WebSocket + STOMP 실시간 채팅 간단하게 구현하기!"

categories:
  - spring
tags:
  - [spring, websocket, stomp]

toc: true
toc_sticky: true

date: 2024-04-20
last_modified_at: 2024-04-20
---

# 개요

 [이전 글](https://min9805.github.io/spring/WebSocket/) 에서는 WebSocket 으로 간단한 채팅 서버를 구현했다. 이것을 STOMP 를 사용해서 구현해보도록 하겠다. 

 Spring 에서 Websockt 의존성을 받아오면 Spring Messaging 이라는 의존성도 함께 추가된다.

 이를 이해하기 위해서는 STOMP 를 이해해야한다. 


# STOMP

- Simple Text Oriented Messaging Protocol

STOPM 는 메세지 브로커를 활용해 쉽게 메세지를 주고받을 수 있는 프로토콜이다.

pub-sub 이라는 발행-구독 형태를 사용해 메세지를 주고받을 수 있다.

웹 소켓 위에 얹어 함께 사용할 수 있는 하위(서브) 프로토콜이다!

## 데이터 형식

WebSocket 을 사용할 때는 메세지를 주고받는 형식은 따로 정해져있지 않다. 

반면 STOMP 에서는 커맨드, 헤더, 바디의 구조로 데이터 형식을 지정한다.

```
COMMAND
header1:value1
header2:value2
Body^@
```

- COMMAND
    - SEND, SUBSCRIBE 를 지시할 수 있다.
- HEADER 
    - 기존 websocket 으로 표현 불가능한 헤더를 작성할 수 있다. 
- BODY
    - 실제 데이터가 담기는 부분이다.

이는 실제 데이터 예시를 살펴보면 이해가 빠르다

```
SEND
destination:/pub/chat
content-type:application/json

{"chatRoomId":1, "type":"TALK", "sender":"UserA", "message":"Hello world!"}
^@

```

위 데이터는 UserA 가 1번 채팅방에 메세지를 보내는 데이터 예시이다.

- COMMAND 
    - SEND 로 메세지 전송을 의미한다.
- HEADER
    - 메세지를 보낼 destination, type 이 지정되어있다.
- BODY
    - 실제 메세지가 담고있는 정보가 존재한다.

## PUB-SUB

![image](https://github.com/min9805/SpringFrameWork/assets/56664567/153ca740-fddb-4aa0-a0c6-95a8ff5678d3)

STOMP 의 구조는 위와 같다.

발신자는 a 라는 토픽에 메세지를 보내고, 구독자들은 a 라는 토픽을 구독하고 있다 가정한다.

발신자는 destination 을 "/topic/a" 로 설정해 메세지 브로커를 거쳐 구독자들에게 바로 메세지를 보낼 수 있다.

발신자가 서버 내에서 임의의 처리 및 가공이 필요하다면 destination 을 "/app/a" 로 설정해 메세지 가공 후 메세지 브로커에게 이를 전달할 수 있다. 

# 구현

## 1. WebSocketBrokerConfig

```
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketBrokerConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // /ws-stomp로 연결하는 endpoint를 생성하고, CORS 허용
        registry
                .addEndpoint("/ws-stomp")
                .setAllowedOrigins("*");
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        // /pub로 시작되는 메시지가 message-handling methods로 라우팅 되어야 한다.
        registry.setApplicationDestinationPrefixes("/pub");
        // /sub, /topic, /queue 로 시작되는 메시지가 메시지 브로커로 라우팅 되어야 한다.
        registry.enableSimpleBroker("/sub", "/topic", "/queue");
    }
}

```

STOMP 구현은 WebSocket 보다도 간편하게 할 수 있다. WebSocketMessageBrokerConfigurer 를 구현하고 두 가지 설정만 하면 된다.

- registerStompEndpoints
    - /ws-stomp로 연결하는 endpoint를 생성하고, CORS 허용한다.
    - WebSocket 과 거의 동일하다.

- configureMessageBroker
    - setApplicationDestinationPrefixes
        - /pub로 시작되는 메시지가 message-handling methods로 라우팅 되도록 지정한다.
        - 메세지 도착의 prefix 를 지정하는 것이다.
    - enableSimpleBroker
        - /sub, /topic, /queue 로 시작되는 메시지가 메시지 브로커로 라우팅 되도록 지정한다. 
        - 쉽게 말해 메시지 브로커가 메세지를 전달할 수 있는 경로이다.

## 2. Controller

```
@Controller
@RequiredArgsConstructor
@Slf4j
public class StompController {

    @MessageMapping("/chat/{chatRoomId}")
    @SendTo("/sub/chat/{chatRoomId}")
    public String message(
            @DestinationVariable Long chatRoomId,
            @Payload ChatDto request) {
        log.info("chatRoomId: {}, message: {}", chatRoomId, request.getMessage());

        return request.getMessage();
    }
}

@Getter
public class ChatDto {

    private Long chatRoomId;
    private String sender;
    private String message;
}
```

Spring 에서는 Stomp 를 매우 쉽게 구현할 수 있다. 

@MessageMapping 은 해당 경로로 들어오는 메세지들에 대해 동작한다는 뜻이다. 이때 앞서 설정한 prefix 가 존재하기에 "/pub/chat/1" 등의 경로로 들어와야 동작한다.

경로에 존재하는 파라미터는 @DestinationVariable 을 사용해 매핑할 수 있다.

@SendTo 는 말 그대로 메서드의 반환값을 해당 경로로 전달해준다는 뜻이다.

들어온 메세지는 @Payload 로 메세지만 매핑해 원하는 처리 후 반환할 수 있다. 

## 실행

![image](https://github.com/min9805/SpringFrameWork/assets/56664567/0f4bb728-8a40-4d78-aea0-c8f1efafabf3)

발행자 / 2번 구독자 / 1번 구독자 가 존재한다.
발행자가 1번 채팅방 경로로 메세지를 보낼 때 1번 구독자가 받는 것을 확인할 수 있다. 

![image](https://github.com/min9805/SpringFrameWork/assets/56664567/3a96c4f6-231c-4b9b-8ae9-72ee9f1566b5)

2번 경로에 대해서도 동일하게 동작한다.

APIC 에서는 연결과 동시에 구독을 지정해주어야해서 여러 구독을 만들어낼 수 없었다. 이대로 끝내기에는 구독과 연결 과정이 불명확하므로 페이로드를 자세히 열어보자.

# 인터셉터

## 1. Interceptor

```
@Component
@Slf4j
public class StompHandler implements ChannelInterceptor {

    @Override
    public void postSend(Message<?> message, MessageChannel channel, boolean sent) {
        StompHeaderAccessor accessor = StompHeaderAccessor.wrap(message);
        String sessionId = accessor.getSessionId();

        switch (Objects.requireNonNull(accessor.getCommand())) {
            case CONNECT -> log.info("CONNECT: " + message);
            case CONNECTED -> log.info("CONNECTED: " + message);
            case DISCONNECT -> log.info("DISCONNECT: " + message);
            case SUBSCRIBE -> log.info("SUBSCRIBE: " + message);
            case UNSUBSCRIBE -> log.info("UNSUBSCRIBE: " + sessionId);
            case SEND -> log.info("SEND: " + sessionId);
            case MESSAGE -> log.info("MESSAGE: " + sessionId);
            case ERROR -> log.info("ERROR: " + sessionId);
            default -> log.info("UNKNOWN: " + sessionId);
        }

    }
}
```

메세지가 도착했을 때 이를 열어보고 필요하다면 전처리를 위해 인터셉터를 구성했다. 

```
    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(stompHandler);
    }
```

해당 인터셉터를 config 에서 등록해주면 끝이다.

## 실행

![image](https://github.com/min9805/SpringFrameWork/assets/56664567/1902b50a-c3e9-4da5-aa4b-6dc97dd22710)

실제 실행 시 APIC 에서 연결을 진행하면 연결과 동시에 구독이 진행되는 것을 로그로 파악할 수 있었다. 

# 결론

아주 간단하게 Spring 에서 STOMP 구현이 가능하다. 이를 활용해 실시간 서비스 구현에 사용할 수 있다. 마지막으로 STOMP 의 활용 방안과 장점에 대해 정리해보자.

## STOMP 장점

- 정해진 컨벤션 (데이터 구조.. 커맨드/헤더/바디)
- 외부 Messaging Queue 사용 가능
- Spring Security

### 외부 Messaging Queue

위 코드에서 메세지 브로커, 큐는 메모리 상에 존재한다. 이는 다수의 서버를 동작시킬 때 문제가 발생할 수 있는데, A 를 구독하는 3명의 사용자의 메세지 큐가 각각의 서버에 존재한다면 발행자가 요청한 서버에서만 메세지가 전달될 가능성이 있다. 

![image](https://github.com/min9805/SpringFrameWork/assets/56664567/66eee350-c12b-4829-af39-286fb190d7fe)

인메모리 시스템의 위험성 등의 이유로 RabbitMQ, Kafka 등 외부 메세지 큐를 사용할 수 있다. 

### Spring Security

위에서 configureClientInboundChannel 를 통해 설정한 인터셉터를 사용해 JWT 등 인증 과정도 처리할 수 있다. 


[Github Code](https://github.com/min9805/SpringFrameWork/tree/master/WebSocket_Chat_STOMP)


# 참고

[우아한테크 - 10분 테크톡](https://www.youtube.com/watch?v=rvss-_t6gzg&t=80s)
[Spring Websocket & STOMP](https://brunch.co.kr/@springboot/695)