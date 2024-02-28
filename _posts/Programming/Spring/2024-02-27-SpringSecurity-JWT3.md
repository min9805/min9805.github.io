---
title: "[SpringSecurity + JWT] 03. JWTTokenProvider"
excerpt: "미니프로젝트를 위한 SpringSecurity, JWT"

categories:
  - spring
tags:
  - [spring, jwt]

toc: true
toc_sticky: true

date: 2024-02-27
last_modified_at: 2024-02-27
---

 JWT 토큰을 생성하는데 필요한 UserDetails 가 준비되었기 때문에 이제 JWT 토큰을 실제 생성하는 JWTTokenProvider 를 생성합니다.

Spring Security 의 인증 절차는 UsernamePasswordAuthenticationFilter 에서 이루어집니다. 

해당 필터에는 doFilter() 와 attemptAuthentication() 이라는 추상 메서드가 존재하는데, 실제 인증 절차는 attemptAuthentication() 에 존재하는 것을 실제 확인할 수 있습니다. 

```

	@Override
	public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
			throws AuthenticationException {
		if (this.postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
		}
		String username = obtainUsername(request);
		username = (username != null) ? username.trim() : "";
		String password = obtainPassword(request);
		password = (password != null) ? password : "";
		UsernamePasswordAuthenticationToken authRequest = UsernamePasswordAuthenticationToken.unauthenticated(username,
				password);
		// Allow subclasses to set the "details" property
		setDetails(request, authRequest);
		return this.getAuthenticationManager().authenticate(authRequest);
	}

```

여기서 확인해야할 것은 UsernamePasswordAuthenticationToken 입니다. 


```

public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {

public abstract class AbstractAuthenticationToken implements Authentication, CredentialsContainer {


```

한번 더 확인하면 UsernamePasswordAuthenticationToken -> AbstractAuthenticationToken  -> Authentication 으로 상속받는 것을 볼 수 있습니다. 

즉, UsernamePasswordAuthenticationToken 는 인증 과정 후 SecurityContextHolder.getContext()에 등록될 Authentication 객체입니다.

그리고 이 객체를 JWTTokenProvier 에서 UserDetails 와 함께 생성해냅니다.

# JWTTokenProvider

```

/**
 * 토큰 관련 기능
 * <p>
 * - 토큰 생성
 * - 토큰 해석
 * - 토큰 유효성 검사
 */
@Slf4j
@Component
public class JwtTokenProvider {

    @Autowired
    private JwtProp jwtProp;
    @Autowired
    private UserRepository userRepository;

    public String createToken(Long uid, String username, List<String> roles) {
        byte[] signingKey = getSigningKey();

        return Jwts.builder()
                .signWith(getShaKey(signingKey), Jwts.SIG.HS256)
                .header().add("typ", JwtConstants.TOKEN_TYPE)
                .and()
                .expiration(new Date(System.currentTimeMillis() + JwtConstants.TOKEN_EXPIRATION))
                .claim("uid", uid)
                .claim("rol", roles)
                .claim("username", username)
                .compact();
    }


    @Transactional
    public UsernamePasswordAuthenticationToken getAuthentication(String header) {
        if (header == null || header.isEmpty()) {
            return null;
        }

        try {

            String jwt = header.replace(JwtConstants.TOKEN_PREFIX, "").trim();

            Jws<Claims> parsedToken = Jwts.parser()
                    .verifyWith(getShaKey(getSigningKey()))
                    .build()
                    .parseSignedClaims(jwt);

            Claims claims = parsedToken.getPayload();

            Object roles = claims.get("rol");
            log.info("roles :" + roles);

            Long uid = (Long) claims.get("uid");
            log.info("uid : " + uid);

            String username = claims.get("username").toString();
            log.info("username : " + username);

            if (username == null || username.isEmpty()) {
                return null;
            }

            User user = User.builder()
                    .id(uid)
                    .name(username)
                    .build();

            List<UserAuth> authList = ((List<UserAuth>) roles).stream()
                    .map(auth -> new UserAuth(auth.toString(), user))
                    .toList();

            //CustomUser 에 권한 담기
            List<SimpleGrantedAuthority> authorities = ((List<UserAuth>) roles).stream()
                    .map(auth -> new SimpleGrantedAuthority(auth.getAuth()))
                    .toList();

            try {
                User userInfo = userRepository.findById(uid).orElse(null);
                if (userInfo != null) {
                    user.toBuilder()
                            .email(userInfo.getEmail())
                            .build();
                }
            } catch (Exception e) {
                log.error(e.getMessage());
                log.error("토큰 유효 -> DB 추가 정보 조회 시 에러 발생");
            }

            UserDetails userDetails = new CustomUser(user);

            return new UsernamePasswordAuthenticationToken(userDetails, null, authorities);

        } catch (ExpiredJwtException exception) {
            log.warn("Request to parse expired JWT : {} failed : {}", header, exception.getMessage());
        } catch (UnsupportedJwtException exception) {
            log.warn("Request to parse unsupported JWT : {} failed : {}", header, exception.getMessage());
        } catch (MalformedJwtException exception) {
            log.warn("Request to parse invalid JWT : {} failed : {}", header, exception.getMessage());
        } catch (IllegalArgumentException exception) {
            log.warn("Request to parse empty or null JWT : {} failed : {}", header, exception.getMessage());
        }
        return null;
    }

    /**
     * 토큰 유효성 검사 (만료기간 검사)
     */
    public boolean validateToken(String jwt) {
        try {


            Jws<Claims> parsedToken = Jwts.parser()
                    .verifyWith(getShaKey(getSigningKey()))
                    .build()
                    .parseSignedClaims(jwt);

            log.info("#### 토큰 만료 기간 ####");
            Date expiration = parsedToken.getPayload().getExpiration();
            log.info("-> " + expiration);

            return expiration.after(new Date());
        } catch (ExpiredJwtException exception) {
            log.warn("Request to parse expired JWT failed : {}", exception.getMessage());
            return false;
        } catch (JwtException exception) {
            log.warn("Request to parse tampered JWT failed : {}", exception.getMessage());
            return false;
        } catch (NullPointerException exception) {
            log.warn("Request to parse null JWT failed : {}", exception.getMessage());
            return false;
        }
    }

    private static SecretKey getShaKey(byte[] signingKey) {
        return Keys.hmacShaKeyFor(signingKey);
    }

    private byte[] getSigningKey() {
        return jwtProp.getSecretKey().getBytes();
    }
}

```

