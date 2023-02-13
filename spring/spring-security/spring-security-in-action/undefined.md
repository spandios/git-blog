---
description: "\b기본 구성을 큰 그림으로 알아보면서 깊이 학습하기 전 친해져보자."
---

# 스프링 시큐리티 기본 구성 맛보기

## 인증 흐름&#x20;

<img src="../../../.gitbook/assets/file.excalidraw (3).svg" alt="authentication flow" class="gitbook-drawing">

### Authentication Filter

인증 필터는 인증 요청을 인증 관리자에게 위임하고 응답을 보안 컨텍스트를 구성한다.

### Authentication Manager

인증 공급자를 이용해 인증을 처리한다.

### Authentication Provider

* 실제 인증 로직을 구현한다.
* UserDetailService를 이용해 사용자 관련 데이터를 얻는다.
* PasswordEncoder를 이용해 암호 관련 기능을 사용한다.

### 보안 컨텍스트

인증 결과를 유지한다.&#x20;



### 따로 직접 구성 안하면?

만약 스프링 시큐리티를 사용하면서 아무 설정을 안하면 기본적으로 HttpBasic방식으로 앱을 보호한다.&#x20;

UserDetailsService와 PasswordEncoder를 재정의 하지 않을 시 기본값으로 구성해주기 때문이다.&#x20;



## 기본 구성요소 맛보기

### UserDetailsService & PasswordEncoder

UserDetailsService는 유저 정보와 관련한 책임을 가지고 있고 PasswordEncoder는 암호와 관련한 책임을 가지고 있다.&#x20;

미리 제공되는 클래스를 이용해 구현해 빠르게 어떤 느낌인지만 알아보자.

다음 코드는 계정을 등록한 뒤 메모리에 저장하는 InMemoryUserDetailsManager를 UserDetailsService 빈을 등록했고 따로 암호처리 하지 않는 NoOpPasswordEncoder를 PasswordEncoder 빈으로 등록했다.

UserDetailsService를 재정의하면 PasswordEncoder또한 재정의 해줘야한다.

```java
@Configuration
public class ProjectConfig {

    @Bean
    public UserDetailsService userDetailsService() {
        var userDetailsService = new InMemoryUserDetailsManager();

        var user = User.withUsername("heo")
                .password("12345")
                .authorities("read")
                .build();

        userDetailsService.createUser(user);

        return userDetailsService;
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }

 
}

```

### 엔드포인트 권한 부여 방식 재정의

이제 엔드포인트의 권한 부여 방식을 재정의를 해보자. 아래 코드는 WebSecurityConfigurerAdapter를 상속받아 configure 메서드를 재정의 했다.&#x20;

```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
    // 위의 코드 생략

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.httpBasic();
        http.authorizeRequests().anyRequest().authenticated(); // 모든 요청에 인증 필요
        http.authorizeRequests().anyRequest().permitAll(); // 모든 요청에 인증 필요 없음
    }
}

```



{% hint style="info" %}
현재 WebSecurityConfigurerAdapter은 Deprecated 되었다.

[https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)

추후 권한부여 챕터 때 이제 어떤 방식을 권장하는지 알아보겠다.

또, 앞서 빈으로 등록한 방식 이외에 WebSecurityConfigurerAdapter의 configure 메서드를 재정의 해 AuthenticationManagerBuilder를 이용하는 방법도 있지만, Deprecated된 이상 더 이상 알아보지 말자
{% endhint %}



### AuthenticationProvider 재정의&#x20;

AuthenticationProvider는 앞서 살펴본 UserDetailsService와 PasswordEncoder를 이용해 실제 인증 로직을 구현한다. 하지만 기본 인증 흐름을 사용하지 않고 직접 재정의 해 로직을 변경할 수도 있다는 것만 일단 알아두자.

아이디가 heo인지 비밀번호가 12345인지 확인하는 로직을 직접 코드로 구현한 것을 볼 수 있다.

```java
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        // 인증 로직을 구현할 위치
        String username = authentication.getName();
        String password = String.valueOf(authentication.getCredentials());

        if ("heo".equals(username) && "12345".equals(password)) {
            return new UsernamePasswordAuthenticationToken(username, password, Arrays.asList());
        } else {
            throw new AuthenticationCredentialsNotFoundException("Error!");
        }
    }

    @Override
    public boolean supports(Class<?> authenticationType) {
        // Authentication 형식의 구현을 추가할 위치 
        return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authenticationType);
    }
}

```

&#x20;







