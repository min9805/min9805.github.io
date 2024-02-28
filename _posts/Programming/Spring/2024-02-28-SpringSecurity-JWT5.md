---
title: "[SpringSecurity + JWT] 04. Kotlin Code"
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

 구조를 우선적으로 파악한 이후 Kotlin 으로 컴팩트하게 JWT Token 처리를 진행해보겠습니다. 


 # CustomUserDetails, CustomUserDetailsService

```

class CustomUser(
    val userId: Long,
    userName: String,
    password: String,
    authorities: Collection<GrantedAuthority>
) : User(userName, password, authorities)

```

 ```

@Service
class CustomUserDetailsService(
    private val memberRepository: MemberRepository,
) : UserDetailsService {
    override fun loadUserByUsername(username: String): UserDetails =
        memberRepository.findByLoginId(username)
            ?.let { createUserDetails(it) } ?: throw UsernameNotFoundException("해당 유저는 없습니다.")


    private fun createUserDetails(member: Member): UserDetails =
        CustomUser(
            member.id!!,
            member.loginId,
            member.password,
            member.memberRole!!.map { SimpleGrantedAuthority("ROLE_${it.role}") }
        )
}

 ```

userId 를 추가하기 위해서 CustomUser 는 User 를 상속받습니다.
이때 USer 는 org.springframework.security.core.userdetails.User 입니다.

CustomUserDetailsService 에서도 역시 UserDetailService 를 상속받습니다.

로그인의 핵심 로직은 loadUserByUsername 이며 DB 에서 데이터를 가져와 비교할 준비를 합니다. 
Repository 를 통해 가져온 member 가 UserDetails 로 변환되는 것이 위 과정입니다.

여기서 주의해야할 점은 비교 시 입력된 password 에 PasswordEncoder 의 encode() 가 강제로 처리되기에 DB 에서 가져오는 password 는 encode() 가 된 상태이어야합니다.

# JWTTokenProvider

```

@Component
class JwtTokenProvider {
    @Value("\${jwt.secretKey}")
    lateinit var secretKey: String

    private val key by lazy { Keys.hmacShaKeyFor(Decoders.BASE64.decode(secretKey)) }

    /**
     * Token 생성
     */
    fun createToken(authentication: Authentication): TokenInfo {
        val authorities: String = authentication
            .authorities
            .joinToString(",", transform = GrantedAuthority::getAuthority)

        val now = Date()
        val accessExpiration = Date(now.time + EXPIRATION_MILLISECONDS)

        // Access Token
        val accessToken = Jwts
            .builder()
            .setSubject(authentication.name)
            .claim("auth", authorities)
            .claim("userId", (authentication.principal as CustomUser).userId)
            .setIssuedAt(now)
            .setExpiration(accessExpiration)
            .signWith(key, SignatureAlgorithm.HS256)
            .compact()

        return TokenInfo("Bearer", accessToken)
    }

    /**
     * Token 정보 추출
     */
    fun getAuthentication(token: String): Authentication {
        val claims: Claims = getClaims(token)

        val auth = claims["auth"] ?: throw RuntimeException("잘못된 토큰입니다.")
        val userId = claims["userId"] ?: throw RuntimeException("잘못된 토큰입니다.")

        // 권한 정보 추출
        val authorities: Collection<GrantedAuthority> = (auth as String)
            .split(",")
            .map { SimpleGrantedAuthority(it) }

        val principal: UserDetails = CustomUser(userId.toString().toLong(), claims.subject, "", authorities)

        return UsernamePasswordAuthenticationToken(principal, "", authorities)
    }

    /**
     * Token 검증
     */
    fun validateToken(token: String): Boolean {
        try {
            getClaims(token)
            return true
        } catch (e: Exception) {
            when (e) {
                is SecurityException -> {}  // Invalid JWT Token
                is MalformedJwtException -> {}  // Invalid JWT Token
                is ExpiredJwtException -> {}    // Expired JWT Token
                is UnsupportedJwtException -> {}    // Unsupported JWT Token
                is IllegalArgumentException -> {}   // JWT claims string is empty
                else -> {}  // else
            }
            println(e.message)
        }
        return false
    }

    private fun getClaims(token: String): Claims =
        Jwts.parserBuilder()
            .setSigningKey(key)
            .build()
            .parseClaimsJws(token)
            .body
}

```

JWTTokenProvider 는 역시 생성, 정보 추출, 검증 3가지 메서드를 가지고 있습니다. 



## createToken

```

fun createToken(authentication: Authentication): TokenInfo {
        val authorities: String = authentication
            .authorities
            .joinToString(",", transform = GrantedAuthority::getAuthority)

        val now = Date()
        val accessExpiration = Date(now.time + EXPIRATION_MILLISECONDS)

        // Access Token
        val accessToken = Jwts
            .builder()
            .setSubject(authentication.name)
            .claim("auth", authorities)
            .claim("userId", (authentication.principal as CustomUser).userId)
            .setIssuedAt(now)
            .setExpiration(accessExpiration)
            .signWith(key, SignatureAlgorithm.HS256)
            .compact()

        return TokenInfo("Bearer", accessToken)
    }

```

