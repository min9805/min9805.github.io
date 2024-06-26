---
title: "[JAVA] Spring 없이 웹 서버 구축! Spring Boot 웹 서버의 밑바닥 살펴보기"
excerpt: "[JAVA] Spring 없이 웹 서버 구축! Spring Boot 웹 서버의 밑바닥 살펴보기"

categories:
  - java
tags:
  - [JAVA, Webserver]

toc: true
toc_sticky: true

date: 2024-07-02
last_modified_at: 2024-07-02
---

# 개요

Spring 프레임워크 없이 웹 서버를 구현해보고 실제 Spring Boot 에서 사용하는 방법과 비교해보고자 한다. 

# 웹 서버 Without Spring

실제 웹 서버를 Spring 없이 구현해보자.

```
        try (ServerSocket listenSocket = new ServerSocket(port)) {
            logger.info("Web Application Server started {} port.", port);

            // 클라이언트가 연결될때까지 대기한다.
            Socket connection;
            while ((connection = listenSocket.accept()) != null) {
                Thread thread = new Thread(new RequestHandler(connection));
                thread.start();
            }
        }
```

ServerSocket 을 사용해 실제 서버를 열어놓는다. 포트는 8080 을 그대로 사용하였다.
한 번 웹 서버가 시작되고 난 이후로는 계속해서 Socket 연결을 요청받는다.

Socket 연결이 완성되면 각 커넥션마다 쓰레드를 생성하고, Runnable 을 구현한 RequestHandler 를 통해 요청을 처리하게된다.

위 과정을 통해 아주 간단하게 Socket 연결을 생성하고 처리하는 웹 서버가 완성된다.


## Runnable

Thread 를 실행시키기 위해서는 Runnable 이 필요하다. 또한 실제 실행 시킬 내용이 있는 run() 메소드도 필수이다.

```
public class RequestHandler implements Runnable {
    private Socket connection;
    
    ...
    
    public RequestHandler(Socket connectionSocket) {
        this.connection = connectionSocket;
    }

    public void run() {
        try (InputStream in = connection.getInputStream(); OutputStream out = connection.getOutputStream()) {
            ...
        }
    }
}
```

실제 Http 요청을 받기 위한 RequestHandler 의 기본 구조이다.

생성자로 실제 커넥션을 받아오고, 해당 커넥션에서 InputStream 과 OutputStream 을 꺼낸다.

해당 스트림들은 요청의 내용을 읽거나 응답을 작성할 때 사용된다.

## Request Header 출력

요청같은 경우는 InputStream 내부에 존재하며 이를 꺼내 사용해야한다.

실제 Java 에서 Input 을 입력받을 때에도, Scanner 가 아니라면 BufferedReader 를 아래와 같이 사용한다.

> BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

이때 System.in 의 타입은 InputStream 이다.

이를 활용하면 아래와 같이 InputStream 의 내용을 뽑아낼 수 있다.

> BufferedReader br = new BufferedReader(new InputStreamReader(in));

이후 실제 출력하는 과정은 생략하겠다.

## Path 분리 및 주입

> String uri = HttpRequestParser.parseRequestURI(line);

Request 의 가장 첫 줄을 통해서 Path 를 추출해낼 수 있다.

```
    public static String parseRequestURI(String requestLine) {
        String[] tokens = requestLine.split("\\s+");

        if (tokens.length >= 2) {
            return tokens[1];
        } else {
            throw new IllegalArgumentException("Invalid HTTP request line: " + requestLine);
        }
    }
```

일단은 가장 단순하게 공백을 기준으로 두 번째 위치하는 요소를 가져온다.
Path 에 공백이 존재하지만 않는다면 문제없이 Path 를 가져올 수 있다.

여기서 고민해야할 점은 의존성 주입이다.

실제 Spring 같은 경우에서는 **@Autowire** 를 기반으로 IoC 컨테이너에 필요한 클래스를 주입시킬 수 있다.
이외에서 private final 과 생성자를 통해서도 주입이 가능하며 가장 많이 쓰는 방법이다.

현재 기본 Java 를 통한 웹 서버에서는 이런 주입이 불가능하기에 파라미터로 넘겨주어야한다.
혹은 static 을 사용해 Bean 에 등록하는 느낌을 줄 수 있을 것 같다.

## 파일 읽고 응답

Path 를 얻었으니 이제 해당 Path 의 파일을 불러와야한다.

> java.nio 가 아닌 java.io 를 사용하도록 하겠다.

```
    public static byte[] readStaticResource(String uri) {
        String path = "src/main/resources/static/";
        File file = new File(path + uri);

        // Not Found
        if (!file.exists() || !file.isFile()) {
            return "<h1>404 NOT FOUND</h1>".getBytes();
        }

        StringBuilder contentBuilder = new StringBuilder();
        try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
            String line;
            while ((line = reader.readLine()) != null) {
                contentBuilder.append(line).append("\n");
            }
        } catch (IOException e) {
            System.err.println("Error reading file: " + path);
            e.printStackTrace();
            return null;
        }

        return contentBuilder.toString().getBytes();
    }

```

위 처럼 정적 파일들에 대한 경로를 미리 지정한다.
이후 File 을 통해 해당 파일을 열어본다.

