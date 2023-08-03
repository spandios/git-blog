---
description: 사용자를 관리하는 방법
---

# UserDetailsService

사용자 명세를 알았으니 이제 실제로 사용자를 관리하는 기술을 알아보도록 하자.



## UserDetailsService

```java
// 유저 아이디를 통해 유저 정보를 가져온다.
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

<img src="../../../../.gitbook/assets/file.excalidraw (17).svg" alt="UserDetails" class="gitbook-drawing">

AuthenticationProvider는 자신의 책임이 유저의 자세한 정보를 찾는게 아니기 때문에 UserDetailsService에게 유저 조회를 요청한다. &#x20;

AuthenticationProvider는 UserDetails 인터페이스를 지킨 사용자 정보만 받으면 되기 때문에 UserDetailsService가 어떤 리소스로부터 어떻게 사용자를 찾는지는 관심이 없다.&#x20;

따라서 우리는 DB든 메모리든 어떤 리소스든지 우리에게 요구사항에 맞는 사용자 조회 기능을 구현할 수 있으며, 프레임워크의 한 구성요소로서 아무 문제없이 동작할 것이다.

아래는 직접 InMemory방식대로 구현한 UserDetailsService의 예시이다. 실무에 가까운 코드는 후에 정리하도록 하겠다.

```java
public class InMemoryUserDetailsService implements UserDetailsService {

    private final List<UserDetails> users;

    public InMemoryUserDetailsService(List<UserDetails> users) {
        this.users = users;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return users.stream()
                .filter(u -> u.getUsername().equals(username))
                .findFirst()
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));
    }
}


```

{% hint style="info" %}
JdbcUserDetailsManager

JDBC를 통해 데이터베이스에 연결하여 저장된 사용자를 관리한다.&#x20;

어떤 테이블명에 어떤 컬럼명을 대상으로 사용자를 찾는 지에 대해 기본값이 있고 원한다면 쿼리를 변경해 커스텀 가능하다.&#x20;
{% endhint %}

