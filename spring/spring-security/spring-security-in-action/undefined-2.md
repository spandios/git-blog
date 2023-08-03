# 암호 처리

스프링 시큐리티에서는 암호를 암호화하고 검증하는 처리를 PasswordEncoder인터페이스를 통해 정의한다.

### 준비된 구현체들

* BcryptPasswordEncoder - bcrypt 해싱함수 사용
* ScryptPasswordEncoder - scrypt 해싱 함수
* Pbkdf2PasswordEncoder - pbkdf2 이용
* StandardPasswordEncoder - SHA-256을 이용. 구식이기 때문에 사용 X

BcryptPasswordEncoder를 보통 많이 사용한다.

```jsx
val PasswordEncoder = BcryptPasswordEncoder()
```

## DelegatingPasswordEncoder

기존 애플리케이션이 사용하던 암호 인코딩 알고리즘이 달라진다면 어떻게 해야할까? DelegatingPasswordEncoder는 암호화된 문자열 맨 앞에 `{알고리즘 명칭}암호`와 같이 알고리즘 명사를 파악하고 그것을 기준으로 PasswordEncoder 구현체에 위임을 하는 방식으로 해결 할 수 있다.

Map자료구조를 통해 키에 알고리즘 명칭, 값에 PasswordEncoder 구현체를 넣어주면 된다.

```jsx
@Bean
public PasswordEncoder passwordEncoder(){
	Map<String, PasswordEncoder> encoders = new HashMap<>();
  encoders.put("bcrypt", new BcryptPasswordEncoder());
	encoders.put("sha256", new StandardPasswordEncoder());
}
```