java.io 에서는 readallbytes() 메서드가 없기에 한 줄씩 읽어 반환하도록 한다.

# 웹 서버 With Spring

Spring Boot 에서의 요청 응답과 정적 파일 로딩을 웹 서버 측면에서 살펴보겠다. 

## Tomcat

익히 알다시피 Spring 은 내장 Tomcat 을 사용한다. 

```
2024-07-02T15:21:40.313+09:00  INFO 8306 --- [demo] [           main] com.example.demo.DemoApplication         : Starting DemoApplication using Java 17.0.11 with PID 8306 (/Users/admin/Desktop/demo/build/classes/java/main started by admin in /Users/admin/Desktop/demo)
2024-07-02T15:21:40.314+09:00  INFO 8306 --- [demo] [           main] com.example.demo.DemoApplication         : No active profile set, falling back to 1 default profile: "default"
2024-07-02T15:21:40.618+09:00  INFO 8306 --- [demo] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port 8080 (http)
2024-07-02T15:21:40.623+09:00  INFO 8306 --- [demo] [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2024-07-02T15:21:40.623+09:00  INFO 8306 --- [demo] [           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/10.1.25]
2024-07-02T15:21:40.641+09:00  INFO 8306 --- [demo] [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2024-07-02T15:21:40.641+09:00  INFO 8306 --- [demo] [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 311 ms
2024-07-02T15:21:40.677+09:00  INFO 8306 --- [demo] [           main] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page: class path resource [static/index.html]
2024-07-02T15:21:40.762+09:00  INFO 8306 --- [demo] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
2024-07-02T15:21:40.766+09:00  INFO 8306 --- [demo] [           main] com.example.demo.DemoApplication         : Started DemoApplication in 0.587 seconds (process running for 5.803)
```

매 실행에서 나오는 로그에서도 해당 내용을 찾아볼 수 있다. 

그럼 Tomcat 의 Socket 통신 연결을 처리를 구경해보자.

> package org.springframework.boot.web.embedded.tomcat;

해당 패키지에서 getWebserver 메서드를 보자

```
public WebServer getWebServer(ServletContextInitializer... initializers) {
  ...

  Tomcat tomcat = new Tomcat();
  File baseDir = this.baseDirectory != null ? this.baseDirectory : this.createTempDir("tomcat");
  tomcat.setBaseDir(baseDir.getAbsolutePath());
  Iterator var4 = this.serverLifecycleListeners.iterator();

  ...

  Connector connector = new Connector(this.protocol);
  connector.setThrowOnFailure(true);
  tomcat.getService().addConnector(connector);
  this.customizeConnector(connector);
  tomcat.setConnector(connector);
  this.registerConnectorExecutor(tomcat, connector);
  tomcat.getHost().setAutoDeploy(false);
  this.configureEngine(tomcat.getEngine());
  Iterator var8 = this.additionalTomcatConnectors.iterator();

  ...
}
```

위와 같이 자동 설정 속에 톰캣과 서블릿 등의 웹 서버 기본 설정이 모두 포함되어있다. 

## HTTP11

이후 웹 서버 들어오는 요청 같은 경우는 Http11Processor 에서 처리한다.

> package org.apache.coyote.http11;

해당 패키지에서 실제 Header 에 대한 처리가 이루어진다.  

```
public class Http11Processor extends AbstractProcessor {
  ...

    public AbstractEndpoint.Handler.SocketState service(SocketWrapperBase<?> socketWrapper) throws IOException {
      ...
            if (!this.http09 && !this.inputBuffer.parseHeaders()) {
            ...
            }
      ...
    }
  ...
}
```

결국 Tomcat 은 요청을 파싱해 HttpServletRequest 객체로 생성해내고 이를 통해 헤더와 바디에 접근이 가능하다.

## DispatcherServlet

실제 Spring Boot 의 동작은 여기서부터 진행된다. Spring Boot 는 모든 요청을 DispatcherServlet 를 사용하여 처리한다. 

> package org.springframework.web.servlet;

DispatcherServlet 은 Http 요청을 수신하고 적절한 핸들러로 라우팅한다. 그리고 요청 처리 후 응답을 생성하기 위한 뷰를 선택하고 렌더링하기도 한다. 

## WebMvcAutoConfiguration

마지막으로 정적 페이지에 대한 처리이다. 

Spring Boot는 기본적으로 src/main/resources/static 또는 src/main/resources/public 디렉토리 내에 있는 정적 파일을 자동으로 서빙한다. 

이 동작은 Spring Boot의 자동 설정(Auto-Configuration) 기능에 의해 처리되며, 구체적으로는 WebMvcAutoConfiguration 클래스에 정의되어 있다.

- WebMvcAutoConfiguration 클래스가 정적 리소스의 기본 경로를 설정한다.
- ResourceHttpRequestHandler 가 실제로 정적 리소스를 처리하는 역할을 한다.

추가적으로 "/" 루트 경로에도 index.html 이 반환된다. 

이는 WelcomePageHandlerMapping 이 index.html 파일을 찾고 루트 경로에 매핑하는 역할을 하기 때문이다.
