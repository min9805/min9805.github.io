---
title: "[우아콘2022] API Gateway Pattern 에는 API Gateway 가 없다."
excerpt: "우아콘 2022 발표 'API Gateway Pattern 에는 API Gateway 가 없다'"
categories:
  - programming
tags:
  - [Python, Design Pattern]

toc: true
toc_sticky: true

date: 2024-03-27
last_modified_at: 2024-03-27
---

 실제 졸업 프로젝트에서도 MSA 관련 프로젝트를 진행했었고, 다수의 컨퍼런스, 테크톡에서 흔히 찾아볼 수 있는 주제가 MSA 이다.
  
 새로운 개인 프로젝트를 MSA 구조를 사용해보려 하다가 좋은 영상이 있어 정리하고자 한다.

 # API Gateway Framework

 ![image](https://github.com/min9805/min9805.github.io/assets/56664567/271af607-9d58-4108-92a2-7cdd5f2b9ee0)

 나를 포함해 대부분이 API Gateway 라고 여길만한 구조이다. 하나의 엔드포인트를 가지며 게이트웨이에서 다양한 마이크로서비스에 접근하는 것처럼 보인다. 하지만 안타깝게도 해당 구조는 API Gateway Pattern 이라 말할 수 없다. 

# API Gateway Pattern 정의

![image](https://github.com/min9805/min9805.github.io/assets/56664567/5b7b3ef9-dce6-437b-92a6-e23a4fcd17d2)

마이크로서비스 패턴 책에 나오는 예시는 위와 같다.

앞선 그림과 차이점이 무엇일까??

가장 쉽게 찾아볼 수 있는 것은 호출 메서드의 차이이다. 위에서는 클라이언트가 마이크로서비스의 메서드를 직접 호출하는 것과 같다. 게이트웨이는 그저 라우팅만 해주는 형태로 설정만으로 가능하다. 하지만 아래 그림에서는 게이트웨이로 getOrderDetails() 메서드가 호출되며 게이트웨이는 3가지 마이크로서비스에 요청을 보내고 이를 종합해 클라이언트측에 전달하는 것이다. 이를 위해서는 개발자가 특정 비즈니스 요청에 대한 로직을 작성해야한다.

## 정리

간단하게 정리하자면 

- API Gateway Pattern 은 클라이언트의 요청을 받아 내부 마이크로서비스를 호출하는 로직을 직접 작성하고 응답 내용을 취사 선택하여 필요한 것만 내보내는 것
- 해당 과정이 핵심이지 Framework 사용 유무가 Gateway Pattern 이 되지는 않는다.
- 즉, Spring Cloud Gateway 나 Netflix Zuul 같은 Framework 를 사용하더라도 단순 API 라우팅만 존재한다면 API Gateway Pattern 이라 볼 수 없다.

### API gateway Framework

외부에 API를 제공하기 위한 API Gateway Pattern 은 Spring Cloud Gateway 나 Netflix Zuul

### API Gateway Pattern

클라이언트의 요청을 받아서 내부 마이크로서비스를 호출하는 로직을 직접 작 성하고 응답 내용도 취사 선택하여 필요한 것만 내보내는 것

### BFF - Backend For Frontend

API Gateway Pattern 과 유사하게 직접 구현하는 로직을 작성하는 방법과 API Gateway 애플리케이션을 클라이언트 기기 단위로 구분하고 소유권을 정하는 방식 조합

![image](https://github.com/min9805/min9805.github.io/assets/56664567/521394db-e5b1-4ef3-9128-cb8cd2f89fea)
![image](https://github.com/min9805/min9805.github.io/assets/56664567/96c77fc4-ce9b-40f4-a511-1f96968a4f26)

BFF 와 API Gateway 차이점은 진입점의 차이이다. API Gateway 는 단일 진입점인 반면 BFF 는 단일 유형의 클라이언트에만 책임이 있다. 즉, BFF 는 특정 프런트엔드에 맞게 데이터를 가져올 수 있다는 점이다. BFF 는 API Gateway 유형이라 볼 수 있으며 동일한 기능을 수행하지만 범위와 상호 작용하는 클라이언트 수에서 차이가 발생한다.

# MVC 기반 계층 구조

![image](https://github.com/min9805/min9805.github.io/assets/56664567/c742c46f-7acb-408d-afa4-70ee08e974c6)

 가장 기본이 되는 Spring 의 WebMVC 기반 Monolithic 아키텍쳐이다.

 그림에서도 보이듯이 상위 계층인 프리젠테이션 계층과 하위 계층인 비즈니스 계층이 명확하게 구분되어있다. 

프레젠테이션 계층에는 모든 요청에 대한 필터와 인터셉터, 컨트롤러와 파사드가 존재한다. 파사드는 해당 요청이 여러 비즈니스의 응답을 필요로 할 때 이를 처리해 각 비즈니스에 요청을 보낼 수 있게하는 계층이라 볼 수 있다.

비즈니스 계층은 해당 비즈니스의 필터같은 AOP, 로직을 처리하는 서비스와 레포지토리가 존재한다. 

![image](https://github.com/min9805/min9805.github.io/assets/56664567/50114192-7d31-4fe4-af0b-165869b05e6c)

위 그림은 Gateway 가 라우팅만 진행하는 경우의 상황이다. 

프레젠테이션 계층에서 컨트롤러와 파사드 계층이 사라진 것을 확인할 수 있다. 게이트웨이가 내부 서비스 API 를 외부로 노출시키는 역할만 하기 때문이다. 

## 문제점

그렇다면 라우팅만 진행한 경우 실제 발생할 수 있는 문제점들이다.

### 1. 보안 취약성

- 인증과 권한 처리 누락
- IDOR(Insecure Direct Object Reference) 중 "수평적 권한 상승"

### 2. 잘못된 로직 구현 전가

- 단순 라우팅을 하게 되면 Facade가 필요한 경우에 엉뚱한 로직을 전가하게 됨
  
  
1. 클라이언트에 구현 전가

- 클라이언트가 직접 비즈니스 계층이 로직을 조합해서 호출

클라이언트 개발자가 UI 처리에 집중하지 못하게 됨
버그가 존재할 경우 Mobile App은 각종 앱스토어 인증 절차에 걸리는 시간과 사용자들이 자발적으로 앱을 업데이트 할 때까지의 시간동안 버그 해결이 불가능해짐


2. 각 마이크로서비스에 전가

- 서비스가 엉뚱하게 타 서비스를 호출해서 클라이언트 요청에 필요한 정보를 조합해 반환하는 것처럼 본인 비즈니스가 아닌것에 대해 구현하고 의존하게 됨

특정 서비스 개발자들은 알지 못하는 로직 구현 책임 맡게 됨
불필요한 복잡도, 결합도 증가 -> 장애 여파 증가


### 3. 클라이언트, 비즈니스 계층 간 강결합

Business 계층 개발자는 항상 Clinet의 변경사항을 주시해야 함
  - 클라이언트에서 호출하고 있는 API 수정 시 예상치 못한 정보가 유출될 수 있다.
단순 요청/응답 라우팅은 어떠한 경우에도 사용하지 말고항상 응답 내용을 프리젠테이션 계층에서 Client에 필요한 것만 정제해서 내보내야한다.


### 4. 클라이언트 성능 저하

여러 API 호출을 조합해야 하므로 network 호출이 여러번 일어나게 되고 불필요한 데이터들을 포함하고 있거나, 필요한 데이터가 없어서 추가 API 호출 시 Overhead 발생한다.
API Gateway Pattern을 Non Blocking IO / Reactive하게 구현하여 성능 향상 추천(Spring WebFlux 등)
GraphQL로 이 문제를 해소 가능하지만, 비즈니스 계층 Microservice 에 GraphQL을 구축하고 이를 그대로 프리젠테이션 계층까지 노출하면 앞서 말했던 보안 문제들이 그대로 발생한다. GraphQL 도 API Gateway Pattern에 따라 프리젠테이션 계층 서 버 애플리케이션으로 적절한 보안 처리를 해서 별도 구축해야한다. 
 

### 5. 내부 Service들 간의 Protocol 제약

AGF로 프레젠테이션 계층을 구현한다는 것은 비즈니스 계층 서비스 모두 HTTP API(REST)로 고정된다는 의미이다.
내부 비즈니스 계층을 밖으로 한 번 노출하면 프로토콜 변경 어려워지기 때문이다. 
즉, gRPC, Message Queue, HTTP API, 기타 등등 을 적용할 수 있는 자유도를 포기하는 일이다.


## 계층형 아키텍처의 일반적인 원칙 위반 사례

![image](https://github.com/min9805/min9805.github.io/assets/56664567/0a129fd0-0de0-47e7-84a6-662bdfe0ef60)

마이크로서비스를 나눌 때 비즈니스 계층으로 나누는 것이 아니라 프레젠테이션 계층으로 나누어버리는 것이다. 인증 및 권한 처리와 비즈니스 로직을 함께 처리해버리는 것이다.

여기까지는 괜찮을 수 있지만 문제는 다음이다.

![image](https://github.com/min9805/min9805.github.io/assets/56664567/28fd7a6e-32be-4b3a-83e7-7294f897a590)

대부분의 경우에 프레젠테이션 계층 API 가 다른 프레젠테이션 계층 API 를 호출하는 경우가 존재한다. 이는 컨트롤러가 컨트롤러를 호출하고, 서비스가 컨트롤러를 호출하는 사례이다. 

결론적으로 IDOR 수직적 권한 상승에 취약해진다. 이는 해커가 무제한의 권한을 가진 API 를 알아낼 수 있다는 뜻이다. 


![image](https://github.com/min9805/min9805.github.io/assets/56664567/facef54c-f002-48f7-aa80-2b376761b4c3)

하나의 마이크로서비스는 하나의 계층 역할만 해야한다.

의존 관계는 항상 프레젠테이션 계층에서 비즈니스 계층으로 흐르거나 비즈니스 계층 간에만 이루어져야한다.

또한 비즈니스 계층 간 순환 호출이 발생해서는 안된다.


## AGF 를 횡단 관심사 처리 관점에서도 사용하지 말기

앞서 AGF 가 필터나 인터셉터같이 횡단 관심사 처리에 유용할 수 있음을 보였다. 하지만 되도록 사용하지 않아야한다.

- Single Point Of Failure
- 불필요한 서버 관리 부담 늘어나고 비용 증가
-  Client가 하나의 end point만 바라본다고 해서 좋아질 것은 크게 없음
  - Service.메소드수천개() 를 좋다고 할 수 있는 가?

결국 가장 중요한 것은 문서화이다. 필요한 API를 빠르게 찾아볼 수 있게 문서들 잘 모아두자.


# 출처

[API Gateway Pattern에는 API Gateway가 없다 #우아콘2022 #Day2_음식그이상의것을문앞으로](https://www.youtube.com/watch?v=P2nM0_YptOA)

[BFF Pattern vs Gateway Pattern](https://alirezafarokhi.medium.com/bff-pattern-vs-gateway-pattern-45706ffb9978)