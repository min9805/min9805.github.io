---
title: "[SpringSecurity + JWT] 06. BaseResponse, ExceptionHandler"
excerpt: "미니프로젝트를 위한 SpringSecurity, JWT"

categories:
  - spring
tags:
  - [spring, jwt]

toc: true
toc_sticky: true

date: 2024-02-29
last_modified_at: 2024-02-29
---

<details markdown="1" style="border: 1px solid black; padding: 10px; background-color: #333333; border-radius: 10px;">
  <summary style="font-size: 35px;">JWT Series</summary>
  <div>
    <li>
        <a href="https://min9805.github.io/spring/SpringSecurity-JWT1/">[SpringSecurity + JWT] 01. JWT?</a>
    </li>
        <li>
        <a href="https://min9805.github.io/spring/SpringSecurity-JWT2/">[SpringSecurity + JWT] 02. CustomUserDetails</a>
    </li>
        <li>
        <a href="https://min9805.github.io/spring/SpringSecurity-JWT3/">[SpringSecurity + JWT] 03. JWTTokenProvider</a>
    </li>
        <li>
        <a href="https://min9805.github.io/spring/SpringSecurity-JWT4/">[SpringSecurity + JWT] 04. AuthenticationFilter</a>
    </li>
        <li>
        <a href="https://min9805.github.io/spring/SpringSecurity-JWT5/">[SpringSecurity + JWT] 05. Kotlin Code</a>
    </li>
        <li>
        <a href="https://min9805.github.io/spring/SpringSecurity-JWT6/">[SpringSecurity + JWT] 06. BaseResponse, ExceptionHandler</a>
    </li>
  </div>
</details>


----


추가 작업으로 일정한 Response 형식을 반환하고 ExceptionHandler 를 추가하겠습니다.

# BaseResponse

```
data class BaseResponse<T>(
    val resultCode: String = ResultCode.SUCCESS.name,
    val data: T? = null,
    val message: String = ResultCode.SUCCESS.msg,
)
```

응답은 다음과 같이 나타낼 것입니다. Data 와 message 를 분리하고 Exception 에 Custom message 가 정확하게 뜨는 것을 예상합니다.

# ExceptionHandler

```

@RestControllerAdvice
class CustomExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException::class)
    protected fun methodArgumentNotValidException(ex: MethodArgumentNotValidException): ResponseEntity<BaseResponse<Map<String, String>>> {
        val errors = mutableMapOf<String, String>()
        ex.bindingResult.allErrors.forEach { error ->
            val fieldName = (error as FieldError).field
            val errorMessage = error.defaultMessage
            errors[fieldName] = errorMessage ?: "Not Exception Message"
        }

        return ResponseEntity(BaseResponse(ResultCode.ERROR.name, errors, ResultCode.ERROR.msg), HttpStatus.BAD_REQUEST)
    }

    @ExceptionHandler(InvalidInputException::class)
    protected fun invalidInputException(ex: InvalidInputException): ResponseEntity<BaseResponse<Map<String, String>>> {
        val errors = mapOf(ex.fieldName to (ex.message ?: "Not Exception Message"))
        return ResponseEntity(BaseResponse(ResultCode.ERROR.name, errors, ResultCode.ERROR.msg), HttpStatus.BAD_REQUEST)
    }

    @ExceptionHandler(BadCredentialsException::class)
    protected fun badCredentialsException(ex: BadCredentialsException): ResponseEntity<BaseResponse<Map<String, String>>> {
        val errors = mapOf("로그인 실패" to "아이디 혹은 비밀번호를 다시 확인하세요.")
        return ResponseEntity(BaseResponse(ResultCode.ERROR.name, errors, ResultCode.ERROR.msg), HttpStatus.BAD_REQUEST)
    }

    @ExceptionHandler(Exception::class)
    protected fun defaultException(ex: Exception): ResponseEntity<BaseResponse<Map<String, String>>> {
        val errors = mapOf("미처리 에러" to (ex.message ?: "Not Exception Message"))
        return ResponseEntity(BaseResponse(ResultCode.ERROR.name, errors, ResultCode.ERROR.msg), HttpStatus.BAD_REQUEST)
    }
}

```

ExceptionHandler 는 다음과 같습니다. 필요에 따른 Exception 을 처리하고 메세지 등을 BaseResponse 에 담아 깔끔하게 처리합니다. 

해당 클래스에서는 작성된 순서대로 처리하기 때문에 Exception 클래스를 처리하는 핸들러는 꼭 마지막에 작성되어야 위의 예외들을 처리하지 못한 나머지 예외를 처리할 수 있습니다. 


![image](https://github.com/min9805/min9805.github.io/assets/56664567/9976e2db-deb7-422b-a17c-10f7842898aa)


위와 같이 잘못된 요청이 들어올 경우 설정해놓은 메세지로 빠르게 에러를 파악할 수 있습니다.