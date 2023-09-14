---
title: "[RestTemplate] Double Encoding"

categories:
  - spring
tags:
  - [spring, resttemplate]

toc: true
toc_sticky: true

date: 2023-09-14
last_modified_at: 2023-09-14
---

RestTemplate 을 사용해서 요청을 할 때 계속해서 Error 500 Internal Server Error 가 발생했다.
이를 해결한 방법에 대해서 작성해보도록 하겠다.

# RestTemplate

```

		UriComponents uriComponents = UriComponentsBuilder.fromUriString(baseUrl)
				.queryParam("IF", input.getIF())
				.queryParam("id", input.getId())
        .encode()
				.build();

		ResponseEntity<ExampleDto> response = restTemplate.exchange(uriComponents.toUriString(), HttpMethod.GET, httpHeaders, ExampleDto.class);


```

위와 같은 형식이 RestTemplate 을 사용하는 기본 예시이다. 하지만 위와 같은 예시에서 주의할 점은 Double Encoding 이다!

간단하게 uriComponents 에 들어가는 파라미터들이 두 번 인코딩 된다는 뜻이다. 각 인코딩이 포함되는 단계는 아래와 같다.

> UriComponentsBuilder.encode()

> uriComponents.toUriString()

위의 두 단계를 거치면서 인코딩이 발생하게 된다.

이를 방지하기 위해서는

1. UriComponentsBuilder.encode() 을 사용하면서 uriComponents.toUri() 를 사용한다.
2. UriComponentsBuilder.encode() 를 생략하고 uriComponents.toUriString() 를 사용한다.

## DefaultUriBuilderFactory

하지만 위를 적용하더라도 Internal Error 가 발생하였다.
원인은 input 에서 getId 를 했을 때 입력값이 "%7Babc-123-def%7D" 로 주어졌기 때문이다.
"%7Babc-123-def%7D" 는 "{abc-123-def}" 가 인코딩 된 형태이다.

RestTemplate 에서는 특정 문자를 자동으로 인코딩해주기 때문에 "%" 가 자동으로 인코딩된다.
"%727Babc-123-def%727D" 형태로 전달되기 때문에 에러가 발생한다.

이를 해결하기 위한 두 가지 방법이 있다.

1. 해당 id 값을 decoding 후 적용
2. DefaultUriBuilderFactory 사용

1번은 간단하니 생략하도록 하고 2번은 아래의 코드를 적용시켰다.

```

DefaultUriBuilderFactory defaultUriBuilderFactory = new DefaultUriBuilderFactory();
defaultUriBuilderFactory.setEncodingMode(DefaultUriBuilderFactory.EncodingMode.NONE);

restTemplate.setUriTemplateHandler(defaultUriBuilderFactory);

```

RestTemplate 에서 EncodingMode 를 None 으로 설정한다.
이를 통해 "%" 를 자동으로 인코딩하지 않아 정상적으로 동작하는 것을 확인할 수 있었다.

```
	public ExampleDto getExample(ExampleInput input) {

		DefaultUriBuilderFactory defaultUriBuilderFactory = new DefaultUriBuilderFactory();
		defaultUriBuilderFactory.setEncodingMode(DefaultUriBuilderFactory.EncodingMode.NONE);

		restTemplate.setUriTemplateHandler(defaultUriBuilderFactory);

		UriComponents uriComponents = UriComponentsBuilder.fromUriString(baseUrl)
				.queryParam("IF", input.getIF())
				.queryParam("id", input.getId())
				.build();

		ResponseEntity<ExampleDto> response = restTemplate.exchange(uriComponents.toString(), HttpMethod.GET, httpHeaders, ExampleDto.class);

		return response.getBody();
	}
```

# etc

- 입력 값을 "{abc-123-def}" 로 주도록 하자!!
- RestTemplate 은 Deprecated 되었다. 대신 WebClient 등을 사용하도록 하자!
