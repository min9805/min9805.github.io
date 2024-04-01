---
title: "[Kotlin + Srping][Kotest BehaviorSpec] Repository, Service Test Code"
excerpt: "[Kotlin + Srping][Kotest BehaviorSpec] Repository, Service Test Code"

categories:
  - spring
tags:
  - [spring, kotlin]

toc: true
toc_sticky: true

date: 2024-04-01
last_modified_at: 2024-04-01
---

Kotest 를 사용한 Repositroy, Service Test Code 를 직접 작성해보도록 하겠다. 

Kotest 의 BehaviorSpec 을 사용하였다.

# Repository Test

```
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class UserRepositoryTest(
    @Autowired private val userRepository: UserRepository
) : BehaviorSpec() {

    init {

        beforeSpec {
            userRepository.save(
                User(
                    email = "user@email.com", password = "password123!",
                    name = "name", nickname = "nickname", phone = "01012345678",
                    profileImg = "profileImg", description = "description", userType = null
                )
            )
        }

        afterSpec {
            userRepository.deleteAllInBatch()
        }

        given("UserRepository") {
            `when`("findByEmail 을 했을 때 이메일 유저가 있다면") {
                then("should return User") {
                    val findByEmail = userRepository.findByEmail("user@email.com")
                    findByEmail?.let { findByEmail.email shouldBe "user@email.com" }
                }
            }
            `when`("findByEmail 을 했을 때 이메일 유저가 없다면") {
                then("should return null") {
                    val findByEmail = userRepository.findByEmail("NotExistsEmail")
                    findByEmail shouldBe null
                }
            }
            `when`("findByNickname 을 했을 때 닉네임 유저가 있다면") {
                then("should return User") {
                    val findByNickname = userRepository.findByNickname("nickname")
                    findByNickname?.let { findByNickname.nickname shouldBe "nickname" }
                }
            }
            `when`("findByNickname 을 했을 때 닉네임 유저가 없다면") {
                then("should return null") {
                    val findByNickname = userRepository.findByNickname("NotExistsEmail")
                    findByNickname shouldBe null
                }
            }
        }
    }
}

```

TestCode 의 어노테이션부터 하나씩 살펴보도록 하자

## Annotation

### @DataJpaTest

