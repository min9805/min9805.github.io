---
title: "[SpringSecurity + JWT] 02. CustomUserDetails"
excerpt: "미니프로젝트를 위한 SpringSecurity, JWT"

categories:
  - spring
tags:
  - [spring, resttemplate]

toc: true
toc_sticky: true

date: 2024-02-27
last_modified_at: 2024-02-27
---

 그럼 본격적으로 SpringSecurity 를 사용한 JWT 를 구현해보겠습니다. 해당 게시글에서는 구조 파악에 우선순위를 두어 예제를 실행했습니다.

로그인 등의 인증을 위해 DB 에서 직접 ID, Password 를 가져와 처리할 수 있지만, 여러 정보들 중 제한적 내용만 이용한다는 단점이 있습니다. 

또한 Security 에서는 DaoAuthenticationProvider 에서 요청받은 유저의 ID, Password 와 저장된 ID, Password 를 검증하는 역할을 하고 있으며, 이를 위해 UserDetailService 와 협력합니다. 

즉, ID, Password 를 넘겨줄 수 있는 UserDetailService 가 필요합니다! 

# UserDetails, UserDetailsService


```

#UserDetails 인터페이스

public interface UserDetails extends Serializable {

	Collection<? extends GrantedAuthority> getAuthorities();

	String getPassword();

	String getUsername();

	boolean isAccountNonExpired();

	boolean isAccountNonLocked();

	boolean isCredentialsNonExpired();

	boolean isEnabled();

}

```

```

# UserDetailsService

public interface UserDetailsService {

	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;

}


```

UserDetaolsService 는 username 을 받아 UserDetails 를 반환하고 있습니다. 

반환된 UserDetails 는 getAuthorities(), getPassword() 등의 메서드를 가지고 있습니다.

결과적으로는 DaoAuthentication 에서 돌려받은 UserDetails 객체를 가지고 UsernamePasswordAuthentication 객체를 만들어 ProviderManager 에게 돌려줍니다. 
이후 authenticate() 메서드를 통해 검증할 수 있습니다.

# UserDetailsService 구현

```

@RequiredArgsConstructor
@Service
public class CustomUserDetailService implements UserDetailsService {
    private final UserRepository userRepository;

    //Spring security 에서 AuthenticationManager 가 Authentication 메소드 호출 시 인증하기 위해 해당 로직 사용
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username);

        if (user == null) {
            throw new UsernameNotFoundException("사용자를 찾을 수 없습니다 : " + username);
        }

        // Users (CustomDto) -> CustomUser (SecurityDto)
        return new CustomUser(user);
    }
}


```

UserDetailsService 인터페이스를 구현한 클래스입니다. 

loadUserByUsername() 을 오버라이딩 하였습니다.

userRepository 에서 findByUsername 을 진행하고 해당 user 를 CustomUser 로 변환하여 반환하는 역할을 하고 있습니다. 


# UserDetails 구현

```

@RequiredArgsConstructor
@Data
public class CustomUser implements UserDetails {
    private final User user;

    // 권한 getter 메소드
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<UserAuth> authList = user.getUserAuth();

        // SimpleGrantedAuthority :: "ROLE_USER" 값 필요
        Collection<SimpleGrantedAuthority> roleList = authList.stream()
                .map((auth) -> new SimpleGrantedAuthority(auth.getAuth()))
                .toList();

        return roleList;
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return user.getEnabled() != 0;
    }
}


```

UserDetails 인터페이스를 구현한 클래스입니다. 

getAuthorities() 는 Stirng 타입의 "ROLE_USER" 를 Collection<SimpleGrantedAuthority\> 로 변환하는 과정을 가지고 있습니다. 

이외에는 Expired, Locked, Enabled 등의 여부를 판단하는 함수들입니다. 

# 그 외

## Security Config

```
        http.userDetailsService(customUserDetailService);

```

구현한 UserDetailsService 를 사용하기 위해서 Config 에 다음과 같은 설정을 해줘야합니다. 

## PasswordEncoder

추가로 Spring Security 는 PasswordEncoder 사용을 강제하고 있습니다. 실제 DaoAuthenticationProvider 내부 코드에서는 비밀번호를 PasswordEncoder 의 matches 메서드를 사용해서 비교하고 있습니다. 

즉, DB 를 통해 가져온 기존의 Password 가 인코딩되어있어야한다는 뜻입니다. 


## org.springframework.security.core.userdetails.User

해당 라이브러리는 spring 에서 제공하는 UserDetails 구현체입니다. 

따라서 따로 구현하지 않고 UserDetailsSerivce 내부에서 

```
User.builder()
                .username(member.getMemberId())
                .password(passwordEncoder.encode(member.getPassword()))
                .roles("USER")
                .build();

```

바로 사용할 수 있습니다. 