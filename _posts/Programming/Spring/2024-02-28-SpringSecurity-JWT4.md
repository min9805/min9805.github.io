---
title: "[SpringSecurity + JWT] 04. AuthenticationFilter"
excerpt: "미니프로젝트를 위한 SpringSecurity, JWT"

categories:
  - spring
tags:
  - [spring, jwt]

toc: true
toc_sticky: true

date: 2024-02-28
last_modified_at: 2024-02-28
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
JWT 인증은 실제로 Filtter 를 통해 이루어집니다. 
JWTTokenProvider 에서 말했듯이 결국 인증은 UsernamePasswordAuthenticationFilter 에서 이루어집니다.

또한 AuthenticationManager 에서 CustomUserDetailsService 로직을 실행하게 됩니다.

# JwtAuthenticationFilter

```

public class JwtAuthenticationFilter extends UsernamePasswordAuthenticationFilter {

    private final AuthenticationManager authenticationManager;
    private final JwtTokenProvider jwtTokenProvider;

    public JwtAuthenticationFilter(AuthenticationManager authenticationManager, JwtTokenProvider jwtTokenProvider) {
        this.authenticationManager = authenticationManager;
        this.jwtTokenProvider = jwtTokenProvider;
        setFilterProcessesUrl(JwtConstants.AUTH_LOGIN_URL);
    }

    /**
     * 인증 시도 메서드
     */
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        String username = request.getParameter("username");
        String password = request.getParameter("password");

        // 사용자 인증정보 객체 생성
        Authentication authentication = new UsernamePasswordAuthenticationToken(username, password);
        log.info("인증 정보 객체 : " + authentication);

        // 사용자 인증 (로그인)
        // UserDetailsService 의 Custom 로직 실행
        // PasswordEncoder 에 따라 암호화
        Authentication authenticate = authenticationManager.authenticate(authentication);

        log.info("인증 여부 : " + authenticate.isAuthenticated());

        // 인증 실패 (username, password 불일치)
        if (!authenticate.isAuthenticated()) {
            log.info("인증 실패 : 아이디 또는 비밀번호가 일치하지 않습니다.");
            response.setStatus(401);
        }

        //인증 성공 시 successfulAuthentication 호출
        return authenticate;
    }

    /***
     *  인증 성공 메서드
     *
     *  - JWT 생성
     *  - JWT 를 응답 헤더에 설정
     */
    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication) throws IOException, ServletException {

        log.info("successfulAuthentication : " + request);

        CustomUser user = (CustomUser) authentication.getPrincipal();
        Long userId = user.getUser().getId();
        String username = user.getUsername();

        List<String> roles = user.getAuthorities()
                .stream()
                .map(auth -> auth.toString())
                .toList();

        String jwt = jwtTokenProvider.createToken(userId, username, roles);

        response.addHeader(JwtConstants.TOKEN_HEADER, JwtConstants.TOKEN_PREFIX + jwt);
        response.setStatus(200);

    }
}


```

JwtAuthenticationFilter 는 UsernamePasswordAuthenticationFilter 를 상속받습니다. 

그리고 attemptAuthentication 메서드를 통해 인증 시도를 하고 만약 인증이 정상적으로 진행되었다면 ( authenticate.isAuthenticated() == true ) 다음 successfulAuthentication 메서드를 처리시킵니다. 

내부의 로직은 생각보다 간단합니다. 받은 username, password 를 통해 인증 메서드만 실행시키면 됩니다. 이후에는 이전에 설정한 로직에 따라 실행됩니다. 

성공 시에는 JWT 토큰을 발행하고 헤더를 통해 넘겨줍니다.

## Autowired

이를 위해서 authenticationManager 와 jwtTokenProvider 를 주입시켜야합니다.

하지만 필터에서 직접 주입시키지는 못하고 생성자를 통해 주입되어야합니다. 

```

@Autowired
private JwtTokenProvider jwtTokenProvider;

private AuthenticationManager authenticationManager;

@Bean("AuthenticationManager")
public AuthenticationManager authenticationManager(AuthenticationConfiguration authenticationConfiguration) throws Exception {
    this.authenticationManager = authenticationConfiguration.getAuthenticationManager();
    return authenticationManager;
}

...


@DependsOn(value = "AuthenticationManager")
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    ...

    //필터 설정
    http.addFilterAt(new JwtAuthenticationFilter(authenticationManager, jwtTokenProvider), UsernamePasswordAuthenticationFilter.class)
            .addFilterBefore(new JwtRequestFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class);

...
}

```

여기서 잘못하면 순환 참조 오류가 발생하기도 하며 실제 실행했을 때는 생성자로 주입된 authenticationManager is null 이 계속 발생했습니다.

디버깅 결과 AuthenticationManager 가 초기화되기 전에 SecurityFilterChain 가 먼저 실행되어 null 값이 들어갔습니다.

따라서 DependsOn 어노테이션을 활용해 Bean 의 순서를 지정해주어야 에러 없이 주입할 수 있습니다. 