@DataJpaTest 에는 이미 많은 어노테이션이 포함되어있다.

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@BootstrapWith(DataJpaTestContextBootstrapper.class)
@ExtendWith(SpringExtension.class)
@OverrideAutoConfiguration(enabled = false)
@TypeExcludeFilters(DataJpaTypeExcludeFilter.class)
@Transactional
@AutoConfigureCache
@AutoConfigureDataJpa
@AutoConfigureTestDatabase
@AutoConfigureTestEntityManager
@ImportAutoConfiguration
public @interface DataJpaTest {
```

@Transactional 어노테이션도 붙어있기 때문에 각 단위 테스트가 끝나면 자동으로 롤백되어야한다. 

하지만 이대로 실행한다면

> org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'dataSourceScriptDatabaseInitializer' defined in class path resource [org/springframework/boot/autoconfigure/sql/init/DataSourceInitializationConfiguration.class]: Unsatisfied dependency expressed through method 'dataSourceScriptDatabaseInitializer' parameter 0: Error creating bean with name 'dataSource': Failed to replace DataSource with an embedded database for tests. If you want an embedded database please put a supported one on the classpath or tune the replace attribute of @AutoConfigureTestDatabase.

에러가 발생한다.

### @AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)

에러 메세지를 읽어보면 @AutoConfigureTestDatabase 를 사용하라 나와있다. 

결론적으로 @DataJpaTest와 @AutoConfigureTestDatabase를 사용하여 테스트 시에만 특정 DB를 사용할 수 있다.

물론 연결에 관한 설정은 test 쪽의 application.yml 파일을 통해 설정해주어야한다. 

해당 과정이 끝나면 RepositoryTest 는 실제 Test DB 와 연결되었음을 알 수 있다.

## Repository Test Code

UserRepositroy 는 JpaRepository 인터페이스를 상속받았다. 따라서 기본적인 메소드들에 대해서는 굳이 테스트할 필요가 없다고 생각한다. 

```
    init {

        beforeSpec {
            userRepository.save(
                User(
                    email = "user@email.com", password = "password123!",
                    name = "name", nickname = "nickname", phone = "01012345678",
                    profileImg = "profileImg", description = "description", userType = null
                )
            )
        }

        afterSpec {
            userRepository.deleteAllInBatch()
        }

        given("UserRepository") {
            `when`("findByEmail 을 했을 때 이메일 유저가 있다면") {
                then("should return User") {
                    val findByEmail = userRepository.findByEmail("user@email.com")
                    findByEmail?.let { findByEmail.email shouldBe "user@email.com" }
                }
            }
            `when`("findByEmail 을 했을 때 이메일 유저가 없다면") {
                then("should return null") {
                    val findByEmail = userRepository.findByEmail("NotExistsEmail")
                    findByEmail shouldBe null
                }
            }
            `when`("findByNickname 을 했을 때 닉네임 유저가 있다면") {
                then("should return User") {
                    val findByNickname = userRepository.findByNickname("nickname")
                    findByNickname?.let { findByNickname.nickname shouldBe "nickname" }
                }
            }
            `when`("findByNickname 을 했을 때 닉네임 유저가 없다면") {
                then("should return null") {
                    val findByNickname = userRepository.findByNickname("NotExistsEmail")
                    findByNickname shouldBe null
                }
            }
        }
    }
```

그것이 beforeSpec 에 Repository 메서드가 존재하는 이유이다.

명확하게는 save() 메서드도 테스트의 대상이 되겠지만 편의상 DB 에 유저를 넣어놓고 사용하겠다.

테스트가 종료되면 userRepository.deleteAllInBatch() 를 통해서 Test DB 를 정리한다.

이때에도 deleteAll() 메서드는 Row 의 개수만큼 쿼리가 발생하지만 deleteAllInBatch()를 통해 하나의 쿼리로 DB 를 정리할 수 있다.

이후 생성한 메서드인 findByEmail, findByNickname 을 테스트하는 코드이다. 

# Service Unit Test

Service 의 테스트도에서 가장 삽질한 곳은 login 메서드이다.

```
    /**
     * Email password 를 통해 로그인 시 Authenticate 진행
     *
     * @param loginReqDto
     * @return jwt Token
     */
    @Transactional
    fun login(loginReqDto: LoginReqDto): TokenInfo {
        val authenticationToken = UsernamePasswordAuthenticationToken(loginReqDto.email, loginReqDto.password)
        val authentication = authenticationManagerBuilder.`object`.authenticate(authenticationToken)

        return jwtTokenProvider.createToken(authentication)
    }
```

해당 메서드는 Bean 으로 주입받아 사용된다.

## BehaviorSpec

### Service Test Code

```
@SpringBootTest()
class UserServiceTest : BehaviorSpec() {
    final var userRepository: UserRepository = mockk()
    final var userRoleRepository: UserRoleRepository = mockk()


    @MockBean
    lateinit var jwtTokenProvider: JwtTokenProvider

    @MockBean
    lateinit var messageSource: MessageSource

    @MockBean
    lateinit var passwordEncoder: PasswordEncoder

    @MockBean
    lateinit var authenticationManagerBuilder: AuthenticationManagerBuilder

    @Autowired
    lateinit var target: UserService


```

UserService 는 JwtTokenProvider, MessageSource, PasswordEncoder, AuthenticationManagerBuilder 를 주입받는다. 여기서는 단위 테스트이기 때문에 모두 Mock 으로 생성하였다. 

```
given("login 시") {
    `when`("정상적인 정보가 입력되었다면") {
        every { authenticationManagerBuilder.`object`.authenticate(any()) } returns mockk<Authentication>()
        every { userRepository.findByEmail(any()) } returns mockk<User>()
        every { jwtTokenProvider.createToken(any()) } returns TokenInfo("grant", "success")

        then("정상적으로 로그인이 완료된다") {
            val result = target.login(LoginReqDto("test@email.com", "password"))
            result.accessToken shouldBe "success"
        }
    }
    `when`("비정상적인 정보가 입력되었다면") {
        every { authenticationManagerBuilder.`object`.authenticate(any()) } throws BadCredentialsException("Invalid credentials")

        then("Exception 을 발생한다") {
            shouldThrow<AuthenticationException> {
                target.login(LoginReqDto("test@email.com", "password"))
            }
        }
    }
}
```

login 부분의 테스트 코드이다. service 이외의 mockk 으로 생성한 객체의 메서드는 스터빙을 해주어야한다. 

하지만 실제 원하는 테스트는 authenticate() 메서드가 동작해 실제로 로그인이 되는 지 여부이다.

따라서 단위 테스트가 아닌 부분적 통합 테스트를 진행해보았다.

# Service Integration Test
## BehaviorSpec
### Service Test Code 

```
@SpringBootTest
class UserServiceIntegrationTest2 : BehaviorSpec() {

    @MockBean
    private lateinit var userRepository: UserRepository

    @MockBean
    private lateinit var userRoleRepository: UserRoleRepository

