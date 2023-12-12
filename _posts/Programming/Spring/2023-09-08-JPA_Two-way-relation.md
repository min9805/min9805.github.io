---
title: "[JPA] 양방향 연관관계에서 Entity 조회"

categories:
  - spring
tags:
  - [spring, jpa]

toc: true
toc_sticky: true

date: 2023-09-08
last_modified_at: 2023-09-08
---


# 양방향 연관관계 

 ```

public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id;

    ...

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();

    ...

}

 ```

 ```
 
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "order_id")
    private Long id;

    ...

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    ...

}

 ```

 위와 같이 Member 와 Order 엔티티가 존재할 때,
 
 Member 가 필드로 Order 를 가지고 있고,

 Order 도 필드로 Member 를 가지고 있다.

```
    @GetMapping("/api/v1/members")
    public List<Member> membersV1() {
        List<Member> members = memberService.findMembers();
        return members;
    }
```

이때 그냥 Member 를 조회해서 반환하게되면 


> [org.springframework.http.converter.HttpMessageNotWritableException]

> sendError()

> [Request processing failed: org.springframework.http.converter.HttpMessageNotWritableException: Could not write JSON: Infinite recursion (StackOverflowError)]

<details>
<summary>ERROR</summary>

```
2023-09-08T22:39:38.244+09:00  WARN 3984 --- [nio-8080-exec-2] .w.s.m.s.DefaultHandlerExceptionResolver : Failure while trying to resolve exception [org.springframework.http.converter.HttpMessageNotWritableException]

java.lang.IllegalStateException: Cannot call sendError() after the response has been committed 

  ...

2023-09-08T22:39:38.249+09:00 ERROR 3984 --- [nio-8080-exec-2] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed: org.springframework.http.converter.HttpMessageNotWritableException: Could not write JSON: Infinite recursion (StackOverflowError)] with root cause

java.lang.StackOverflowError: null
	at java.base/java.lang.ClassLoader.defineClass1(Native Method) ~[na:na]
```

</details>

---
위 예외와 에러가 발생하게 된다. 이는 Member 와 Order 가 서로 무한루프식으로 서로의 데이터를 호출하기 때문이다. 

결국 아래와 같은 Json 데이터 형식을 띈다.


```
[
  {
    member_id : 1001
    orders : [
      order_id : 3001
      member : {
        member_id : 1001
        orders : [
          order_id : 3001
          member : {
             ...
```

# 해결 방법

## @JsonIgnore

Member 혹은 Order 필드에서 양방향 연관 관계를 만드는 필드에 @JsonIgnore 어노테이션을 추가하는 방법이다.

해당 어노테이션은 직렬화, 역직렬화에 사용되는 논리적 속성 값을 무시할 때 사용한다.

Order class 에서 Member 필드에 사용한다면 직렬화 시 Order 내부의 Member 필드는 JSON 에 담기지 않는다!

### Hibernate5JakartaModule

```

@Bean
Hibernate5JakartaModule hibernate5Module() {
return new Hibernate5JakartaModule();
}

```

하지만 이후에 한 가지 과정을 더 해주어야하는데, 해당 모듈을 빈으로 등록해야한다.

현재 코드는 기본적으로 지연로딩 전략을 사용한다. 즉, 엔티티를 조회하는 시점에서는 실제 객체가 아닌 프록시 객체를 가지고 있다는 뜻이다.

Hibernate5JakartaModule 하이버네이트가 만든 프록시 객체를 JSON으로 읽을 수 있도록 도와주는 역할을 한다.



---
# etc

- 기본적으로 Entity 를 노출하는 것이 잘못된 설계이다.
- 양방향 연관관계를 가지는 엔티티를 그대로 노출하기 때문에 발생하는 문제이다.
- 근본적으로 DTO 로 변환하는 방법이나 DTO 로 조회하는 방법이 옳다. 