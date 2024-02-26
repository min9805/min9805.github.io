---
title: "[SpringSecurity + JWT] 01. JWT?"
excerpt: "미니프로젝트를 위한 SpringSecurity, JWT"

categories:
  - spring
tags:
  - [spring, resttemplate]

toc: true
toc_sticky: true

date: 2024-02-26
last_modified_at: 2024-02-26
---

JWT (Json Web Token) 은 현재 대부분의 인증 허가 관련해서 사용되는 기술입니다. 필요한 모든 정보를 한 객체에 담아서 전달하기에 JWT 하나로 많은 인증이 가능합니다. 또한 웹 표준을 따르기에 대부분의 언어가 이를 지원합니다. 

# JWT 구조

![image](https://github.com/min9805/min9805.github.io/assets/56664567/628074ba-67cf-4425-920a-e8e0de2e6d2a)

 JWT 는 헤더, 페이로드, 시그니처가 "." 를 구분자로 하나의 토큰을 이룹니다. 

[JWT 디버거](https://jwt.io/)

해당 사이트에서 실제 JWT 와 해석된 형태를 확인해볼 수 있습니다. 

다음부터는 가장 간단한 JWT 예시를 들어 설명하겠습니다.

# JWT 생성

```
TOKEN_TYPE = "JWT"

@Autowired
private JwtProp jwtProp;

@PostMapping("login")
public ResponseEntity<?> login(@RequestBody AuthenticationRequest request) {
	String username = request.getUsername();
	String password = request.getPassword();

	//로그인 처리
	...

	//사용자 권한
	List<String> roles = new ArrayList<>();
	roles.add("ROLE_USER");
	roles.add("ROLE_ADMIN");

	byte[] signingKey = jwtProp.getSecretKey().getBytes();

	String jwt = Jwts.builder()
			.signWith(Keys.hmacShaKeyFor(signingKey), Jwts.SIG.HS256)
			.header()
			.add("typ", TOKEN_TYPE)
			.and()
			.expiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 24 * 5))
			.claim("uid", username)
			.claim("rol", roles)
			.compact();

	return new ResponseEntity<String>(jwt, HttpStatus.OK);
}

```

Login 시 JWT 를 생성하는 가장 간단한 예제입니다.

- JwtProp 는 application.yml 요소를 관리하는 클래스입니다.

## JWT 생성 과정

1. username, password 를 받으면 일련의 로그인 처리를 진행합니다. 여기서는 생략하였습니다. 
2. 로그인 성공 시 username, role 등을 가져옵니다.
3. 사전에 생성한 secretKey 를 signingKey 로 가져옵니다.

JWT 생성을 위한 재료가 준비되었으니 본격적으로 JWT 를 생성합니다.

- Jwts 라이브러리를 사용하여 build 합니다.
- Keys.hmacShaKeyFor() 을 사용해 secretKey 와 사용할 알고리즘을 함께 등록합니다.
- 헤더에 토큰 타입 등을 추가합니다.
- 만료 기한을 설정합니다. 여기에서는 5일의 만료 기한을 두었습니다.
- Payload 에 들어갈 정보들을 Claim 형태로 저장합니다.

위 과정을 통해 

![image](https://github.com/min9805/min9805.github.io/assets/56664567/a7c0a1c9-bcfe-4df0-b59e-3abb5993d631)

실제 JWT 를 생성하고 해당 정보들을 확인할 수 있습니다.

# JWT 토큰 확인

```

@GetMapping("/user/info")
public ResponseEntity<?> userInfo(@RequestHeader(name = "Authorization") String header) {
	log.info("=====header=====");
	log.info("Authorization : " + header);

	// Authorization : Bearer ${jwt}
	String jwt = header.replace(SecurityConstants.TOKEN_PREFIX, "").trim();
	byte[] signingKey = jwtProp.getSecretKey().getBytes();

	Jws<Claims> parsedToken = Jwts.parser().verifyWith(Keys.hmacShaKeyFor(signingKey))
			.build()
			.parseSignedClaims(jwt);

	String username = parsedToken.getPayload().get("uid").toString();
	log.info("username :" + username);

	Claims claims = parsedToken.getPayload();
	Object roles = claims.get("rol");
	log.info("roles :" + roles);

	return new ResponseEntity<String>(parsedToken.toString(), HttpStatus.OK);
}

```

이는 가장 기본적인 JWT 해석 예시입니다.
처음 토큰이 생성되면 클라이언트는 서버에 요청을 보낼 때마다 해당 토큰을 헤더에 넣어서 보냅니다.
서버는 헤더에 존재하는 JWT 를 통해 인증되었음을 확인하고 요청을 처리할 수 있습니다.

따라서 서버에서는 헤더에 존재하는 JWT 값을 파싱하는 과정이 필요합니다.

- 우선 해당 JWT 의 서명을 검증하기 위한 절차가 필요합니다.
- 생성시 사용했었던 secretKey 를 사용해 JWT 를 검증합니다. 
- 이후 헤더값에서 토큰 부분만 가져와 parsedToken 으로 파싱합니다.
- parsedToken.getPayLoad() 를 통해 Claims 를 가져올 수 있습니다. 


## Claim 

이때 Claim 이란

```
{
  "sub": "1234567890",
  "name": "Tester",
  "iat": 1516239022
}
```

위와 같이 Payload 가 존재할 때 데이터 각각의 Key 를 Claim 이라 부릅니다.

따라서 Claims 에서 get("claim_name") 을 통해 각각의 데이터에 접근할 수 있습니다. 

Claim은 3가지로 구분됩니다.

- registered claim
- public claim
- private claim

### Reserved Claim

JWT 표준에 정의된 이름의 데이터를 뜻합니다.
iss(issuer), exp(expiration time), sub(subject), aud(audience), iat(issued at) 
등으로 3글자로 미리 정해진 이름이 존재합니다.

### Public Claim

사용자가 자유롭게 정의할 수 있는 데이터입니다.
위에서는 name 등을 자유롭게 정의한 경우를 뜻합니다. 일반적인 정보나 기본 프로필 정보 등을 포함합니다. JWT 의 페이로드를 검증하는 모든 이들은 이러한 공개 클레임을 열어볼 수 있습니다.

### Private Claim

특정 애플리케이션 또는 시스템에서만 의미가 있는 정보입니다. 일반적으로 서버와 해당 어플리케이션 간에만 공유됩니다. 예를 들어 role 등의 정보는 어플리케이션에 따라 의미가 달라질 수 있는 데이터입니다.