    @Autowired
    lateinit var authenticationManagerBuilder: AuthenticationManagerBuilder

    @Autowired
    lateinit var passwordEncoder: PasswordEncoder

    @Autowired
    lateinit var jwtTokenProvider: JwtTokenProvider

    @Autowired
    lateinit var messageSource: MessageSource

    @Autowired
    private lateinit var userService: UserService

    init {
        extension(SpringExtension)

        given("로그인 시도") {
            val userId = 1L
            val expectedUser = User(userId, "test@test.com", passwordEncoder.encode("password123"))
            expectedUser.userRole = listOf(UserRole(1L, Role.MEMBER, expectedUser))

            `when`("사용자가 올바르지 않은 비밀번호로 로그인 시도할 때") {
                val loginReqDto = LoginReqDto("test@test.com", "wrong_password")

                then("로그인이 실패해야 함") {
                    shouldThrow<BadCredentialsException> {
                        userService.login(loginReqDto)
                    }
                }
            }
            `when`("사용자가 올바른 비밀번호로 로그인 시도 시") {
                every { userRepository.findByEmail("test@test.com") } returns expectedUser
                val loginReqDto = LoginReqDto("test@test.com", "password123")

                then("로그인이 성공해야 함") {
                    val result = userService.login(loginReqDto)
                    result.accessToken shouldNotBe null
                }

            }
        }
    }

    companion object {

        ...
    }
}
```

테스트 코드는 단순하다. 로그인 시 실패하면 Exception 을 발생시켜야하고, 성공한다면 TokenInfo 에 토큰이 제대로 담기는지 확인하는 코드이다. 

하지만 로그인 성공 테스트 시 에러가 발생한다.
> Missing calls inside every { ... } block.
io.mockk.MockKException: Missing calls inside every { ... } block.

해당 에러는 every { } 로 설정한 메서드가 실행되지 않았다는 뜻이다. 

검증 과정에서 UserDetailService 에서 findByEmail 을 통해 UserDetails 를 생성하는 과정이 존재하지만 해당 과정의 findByEmail 를 제대로 인식하지 못하는 모양이다..

## FunSpec
### Service Test Code

```
@SpringBootTest
class UserServiceIntegrationTest2 : BehaviorSpec() {

    private var userRepository: UserRepository = mockk()

    private var userRoleRepository: UserRoleRepository = mockk()

    @Autowired
    lateinit var authenticationManagerBuilder: AuthenticationManagerBuilder

    @Autowired
    lateinit var passwordEncoder: PasswordEncoder

    @Autowired
    lateinit var jwtTokenProvider: JwtTokenProvider

    @Autowired
    lateinit var messageSource: MessageSource

    @Autowired
    private lateinit var userService: UserService

    init {
        extension(SpringExtension)

        given("로그인 실패") {
            val userId = 1L
            val expectedUser = User(userId, "test@test.com", passwordEncoder.encode("password123"))
            expectedUser.userRole = listOf(UserRole(1L, Role.MEMBER, expectedUser))

            `when`("사용자가 올바르지 않은 비밀번호로 로그인 시도할 때") {
                val loginReqDto = LoginReqDto("test@test.com", "wrong_password")

                then("로그인이 실패해야 함") {
                    shouldThrow<BadCredentialsException> {
                        userService.login(loginReqDto)
                    }
                }
            }
            `when`("사용자가 올바른 비밀번호로 로그인 시도 시") {
                every { userRepository.findByEmail(any()) } returns User(2L)
                val loginReqDto = LoginReqDto("test@test.com", "password123")

                then("로그인이 성공해야 함") {
                    val result = userService.login(loginReqDto)
                    result.accessToken shouldNotBe null
                }

            }
        }
    }
    ...
```

BDD 가 아닌 FunSpec 으로 테스트를 진행했다.

모두 동일하다고 생각했지만 실행 결과는 달랐다. FunSpec 은 원했던 대로 테스트가 성공적으로 진행되었다. UserDetailService 도 정확하게 실행되어 TokenInfo 를 생성해내는 것을 확인했다.

그렇다면 BDD 와 차이점이 무엇일까??

BDD 는 MockK 를 사용하였고 FunSpec 에서는 Mockito 를 사용하였다. 

사실 검색해도 제대로 된 자료를 찾지 못했지만 둘의 차이가 아닐까 싶다...


# 결론

Repository 와 Service 모두 **단위 테스트** 로 진행하였다. 따라서 Service 에서도 메서드를 통해 통합 테스트를 진행하고 싶더라도 Controller 단에서 통합 테스트를 진행하는 것이 옳아보인다. Service 까지는 단위 테스트로 진행하자!!