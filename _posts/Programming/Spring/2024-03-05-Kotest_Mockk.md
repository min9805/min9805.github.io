---
title: "[Spring][Kotlin] 코프링에서 Kotest, MockK 를 사용한 단위 테스트 작성하기"
excerpt: "Kotest, MockK 코프링 단위 테스트"

categories:
  - spring
tags:
  - [spring, kotlin]

toc: true
toc_sticky: true

date: 2024-03-05
last_modified_at: 2024-03-05
---

# 개요

프로젝트를 진행하면서 테스트에 중요성은 항상 강조되었습니다. 하지만 실제 간단한 개발만 진행했다는 이유로 테스트 코드는 등한시했었습니다.

테스트는 코드의 무결성을 보장해주는 최소한의 안전 장치입니다. 코드가 변경되더라도 핵심 로직이 테스트 코드로 작성되어있다면 변경된 기능의 안정성을 신뢰도 높게 검증할 수 있습니다. 

이외에도 TDD 등의 개발 방법론의 장단점은 어디에서나 쉽게 찾아볼 수 있습니다. 본 게시글에서는 Kotlin 에 적합한 테스트 코드 방법을 알아보고 적용시켜보고자 합니다.

# 단위 테스트 

실제 테스트 코드를 살펴보기 앞서 테스트에 대해 살펴보겠습니다. 단위 테스트의 중요성은 강조되지만 학부 수준에서 테스트 코드의 중요성을 실감하기에는 어려움이 있습니다. 

단위 테스트는 코드를 수정하거나 기능 추가 시 수시로 빠르게 검증할 수 있으며 리팩토링 시에 안정성을 확보할 수 있습니다. 하지만 그것보다 큰 장점으로 개발 및 테스팅에 대한 시간과 비용을 절약할 수 있다는 점이 있습니다. 

실제 개발이 끝나고 배포한다고 할 때, 테스트를 작성하지 않은 코드들은 버그가 존재할 확률이 높습니다. 또한 통합 테스트는 캐시, 데이터베이스 등 외부 컴포넌트들과 연결 등의 이유로 비용이 높습니다. 

따라서 개발 및 테스팅에 대한 비용을 줄이기 위해서 "단위" 테스트가 필요합니다.

## FIRST

CleanCode 에 따르면 FIRST 라는 5가지 규칙을 따라야 합니다.

- Fast: 테스트는 빠르게 동작하여 자주 돌릴 수 있어야 한다.
- Independent: 각각의 테스트는 독립적이며 서로 의존해서는 안된다.
- Repeatable: 어느 환경에서도 반복 가능해야 한다.
- Self-Validating: 테스트는 성공 또는 실패로 bool 값으로 결과를 내어 자체적으로 검증되어야 한다.
- Timely: 테스트는 적시에 즉, 테스트하려는 실제 코드를 구현하기 직전에 구현해야 한다.

결국 좋은 테스트란 빠르고 독립적으로 어느 환경에서든 실행 가능하고 검증할 수 있어야합니다. Timely 에 주목하자면 테스트 코드는 실제 코드가 구현되기 전에 작성되어야한다고 합니다. 테스트 코드 먼저 작성하는 개발 방법론은 TDD 라 불리며 이에 대한 장단점은 쉽게 찾아볼 수 있습니다.

결국 강조하고 싶은 것은 "단위" 테스트 라는 것입니다.

# 테스트 코드 

## Java 

기존의 Java 스프링에서는 **Junit** , **Mockito**, **AssertJ** 를 사용해 테스트 코드를 작성했습니다. 

```

@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @InjectMocks
    private UserService userService;

    @Mock
    private UserRepository userRepository;

    @Spy
    private BCryptPasswordEncoder passwordEncoder;

    @DisplayName("회원 가입")
    @Test
    void signUp() {
        // given
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        SignUpRequest request = signUpRequest();
        String encryptedPw = encoder.encode(request.getPw());

        doReturn(new User(request.getEmail(), encryptedPw, UserRole.ROLE_USER)).when(userRepository)
            .save(any(User.class));
            
        // when
        UserResponse user = userService.signUp(request);

        // then
        assertThat(user.getEmail()).isEqualTo(request.getEmail());
        assertThat(encoder.matches(signUpDTO.getPw(), user.getPw())).isTrue();

        // verify
        verify(userRepository, times(1)).save(any(User.class));
        verify(passwordEncoder, times(1)).encode(any(String.class));
    }

    ...
}

```

위 코드는 Java Spring 에서의 일반적인 테스트 코드입니다. ServiceTest 이기 때문에 Service 를 구현하기 위해 필요한 Repository 등은 Mockito 를 사용해 만들어냅니다.