JWTTokenProvider 는 3가지 메서드를 가지고 있습니다.

  - 토큰 생성
  - 토큰 해석
  - 토큰 유효성 검사
 

## CreateToken

```

public String createToken(Long uid, String username, List<String> roles) {
        byte[] signingKey = getSigningKey();

        return Jwts.builder()
                .signWith(getShaKey(signingKey), Jwts.SIG.HS256)
                .header().add("typ", JwtConstants.TOKEN_TYPE)
                .and()
                .expiration(new Date(System.currentTimeMillis() + JwtConstants.TOKEN_EXPIRATION))
                .claim("uid", uid)
                .claim("rol", roles)
                .claim("username", username)
                .compact();
    }

```

토큰 생성과정은 간단합니다. 미리 정해놓은 secretKey 를 가지고 서명을 진행하고 헤더와 claim 을 추가해 Jwts 의 빌더로 생성하면 됩니다. 

## getAuthentication

```

public UsernamePasswordAuthenticationToken getAuthentication(String header) {
        if (header == null || header.isEmpty()) {
            return null;
        }

        try {

            String jwt = header.replace(JwtConstants.TOKEN_PREFIX, "").trim();

            Jws<Claims> parsedToken = Jwts.parser()
                    .verifyWith(getShaKey(getSigningKey()))
                    .build()
                    .parseSignedClaims(jwt);

            Claims claims = parsedToken.getPayload();

            Object roles = claims.get("rol");
            log.info("roles :" + roles);

            Long uid = (Long) claims.get("uid");
            log.info("uid : " + uid);

            String username = claims.get("username").toString();
            log.info("username : " + username);

            if (username == null || username.isEmpty()) {
                return null;
            }

            User user = User.builder()
                    .id(uid)
                    .name(username)
                    .build();

            List<UserAuth> authList = ((List<UserAuth>) roles).stream()
                    .map(auth -> new UserAuth(auth.toString(), user))
                    .toList();

            //CustomUser 에 권한 담기
            List<SimpleGrantedAuthority> authorities = ((List<UserAuth>) roles).stream()
                    .map(auth -> new SimpleGrantedAuthority(auth.getAuth()))
                    .toList();

            try {
                User userInfo = userRepository.findById(uid).orElse(null);
                if (userInfo != null) {
                    user.toBuilder()
                            .email(userInfo.getEmail())
                            .build();
                }
            } catch (Exception e) {
                log.error(e.getMessage());
                log.error("토큰 유효 -> DB 추가 정보 조회 시 에러 발생");
            }

            UserDetails userDetails = new CustomUser(user);

            return new UsernamePasswordAuthenticationToken(userDetails, null, authorities);

        } catch (ExpiredJwtException exception) {
            log.warn("Request to parse expired JWT : {} failed : {}", header, exception.getMessage());
        } catch (UnsupportedJwtException exception) {
            log.warn("Request to parse unsupported JWT : {} failed : {}", header, exception.getMessage());
        } catch (MalformedJwtException exception) {
            log.warn("Request to parse invalid JWT : {} failed : {}", header, exception.getMessage());
        } catch (IllegalArgumentException exception) {
            log.warn("Request to parse empty or null JWT : {} failed : {}", header, exception.getMessage());
        }
        return null;
    }

```

여기서는 클라이언트에서 헤더와 함께 들어오는 JWT 를 해석하는 메서드입니다.

JWT 를 parse 하여 내부 정보를 바로 취득할 수 있습니다.

이후 JWT 에 담긴 정보를 토대로 실제 User 의 데이터를 추가해 UserDetails 를 생성해냅니다. 

이후 UsernamePasswordAuthenticationToken(userDetails, null, authorities);

를 생성해냅니다.해당 토큰은 이후 인증이 끝나고 
SecurityContextHolder.getContext() 에 등록될 Authentication 객체입니다.

## ValidateToken

```

public boolean validateToken(String jwt) {
        try {


            Jws<Claims> parsedToken = Jwts.parser()
                    .verifyWith(getShaKey(getSigningKey()))
                    .build()
                    .parseSignedClaims(jwt);

            log.info("#### 토큰 만료 기간 ####");
            Date expiration = parsedToken.getPayload().getExpiration();
            log.info("-> " + expiration);

            return expiration.after(new Date());
        } catch (ExpiredJwtException exception) {
            log.warn("Request to parse expired JWT failed : {}", exception.getMessage());
            return false;
        } catch (JwtException exception) {
            log.warn("Request to parse tampered JWT failed : {}", exception.getMessage());
            return false;
        } catch (NullPointerException exception) {
            log.warn("Request to parse null JWT failed : {}", exception.getMessage());
            return false;
        }
    }

```

Token 의 Expiration 을 검사하는 메서드입니다.

토큰을 동일하게 해석하고 expiration 을 검사합니다. 

다음에는 실제 Provider 를 사용하는 Filter 에 대해서 알아보겠습니다.