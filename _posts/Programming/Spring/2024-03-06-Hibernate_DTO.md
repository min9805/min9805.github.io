---
title: "[Spring + React] org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor, localhsot:3000/ 400 Bad Request. Entity 응답 시 에러 발생"
excerpt: "[Spring + React] org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor, localhsot:3000/ 400 Bad Request. Entity 응답 시 에러 발생"

categories:
  - spring
tags:
  - [spring, kotlin]

toc: true
toc_sticky: true

date: 2024-03-06
last_modified_at: 2024-03-06
---

# 개요

N:M 관계를 여러 이유로 1:N, M:1 로 풀어 엔티티를 생성했습니다.

현재 Member 와 Group 기능의 엔티티를 생성하려 했으나 Mysql 에서 "group", "groups" 가 모두 예약어로 생성 안되는 에러가 계속 발생했습니다.
따라서 Member 와 Team 으로 중간 MemberTeam 테이블을 놓고 다대다 관계를 생성했습니다.

이후 로그인 된 Member 가 속한 Team 을 모두 조회하려는 로직을 짜던 중 에러가 발생했습니다.

![image](https://github.com/min9805/min9805.github.io/assets/56664567/a0d64580-bffc-485f-b0b1-4865f2c879cf)

가장 이해가 안되는 부분은 지금까지 다른 axios 의 http 요청에서는 문제가 없었는데 갑자기 localhost:3000 으로 데이터 요청이 오가고 400 Bad Request 가 실행된다는 점.
그리고 response 의 data 에 백엔드에서 보내는 데이터가 분명히 들어 있다는 점이였습니다.

이걸 해결하기 위해 하나하나 까보도록 하겠습니다. 


# Backend

## Entity

```

@Entity
@Table
class Team(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long? = null,

    @Column(nullable = false, length = 100)
    var name: String,

    ) {
    @OneToMany(fetch = FetchType.LAZY, mappedBy = "team")
    val teamEvent: List<TeamEvent>? = ArrayList()

    ...

}

@Entity
class MemberTeam(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    var id: Long? = null,

    @ManyToOne(fetch = FetchType.LAZY)
    val member: Member,

    @ManyToOne(fetch = FetchType.LAZY)
    val team: Team,
) {

}

```

Team 은 TeamEvent 와 1:N 관계를 또 가지고 있는 형태입니다.

## Repository

```

interface MemberTeamRepository : JpaRepository<MemberTeam, Long> {
    fun findByMemberId(memberId: Long): List<MemberTeam>

}

```

이를 위해 Member_Id 로 MemberTeam 리스트를 반환하는 findBy 함수를 지정했습니다.


## Service


```
@Service
@Transactional
class TeamService(
    private val memberTeamRepository: MemberTeamRepository,
    ...
) {
    fun findTeamsByMemberId(memberId: Long): List<Team> {
        val memberTeams = memberTeamRepository.findByMemberId(memberId)
        return memberTeams.map { it.team }
    }

    ...
}
```

Service 는 해당 findBy 함수를 통해 memberTeams 리스트를 가져오고 team 만 추출해 반환합니다.


![image](https://github.com/min9805/min9805.github.io/assets/56664567/fd40f938-5f2a-4f07-b358-30ee24f651f8)

이제 Controller 를 통해 실제 테스트를 해봤을 때 에러가 발생하긴 하지만 결과를 일단 보여주는 것을 확인할 수 있었습니다.
에러를 일단 차치하고 다음으로 진행했습니다.

# FrontEnd

![image](https://github.com/min9805/min9805.github.io/assets/56664567/a0d64580-bffc-485f-b0b1-4865f2c879cf)

이런 에러가 계속 발생했고 localhost:3000 으로 가는 것이 이상해 Frontend 에서 문제점을 우선 찾았습니다. 

axios 자체는 기존 요청들과 차이점이 없었기 때문에 문제는 없었습니다.

## Proxy

```
  "proxy": "http://localhost:8080"

```

잘 모르고 사용했지만 cors 를 위해 proxy 를 사용하고 있었고 이 부분이 error 에 localhost:3000 이 존재하는 이유입니다.


# Error

![image](https://github.com/min9805/min9805.github.io/assets/56664567/fd40f938-5f2a-4f07-b358-30ee24f651f8)

이 부분을 다시 보면 response 가 2개 인 것을 알 수 있습니다. 이게 proxy 로 가면서 2개의 응답을 처리하지 못해 400 Bad Request 가 발생하는 이유로 추정됩니다.

그러면 왜 2개의 응답이 발생할까?

## ByteBuddyInterceptor

> [21:39:23.171][WARN ][org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver.logException:line207] - Resolved [org.springframework.http.converter.HttpMessageConversionException: Type definition error: [simple type, class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor]]

요청을 보내면 위 경고 메시지가 발생합니다.
 이는 HTTP 요청이나 응답의 데이터를 Java 객체로 변환하는 동안에 발생했음을 의미합니다.

 org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor는 Hibernate의 프록시 객체인데, 이 객체를 변환하는 과정에서 문제가 발생했다는 것을 의미합니다. 이러한 문제는 대개 Hibernate에서 관리되는 엔터티 객체를 직렬화하거나 역직렬화할 때 발생합니다.

 이는 위에서 Team 이 Events 를 가지고 있지만 FetchType.Lazy 로 인해 직렬화를 못하기에 발생하는 에러입니다. 

## Exception

```
    @ExceptionHandler(Exception::class)
    protected fun defaultException(ex: Exception): ResponseEntity<BaseResponse<Map<String, String>>> {
        val errors = mapOf("미처리 에러" to (ex.message ?: "Not Exception Message"))
        return ResponseEntity(BaseResponse(ResultCode.ERROR.name, errors, ResultCode.ERROR.msg), HttpStatus.BAD_REQUEST)
    }

```

CustomExceptionHandler 를 사용하고 있고 위에서 exception 이 발생했기 때문에 해당 Error 응답이 한 번 더 전송되고 있었습니다. 


## DTO

```
data class GetTeamsDtoResponse(
    val teamId: Long,
    val name: String,
)

```

```
class TeamService(
    private val memberTeamRepository: MemberTeamRepository,
    ...
) {
    fun findTeamsByMemberId(memberId: Long): List<GetTeamsDtoResponse> {
        val memberTeams = memberTeamRepository.findByMemberId(memberId)
        return memberTeams.map { it.team.toDto() }
    }
    ...
}
```

결국 Team 엔티티의 Events 는 당장에 필요없기 때문에 필요한 정보들만 DTO 에 넣어 반환하도록 변경했습니다.

![image](https://github.com/min9805/min9805.github.io/assets/56664567/dd54724a-10f0-472c-abb8-c2f7ef55fbc2)

백엔드에 대한 요청이 정상적으로 1개의 응답이 오는 것을 확인할 수 있었고 프론트엔드에서도 정상 처리하는 것을 확인했습니다. 


# 결론

밖과 연결된 데이터는 DTO 를 사용하자!!