PasswordEncoder 는 @Spy 객체로 선언하였습니다. Spy 에서는 Stub 하지 않은 메소드는 실제 메소드로 동작하게 됩니다. 즉, 해당 객체로 비밀번호를 실제로 암호화할 수 있다는 뜻입니다. 

Mokito 에서는 When 을 사용해 userRepository에서 User 를 저장한다면 미리 지정해놓은 new User 를 반환하겠다고 stub 해놓는 것입니다. 

이후 assertThat 으로 결과값을 검즘함과 동시에 verify 를 톰해 Mockito 로 만들어진 객체의 특정 메소드가 호출된 횟수를 검증할 수 있습니다. 


## Kotlin

Kotlin 은 DSL (Domain Specific Language) 스타일을 가지고 있습니다. 중괄호를 활용한 코드 스타일인데, 코틀린 내부 라이브러리들도 대부분 DSL 을 이용해 작성되었습니다.

하지만 Junit, Mockito 를 사용하면 Mocking 이나 Assertion 과정에서 DSL 을 활용할 수 없습니다. 

따라서 Kotest 와 MockK 를 사용해 기존 DSL 구조를 활용하여 코틀린에서 테스트 코드를 작성할 수 있습니다. 

Kotest 에서는 Junit 과 유사한 "Kotest Annotation Spec" 스타일이 있지만 본 게시글에서는 다루지 않습니다.

### Kotest Behavior Spec

```

private var memberRepository: MemberRepository = mockk(relaxed = true)
private var memberRoleRepository: MemberRoleRepository = mockk(relaxed = true)
private var authenticationManagerBuilder: AuthenticationManagerBuilder = mockk()
private var jwtTokenProvider: JwtTokenProvider = mockk()
private var passwordEncoder: PasswordEncoder = mockk(relaxed = true)


@InjectMockKs
var memberService: MemberService = MemberService(
    memberRepository,
    memberRoleRepository,
    authenticationManagerBuilder,
    jwtTokenProvider,
    passwordEncoder
)


class MemberServiceTest : BehaviorSpec({

    Given("회원가입 테스트") {
        val memberSignupDto = MemberDtoRequest(
            id = null,
            _loginId = "테스트_로그인_아이디",
            _password = "password123!",
            _name = "테스트_이름",
            _birthDate = LocalDate.now().toString(),
            _gender = Gender.MAN.toString(),
            _email = "test@example.com",
        )
        val member = Member(
            id = 1L,
            loginId = "테스트_아이디",
            password = "테스트_비밀번호",
            name = "테스트_이름",
            birthDate = LocalDate.now(),
            gender = Gender.MAN,
            email = "테스트_이메일",
        )
        val memberRole = MemberRole(role = ROLE.MEMBER, member = member)

        every { memberRepository.findByLoginId("테스트_로그인_아이디") } returns null
        every { memberRepository.save(any()) } returns member
        every { memberRoleRepository.save(any()) } returns memberRole


        When("유저 회원 가입을 진행하면") {
            val result = memberService.signUp(memberSignupDto)
            Then("정상적으로 회원 가입이 진행되어야한다.") {

                result should be("회원가입이 완료되었습니다.")
                //                val savedMember = memberRepository.findByLoginId("테스트_로그인_아이디")
//                assertNotNull(savedMember)
//                assertEquals("테스트_로그인_아이디", savedMember!!.loginId)
//                assertEquals("테스트_이름", savedMember.name)
//                assertEquals("test@example.com", savedMember.email)
            }
        }
    }
})

```

기존의 Given, When, Then 패턴을 해당 Spec 을 사용해 간결하게 정의할 수 있습니다. 

![image](https://github.com/min9805/min9805.github.io/assets/56664567/384b31e9-304e-4dd9-ab5b-515b7090cf77)

memberService 에 필요한 정보들은 MockK 으로 생성하고 필요한 메소드들을 stub 합니다. 
이때 해당 테스트는 memberService 에 대한 "단위" 테스트 이므로 이하 객체들은 Mock 을 생성하며, 메소드의 동작들도 미리 stub 함을 잘 이해해야 합니다. 

### Kotest Describe Spec

Kotest 는 DCI(Describe, Context, It) 패턴도 제공합니다. 

```

internal class CalculatorDescribeSpec : DescribeSpec({
    val sut = Calculator()

    describe("calculate") {
        context("식이 주어지면") {
            it("해당 식에 대한 결과값이 반환된다") {
                calculations.forAll { (expression, data) ->
                    val result = sut.calculate(expression)

                    result shouldBe data
                }
            }
        }


```

# 마치며 

Kotlin 뿐만 아니라 테스트 코드에 생소하기에 어려움이 있었습니다.
하지만 Kotest + MockK 는 다소 직관적인 테스트 코드이며 "단위" 테스트임을 이해할 수 있었습니다. 