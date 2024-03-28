---
title: "[Kotlin + Srping][Kotest BehaviorSpec] Dto Validation 과 Kotest BehaviorSpec 검증"
excerpt: "[Kotlin + Srping][Kotest BehaviorSpec] Dto Validation 과 Kotest BehaviorSpec 검증"

categories:
  - spring
tags:
  - [spring, kotlin]

toc: true
toc_sticky: true

date: 2024-03-28
last_modified_at: 2024-03-28
---

회원가입을 위해 Dto 를 생성 시에 validation 방법과 Kotest 를 사용한 테스트 코드를 작성해보자.

# Dto

```

class UserDtoRequest(
    
    @field:NotBlank
    @field:Email
    @JsonProperty("email")
    private val _email: String?,

    @field:NotBlank
    @field:Pattern(
        regexp = "^(?=.*[a-zA-Z])(?=.*[0-9])(?=.*[!@#\$%^&*])[a-zA-Z0-9!@#\$%^&*]{8,20}\$",
        message = "영문, 숫자, 특수문자를 포함한 8~20자리로 입력해주세요"
    )
    @JsonProperty("password")
    private val _password: String?,

    @field:NotBlank
    @JsonProperty("name")
    private val _name: String?,

    @field:NotBlank
    @field:Pattern(
        regexp = "^01\\d{9}$",
        message = "핸드폰 번호를 다시 확인해주세요"
    )
    @JsonProperty("phone")
    private val _phone: String?,

    ) {

    val password: String
        get() = _password!!
    val name: String
        get() = _name!!

    val email: String
        get() = _email!!

    val phone: String
        get() = _phone!!

    fun toEntity(): User = User(email = email, password = password, name = name, phone = phone)
}

```

작성한 Dto 는 위와 같다. 코틀린에서는 data class 를 통해 간단하게 Dto 를 생성할 수도 있다.

해당 코드에서는 외부에서 Request 를 Dto 에 담아 Validate 후 실제 타입에 담아내는 과정이다. 

검증은 어노테이션을 사용하였으며, 필요 시 정규식으로 검증하였다.

```
    (
        ...

    @field:NotBlank
    @field:Pattern(
        regexp = "^([12]\\d{3}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01]))$",
        message = "날짜형식(YYYY-MM-DD)을 확인해주세요"
    
        ...
    )
    @JsonProperty("birthDate")
    private val _birthDate: String?,

    ){

        ...

    val birthDate: LocalDate
        get() = _birthDate!!.toLocalDate()

    private fun String.toLocalDate(): LocalDate =
        LocalDate.parse(this, DateTimeFormatter.ofPattern("yyyy-MM-dd"))

        ...
    }

```

해당 구조를 사용하는 이유는 타입이 다른 경우 쉽게 확인할 수 있다. 

위와 같은 코드가 추가되었다고 가정해보자. 실제 Json 으로 들어오는 값은 String 이며 이를 검증해야한다. 검증이 끝난 이후에 문제 없이 해당 데이터를 LocalDate 로 변환시킬 수 있기 때문에 _brthDate 를 통해 먼저 검증하는 것이다.

따라서 모든 데이터를 String 타입으로 받아내 Enum 혹은 다른 타입에 맞는 지 검증하고 실제 타입으로 매핑시키는 과정이다.

만약 validation 에 실패한다면 ConstraintViolation 객체에 해당 message 가 담기게 된다.

# Test Code

그렇다면 Dto 검증을 확인할 수 있는 테스트 코드를 작성해보자. 

테스트코드는 Kotest 의 BehaviorSpec 을 따랐다. 

```

class UserDtoTest : BehaviorSpec({
    lateinit var factory: ValidatorFactory
    lateinit var validator: Validator

    beforeSpec {
        factory = Validation.buildDefaultValidatorFactory()
        validator = factory.validator
    }

    ...

```

실제 테스트를 위해 Validator 를 초기화해준다. 이후 validator 를 사용해서 검증했을 때 유효한 값이라면 빈 값이 반환되고 유효하지 않다면 그에 해당하는 ConstraintViolation 객체들을 가지고 있을 것이다. 

```

    Given("UserDtoRequest 를 사용할 때") {
        When("모든 형식이 올바르다면") {
            val rightDto = UserDtoRequest(null, "test@test.com", "Password!123", "name", "01012341234")
            val result: Set<ConstraintViolation<UserDtoRequest>> = validator.validate(rightDto)
            result.isEmpty() shouldBe true
        }


        When("이메일 형식이 올바르지 않다면") {
            val emailDto = UserDtoRequest(null, "test", "Password!123", "name", "01012341234")
            Then("예외가 발생한다.") {
                val result: Set<ConstraintViolation<UserDtoRequest>> = validator.validate(emailDto)
                result.size shouldBe 1
                result.first().message shouldBe "올바른 형식의 이메일 주소여야 합니다"
            }
        }

        When("비밀번호 형식이 올바르지 않다면") {
            val passwordDto = UserDtoRequest(null, "test@test.com", "Password", "name", "01012341234")
            Then("예외가 발생한다.") {
                val result: Set<ConstraintViolation<UserDtoRequest>> = validator.validate(passwordDto)
                result.size shouldBe 1
                result.first().message shouldBe "영문, 숫자, 특수문자를 포함한 8~20자리로 입력해주세요"
            }
        }

        When("전화 번호가 11자리가 아니라면") {
            val phone_number_size_Dto = UserDtoRequest(null, "test@test.com", "Password!123", "name", "0111")
            Then("예외가 발생한다.") {
                val result: Set<ConstraintViolation<UserDtoRequest>> = validator.validate(phone_number_size_Dto)
                result.size shouldBe 1
                result.first().message shouldBe "핸드폰 번호를 다시 확인해주세요"
            }
        }

        When("전화번호 형식이 올바르지 않다면") {
            val valid_phone_number_Dto = UserDtoRequest(null, "test@test.com", "Password!123", "name", "11384728592")
            Then("예외가 발생한다.") {
                val result: Set<ConstraintViolation<UserDtoRequest>> = validator.validate(valid_phone_number_Dto)
                result.size shouldBe 1
                result.first().message shouldBe "핸드폰 번호를 다시 확인해주세요"
            }
        }
    }

})


```

총 5가지의 테스트를 진행했다.

모든 검증을 통과한다면 result 의 사이즈가 0 이다.
이후 이메일, 비밀번호, 전화번호 순서로 검증하며 message 를 통해 테스트를 진행하였다. 

```
result = [ConstraintViolationImpl{interpolatedMessage='올바른 형식의 이메일 주소여야 합니다', propertyPath=_email, rootBeanClass=class com.example.pay.domain.user.dto.UserDtoRequest, messageTemplate='{jakarta.validation.constraints.Email.message}'}]
```
result 에는 다음과 같은 값이 담기게 되는데, propertyPath 를 통해 검증하거나 message 를 하드코딩하는 것이 아니라 파일로 관리하는 것이 더 좋아보인다. 