---
title: "[Kotlin + Srping][Kotest BehaviorSpec] TDD? Entity Test Code"
excerpt: "[Kotlin + Srping][Kotest BehaviorSpec] TDD? Entity Test Code"

categories:
  - spring
tags:
  - [spring, kotlin]

toc: true
toc_sticky: true

date: 2024-03-31
last_modified_at: 2024-03-31
---

TDD 및 테스트 코드의 중요성은 어디에서나 쉽게 찾아볼 수 있다.
하지만 단편적인 코드만 존재할 뿐 실제 어떻게 사용해야할 지 감이 안잡혔었는데, 최범균님의 [세미나 공유 - TDD 소개 및 시연](https://www.youtube.com/watch?v=6Vt-wKPBbuc) 유튜브를 통해 살짝 감을 잡을 수 있었다. 매우 쉽게 이해할 수 있어 추천한다. 


실제 코드와 함께 TDD 를 어떻게 진행해야할 지 살펴보자.

# Test Code

```

class UserTest() : BehaviorSpec() {
    init {
        given("정상적인 유저가 존재할 때") {
            `when`("유저의 비밀번호를 암호화하면") {
                then("비밀번호가 암호화되어야 한다") {
                }
            }
            `when`("유저 정보 password 만 업데이트 하면") {
                then("비밀번호가 업데이트 되어야 한다") {
                }
            }
            `when`("유저 정보 nickname 만 업데이트 하면") {
                then("nickname 이 업데이트 되어야 한다") {
                }
            }
            `when`("유저 정보 phone 만 업데이트 하면") {
                then("phone 이 업데이트 되어야 한다") {
                }
            }
            `when`("nickname 과 password 만 업데이트하면") {
                then("nickname 과 password 가 업데이트 되어야 한다") {
                }
            }
        }
    }
}

```

우선 테스트를 정의한다. 테스트 대상은 User 라는 Entity 이며 해당 Entity 에는 두 가지 로직이 존재한다.

- Password 를 암호화
- Dto 를 통한 필드 업데이트

이를 위해 given-when-then 를 통해 검증해야할 상황을 먼저 작성하자.

## PasswordEncoder

```

class UserTest() : BehaviorSpec() {
    private val passwordEncoder: PasswordEncoder = PasswordEncoderFactories.createDelegatingPasswordEncoder()

    init {
        given("정상적인 유저가 존재할 때") {
            var testUser = User(
                email = "user@email.com", password = "password123!",
                name = "name", nickname = "nickname", phone = "01012345678",
                profileImg = "profileImg", description = "description", userType = null
            )

            `when`("유저의 비밀번호를 암호화하면") {
                then("비밀번호가 암호화되어야 한다") {
                    testUser.encodePassword(passwordEncoder)

                    testUser.password shouldNotBe "password123!"
                    passwordEncoder.matches("password123!", testUser.password) shouldBe true
                }
            }
        }
    }
}

```

passwordEncoder 는 실제 Autowired 로 주입시키는 코드를 그대로 사용하였다.
@Autowired 를 사용할 수 있지만 이를 위해 @SpringBootTest 를 사용해야한다.

해당 어노테이션을 사용하는 순간 단위테스트가 아니라 통합테스트로 비용이 매우 커질 수 있기 때문에 주의해야한다. 

또한 testUser 라는 User 를 생성하고 encodePassword 를 실행한다.

검증은 기존 비밀번호가 아니면서, matches 를 사용해 기존 암호와 matches 를 통과해야한다.

## encode() in Entity

```
@Entity
@Table(
...
)
class User(

 ...

) : BaseEntity() {
    ...

    fun encodePassword(passwordEncoder: PasswordEncoder) {
        password = passwordEncoder.encode(password)
    }

    ...
}
```

User 에서 ERD 정의 등은 논점 외이므로 생략하고 메서드만 설명하도록 하겠다.

위의 테스트 코드를 만족시키기 위해서는 엔티티의 비밀번호를 passwordEncoder 를 통해서 인코딩 시키면 된다. 

Autowired 를 통해 passwordEncoder 를 받기 위해서 파라미터로 전달받으며 이는 Service 등에서 주입할 수 있다.

## Update



```

class UserUpdateReqDto(

    @field:Pattern(
        regexp = "^(?=.*[a-zA-Z])(?=.*[0-9])(?=.*[!@#\$%^&*])[a-zA-Z0-9!@#\$%^&*]{8,20}\$",
        message = "영문, 숫자, 특수문자를 포함한 8~20자리로 입력해주세요"
    )
    @JsonProperty("password")
    private val _password: String? = null,

    @JsonProperty("nickname")
    private val _nickname: String? = null,

    @field:Pattern(
        regexp = "^01\\d{9}$",
        message = "핸드폰 번호를 다시 확인해주세요"
    )
    @JsonProperty("phone")
    private val _phone: String? = null,

    @JsonProperty("profileImg")
    private val _profileImg: String? = null,

    @JsonProperty("description")
    private val _description: String? = null,

    ) {

    val password: String?
        get() = _password

    val nickname: String?
        get() = _nickname

    val phone: String?
        get() = _phone

    val profileImg: String?
        get() = _profileImg

    val description: String?
        get() = _description
}

```

업데이트를 위해서는 DTO 가 필요해 위와 같이 선언하였다. 검증을 위한 구조이고, nullable 설정하였기에 필요한 값들만 담아낼 수 있다. 


```

class UserTest() : BehaviorSpec() {

    init {
        beforeTest {
            testUser = User(
                email = "user@email.com", password = "password123!",
                name = "name", nickname = "nickname", phone = "01012345678",
                profileImg = "profileImg", description = "description", userType = null
            )
        }

        given("정상적인 유저가 존재할 때") {

            ...

            `when`("유저 정보 password 만 업데이트 하면") {
                val userUpdateReqDto = UserUpdateReqDto(
                    _password = "password1234!"
                )

                then("비밀번호가 업데이트 되어야 한다") {
                    testUser.update(userUpdateReqDto)

                    testUser.password shouldBe userUpdateReqDto.password
                }
            }
            `when`("유저 정보 nickname 만 업데이트 하면") {
                val userUpdateReqDto = UserUpdateReqDto(
                    _nickname = "newNickname"
                )

                then("nickname 이 업데이트 되어야 한다") {
                    testUser.update(userUpdateReqDto)

                    testUser.nickname shouldBe userUpdateReqDto.nickname
                }
            }
            `when`("유저 정보 phone 만 업데이트 하면") {
                val userUpdateReqDto = UserUpdateReqDto(
                    _phone = "01012345679"
                )

                then("phone 이 업데이트 되어야 한다") {
                    testUser.update(userUpdateReqDto)

                    testUser.phone shouldBe userUpdateReqDto.phone
                }
            }
            `when`("nickname 과 password 만 업데이트하면") {
                val userUpdateReqDto = UserUpdateReqDto(
                    _nickname = "newNickname",
                    _password = "password1234!"
                )

                then("nickname 과 password 가 업데이트 되어야 한다") {
                    testUser.update(userUpdateReqDto)

                    testUser.nickname shouldBe userUpdateReqDto.nickname
                    testUser.password shouldBe userUpdateReqDto.password
                }
            }
        }
    }

    companion object {
        private val passwordEncoder: PasswordEncoder = PasswordEncoderFactories.createDelegatingPasswordEncoder()
        var testUser = User(
            email = "user@email.com", password = "password123!",
            name = "name", nickname = "nickname", phone = "01012345678",
            profileImg = "profileImg", description = "description", userType = null
        )
    }
}


```

테스트 코드이다. 우선 계속적으로 User 객체를 사용해야하기 때문에 companion object 에 testUser 를 선언해두었다. companion object 는 static 변수라고 생각하면 된다.

그리고 매번 테스트마다 testUser 를 원래의 값을 하나하나 초기화시켜주었다.
이를 통해 매번 테스트에서는 동일한 testUser 값이 들어갈 것이다. 

매 테스트 요청사항에 맞게 userUpdateReqDto 를 선언하였다. 필요한 값만 선언되었기에 이외의 값은 null 로 저장되어있음을 기대한다. 

이후 update 하나의 메서드로 값이 있는 userUpdateReqDto 에 맞추어 최신화한다. 

## Update() in Entity

```
@Entity
@Table(
...
)
class User(
    ...
) : BaseEntity() {
    
    ...

    fun update(userUpdateReqDto: UserUpdateReqDto) {
        userUpdateReqDto.password?.let { password = it }
        userUpdateReqDto.nickname?.let { nickname = it }
        userUpdateReqDto.phone?.let { phone = it }
        userUpdateReqDto.profileImg?.let { profileImg = it }
        userUpdateReqDto.description?.let { description = it }
    }
}

```

update() 메서드를 살펴보자. Dto 를 확인하고 해당 필드에 값이 존재하는 경우에만 엔티티의 값을 바꿔주었다. 

# 결론

현재 하고자하는 것은 **단위 테스트** 임을 명시해야한다. 따라서 Service 의 로직을 테스트하고 싶다면 하위 계층인 Entity, Repository 의 로직은 모두 원하는대로 동작함을 전제하에 명시해주며 테스트를 진행해야한다.

그를 위해 Entity TDD 를 우선적으로 진행해보았다. 

Validation 이 DTO 에서 이루어졌기 때문에 해당 테스트 코드는 매우 간결하다.

하지만 중요한 것은 TDD 의 흐름을 파악하기에는 충분하다고 생각한다!! 