Token 생성 시에는 Authentication 을 전달받습니다.

```
    fun login(loginDto: LoginDto): TokenInfo {
        val authenticationToken = UsernamePasswordAuthenticationToken(loginDto.loginId, loginDto.password)
        val authentication = authenticationManagerBuilder.`object`.authenticate(authenticationToken)

        return jwtTokenProvider.createToken(authentication)
    }
```

전달은 해당 login 함수를 통해 이루어지는데, authenticationManagerBuilder 를 통해 authenticate 를 바로 진행합니다. 여기서 만약 로그인이 정보가 틀리다면 
AuthenticationException 이 발생합니다.

createToken() 이 실행되는 것이 이미 로그인 검증이 되었다는 뜻입니다. 

Jwt 를 생성하는 과정은 이전과 동일하게 유효 기간과 필요한 정보들을 담는 과정을 가지고 있습니다. 

## getAuthentication

```

    fun getAuthentication(token: String): Authentication {
        val claims: Claims = getClaims(token)

        val auth = claims["auth"] ?: throw RuntimeException("잘못된 토큰입니다.")
        val userId = claims["userId"] ?: throw RuntimeException("잘못된 토큰입니다.")

        // 권한 정보 추출
        val authorities: Collection<GrantedAuthority> = (auth as String)
            .split(",")
            .map { SimpleGrantedAuthority(it) }

        val principal: UserDetails = CustomUser(userId.toString().toLong(), claims.subject, "", authorities)

        return UsernamePasswordAuthenticationToken(principal, "", authorities)
    }

```

JWT 해석 또한 간단합니다. Claims 를 추출해내고 권한을 바로 추출해낼 수 있습니다. 해당 내용을 CustomUser 로 변환시켜 인증 과정이 끝난 후 

SecurityContextHolder.getContext().authentication = authentication

로 저장시키기 위해 UsernamePasswordAuthenticationToken 를 반환합니다.

## validateToken()

```

fun validateToken(token: String): Boolean {
  try {
      getClaims(token)
      return true
  } catch (e: Exception) {
      when (e) {
          is SecurityException -> {}  // Invalid JWT Token
          is MalformedJwtException -> {}  // Invalid JWT Token
          is ExpiredJwtException -> {}    // Expired JWT Token
          is UnsupportedJwtException -> {}    // Unsupported JWT Token
          is IllegalArgumentException -> {}   // JWT claims string is empty
          else -> {}  // else
      }
      println(e.message)
  }
  return false
}

private fun getClaims(token: String): Claims =
Jwts.parserBuilder()
    .setSigningKey(key)
    .build()
    .parseClaimsJws(token)
    .body

```

해당 메서드에서는 유효성을 확인합니다. 토큰의 내용을 해독하고 정상적으로 parse 된다면 통과입니다. 이외에는 Exception 을 발생시킵니다. 

실제 Jwt 를 손상시켰을 때는 SignatureException 이 발생하며 catch 에 걸리지 않기 때문에 false 가 반환됩니다.


# JwtAuthenticationFilter

```

class JwtAuthenticationFilter(
    private val jwtTokenProvider: JwtTokenProvider
) : GenericFilterBean() {

    override fun doFilter(request: ServletRequest?, response: ServletResponse?, chain: FilterChain?) {
        val token = resolveToken(request as HttpServletRequest)

        if (token != null && jwtTokenProvider.validateToken(token)) {
            val authentication = jwtTokenProvider.getAuthentication(token)
            SecurityContextHolder.getContext().authentication = authentication
        }

        chain?.doFilter(request, response)
    }

    private fun resolveToken(request: HttpServletRequest): String? {
        val bearerToken = request.getHeader("Authorization")

        return if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer")) {
            bearerToken.substring(7)
        } else {
            null
        }
    }
}

```

JWT 를 검증할 Filter 입니다. 

doFilter 를 사용해 유효성검사를 진행합니다.

만약 jwt 가 아직 null 이거나 훼손되어있다면 다음 filter 로 넘어갑니다.

jwt 가 정상적임을 확인했다면 Token 의 정보를 추출하고 SpringSecurity Context 에 저장합니다.

## SecurityConfig

```
@Bean
fun filterChain(http: HttpSecurity): SecurityFilterChain {

  ...

    .addFilterBefore(
        JwtAuthenticationFilter(jwtTokenProvider),
        UsernamePasswordAuthenticationFilter::class.java
    )
    
```

필터 역시 등록해야합니다. 



![image](https://github.com/min9805/min9805.github.io/assets/56664567/120ed341-b4fa-4a9d-8a50-2fd2e5e5693e)

정상적으로 잘 동작하는 것을 확인했습니다. 

```
data class TokenInfo(
    val grantType: String,
    val accessToken: String,
)
```

추가적으로 Response 를 TokenInfo 에 담아서 보냈습니다.

다음에는 응답 타입을 통일시켜보도록 하겠습니다! 