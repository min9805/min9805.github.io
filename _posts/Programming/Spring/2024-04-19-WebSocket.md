---
title: "[Spring Boot] WebSocket 실시간 채팅 간단하게 구현하기! "
excerpt: "[Spring Boot] WebSocket 실시간 채팅 간단하게 구현하기!"

categories:
  - spring
tags:
  - [spring, websocket]

toc: true
toc_sticky: true

date: 2024-04-19
last_modified_at: 2024-04-19
---

# 개요

 채팅 기능을 구현하기 위해 WebSocket 에 대한 공부를 시작했다. 
 웹소켓을 왜 사용하는 지, 동작부터 구현까지 정리해보도록 하겠다.

## HTTP vs WebSocket 

가장 일반적으로 서버와의 통신은 HTTP 를 통해 이루어진다.
하지만 이 경우 서버는 요청이 오지 않으면 응답을 줄 수 없는 치명적인 단점이 존재한다.
다시말해 채팅 혹은 주식 가격 등 실시간성으로 변하는 데이터를 클라이언트가 확인하기 위해서는 계속해서 HTTP 요청을 보내고 받아야한다.


당연하게도 클라이언트가 매번 똑같은 요청을 보내고 있는 것은 비효율적이다. 이를 위해 2 가지 해결책이 존재한다.

1. Server-Sent Evnet
2. WebSocket


SSE(Server-Sent Event) 는 단방향 데이터 통신이다. HTTP 프로토콜을 사용하며, 클라이언트는 데이터 수신만 가능하다.


반면 WebSocket 은 양방향으로 데이터를 주고 받을 수 있다는 장점이 있다. 


![image](https://github.com/min9805/SpringFrameWork/assets/56664567/aabfa0c2-8253-4ddb-9840-01d2d48bbe53)

HTTP 와 웹소켓의 차이가 정리되어있다. 
가장 큰 차이는 웹소켓이 연결을 유지한다는 점이다.

![image](https://github.com/min9805/SpringFrameWork/assets/56664567/4ffeccb9-d6dd-414c-b6b4-4484e8a83970)

HTTP 와 웹소켓은 주고받는 데이터에서도 차이가 존재한다.

HTTP 는 매번 HTTP 요청을 주고받기에 헤더 등 모든 정보를 계속해서 주고받는다.

반면 웹소켓은 처음 핸드쉐이크를 통해 연결을 생성하면 이후 필요한 메세지만 주고받기에 주고받는 데이터의 양에서 차이가 많이 난다. 

# 웹소켓 구현

## 1. WebSocketConfig

```
@Configuration
@EnableWebSocket
@RequiredArgsConstructor
public class WebSocketConfig implements WebSocketConfigurer {

    private final WebSocketHandler webSocketHandler;

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry
                // /ws/conn 경로로 WebSocket 연결을 허용
                .addHandler(webSocketHandler, "/ws/conn")
                // CORS 허용
                .setAllowedOrigins("*");
    }
}

```

Spring 에서는 간단하게 WebSocketConfig 를 구성할 수 있다.

- .addHandler(webSocketHandler, "/ws/chat")

    - webSocketHandler 핸들러를 사용한다.
    - endpoint = "/ws/conn" 으로 설정한다.
    - ws://localhost:8080/ws/conn 으로 웹소켓 연결이 가능하다.
-  .setAllowedOrigins("*")
    - CORS 허용 설정이다. 

## 2. WebSocketHandler

```
@Slf4j
@Component
@RequiredArgsConstructor
public class WebSocketChatHandler extends TextWebSocketHandler {
    private final ObjectMapper mapper;

    // 소켓 세션을 저장할 Set
    private final Set<WebSocketSession> sessions = new HashSet<>();

    // 소켓 연결 확인
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        // TODO Auto-generated method stub
        log.info("{} 연결됨", session.getId());
        sessions.add(session);
        session.sendMessage(new TextMessage("WebSocket 연결 완료"));
    }

    // 소켓 메세지 처리
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        String payload = message.getPayload();
        log.info("payload {}", payload);

        for (WebSocketSession s : sessions){
            s.sendMessage(new TextMessage(payload));
        }
    }

    // 소켓 연결 종료
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        // TODO Auto-generated method stub
        log.info("{} 연결 끊김", session.getId());
        sessions.remove(session);
        session.sendMessage(new TextMessage("WebSocket 연결 종료"));
    }
}
```

TextWebSocketHandler 를 상속받으며 연결, 메세지 처리, 연결 종료 3가지 메서드를 오버라이딩해야한다. 소켓 통신은 서버와 클라이언트가 1:N 으로 연결을 맺을 수 있다. 또한 여기서의 세션은 흔히 알고 있는 세션과 달리 WebSocket 의 연결 정보를 담고 있는 객체라 보면 된다. 

- afterConnectionEstablished
    - 소켓의 연결부분을 처리하며 소켓 세션에 현재 세션을 추가한다.
- handleTextMessage
    - 실제 메세지를 처리하는 부분이다. 
    - payload 를 받아와 sessions 에 저장된 세션에 모두 sendMessage 를 통해 전달한다.
- afterConnectionClosed
    - 연결 해제가 요청될 경우 sessions 에서 해당 세션을 제거한다. 

![image](https://github.com/min9805/SpringFrameWork/assets/56664567/2656fcf4-5241-46ae-8e39-fa31848de05b)
![image](https://github.com/min9805/SpringFrameWork/assets/56664567/a7656826-f786-4f0a-8112-7725a7233bec)

APIC 툴을 사용해 웹소켓 통신을 테스트해보면 실제 잘 동작하는 것을 확인할 수 있다. 

하지만 이런 채팅은 웹소켓 연결을 가진 사용자 모두에게 메세지를 계속해서 뿌리기 때문에 1:1 채팅을 구현할 수 없다.

# 채팅방 구현

## 1. Message DTO


```
@Builder
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
public class ChatMessageDto {
    // 메시지  타입 : 입장, 채팅, 퇴장
    public enum MessageType{
        JOIN, TALK, LEAVE
    }

    private MessageType messageType; // 메시지 타입
    private Long chatRoomId; // 방번호
    private String message; // 메시지
}
```

우선 메세지 DTO 를 생성한다. 

채팅방을 유지하고 DTO 를 통해 입장, 채팅, 퇴장을 모두 관리할 것이기에 MessageType 을 위와 같이 설정한다.

메세지를 보낼 때에는 방 번호와 보낼 메세지도 필요하다. 

## 2. WebSocketHandler

```
@Slf4j
@Component
@RequiredArgsConstructor
public class WebSocketChatHandler extends TextWebSocketHandler {
    private final ObjectMapper mapper;

    // 소켓 세션을 저장할 Set
    private final Set<WebSocketSession> sessions = new HashSet<>();

    // 채팅방 id와 소켓 세션을 저장할 Map
    private final Map<Long,Set<WebSocketSession>> chatRoomSessionMap = new HashMap<>();

    // 소켓 연결 확인
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        // TODO Auto-generated method stub
        log.info("{} 연결됨", session.getId());
        sessions.add(session);
        session.sendMessage(new TextMessage("WebSocket 연결 완료"));
    }

    // 소켓 메세지 처리
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        String payload = message.getPayload();
        log.info("payload {}", payload);

        // 클라이언트로부터 받은 메세지를 ChatMessageDto로 변환
        ChatMessageDto chatMessageDto = mapper.readValue(payload, ChatMessageDto.class);
        log.info("session {}", chatMessageDto.toString());

        // 메세지 타입에 따라 분기
        if(chatMessageDto.getMessageType().equals(ChatMessageDto.MessageType.JOIN)){
            // 입장 메세지
            chatRoomSessionMap.computeIfAbsent(chatMessageDto.getChatRoomId(), s -> new HashSet<>()).add(session);
            chatMessageDto.setMessage("님이 입장하셨습니다.");
        }
        else if(chatMessageDto.getMessageType().equals(ChatMessageDto.MessageType.LEAVE)){
            // 퇴장 메세지
            chatRoomSessionMap.get(chatMessageDto.getChatRoomId()).remove(session);
            chatMessageDto.setMessage("님이 퇴장하셨습니다.");
        }

        // 채팅 메세지 전송
        for(WebSocketSession webSocketSession : chatRoomSessionMap.get(chatMessageDto.getChatRoomId())){
            webSocketSession.sendMessage(new TextMessage(mapper.writeValueAsString(chatMessageDto)));
        }
    }

    // 소켓 연결 종료
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        // TODO Auto-generated method stub
        log.info("{} 연결 끊김", session.getId());
        sessions.remove(session);
        session.sendMessage(new TextMessage("WebSocket 연결 종료"));
    }
}
```

 추가된 코드는 채팅방 id와 소켓 세션을 저장할 Map 이다.

 여기에 채팅방 별로 연결되어있는 세션이 저장될 것이다.

 - handleTextMessage
    - 클라이언트로부터 받은 메세지를 ChatMessageDto로 변환한다.
    - 메세지 타입에 따라 분기한다.
        - JOIN 일 경우 채팅방이 존재하면 세션을 추가하고 존재하지 않으면 새로 만들어 추가한다. 
        - LEAVE 일 경우 채팅방에서 세션을 삭제한다. 
    - 이후 채팅 메세지를 채팅방에 속한 세션들에게만 전송한다.

위 과정을 통해 채팅방 입장, 대화, 퇴장을 간단하게 구현하였다.
 
 ![image](https://github.com/min9805/SpringFrameWork/assets/56664567/2748ae52-c72f-4a74-8ad0-18dd5f74b524)

사용자 A, B 는 1번 채팅방, 사용자 C 는 2번 채팅방에 입장한 상황이다.

 ![image](https://github.com/min9805/SpringFrameWork/assets/56664567/5d25aec6-2ce1-4104-bdc4-b37837f6f89e)

1번 채팅방에 들어간 사용자 A 가 대화 메세지를 전송했다. 2번 채팅방에 들어가있는 사용자 C 는 메세지를 못받는 상황이다.

![image](https://github.com/min9805/SpringFrameWork/assets/56664567/b734021c-4c68-421b-ae30-01e7b14e3dec)

1번 채팅방에 들어간 사용자 B 가 퇴장하는 상황이다. A 에게 퇴장 메세지가 잘 전달되었다.

![image](https://github.com/min9805/SpringFrameWork/assets/56664567/df9fc700-c794-43ba-8229-10dc5eb95117)

사용자 B 가 2번 채팅방에 입장하였다. 사용자 C 에게 메세지가 잘 전달된다.

![image](https://github.com/min9805/SpringFrameWork/assets/56664567/1718a3d5-97ca-4c07-9f72-ab5396f08b41)

사용자 B 가 1번 채팅방에도 입장하여 1, 2 번 채팅방에 존재하는 상황이다.

![image](https://github.com/min9805/SpringFrameWork/assets/56664567/a9340a9d-fd3a-4f62-81fc-9701fc8b992c)

사용자 B 가 2번 채팅방에 대화를 전송한다. DTO 에 방번호가 존재하기에 해당 B 는 1, 2 번 방에 속하더라도 2번만 특정해서 메세지를 보낼 수 있다. 



# 결론

아주 간단하게 Websocket 구현이 가능하다. 하지만 여기서도 계속해서 메세지 이외에 함께 보내는 데이터가 꽤 있다. 이는 STOMP 를 통해 해결 가능하다. 다음에는 STOMP 를 알아보도록 하자.


[Github Code](https://github.com/min9805/SpringFrameWork/tree/master/WebSocket_Chat)


# 참고

[우아한테크 - 10분 테크톡](https://www.youtube.com/watch?v=rvss-_t6gzg&t=80s)