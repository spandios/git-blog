---
description: Spring Security가 이해하는 사용자의 명세
---

# UserDetails

사용자를 인식하는 것은 인증에서 필수적이다. 이 작업은 당연히 사용자에 대한 정의가 선행 되어야 하고 개발자는 이 정의에 맞게 개발해야 한다.

스프링 시큐리티는 사용자 정의를 UserDetails 인터페이스로 정의를 했다.스프링 시큐리티가 이해하는 유저는 오직 UserDetails 인터페이스에 맞는 객체인 것이다.



## UserDetails Interface

```java
package org.springframework.security.core.userdetails;

import java.io.Serializable;
import java.util.Collection;
import org.springframework.security.core.GrantedAuthority;

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

* `getAuthorities:` 유저의 권한들을 반환한다.
* getUsername & getPassword: 아이디와 패스워드를 반환한다.
* 나머지: 각각 계정 만료, 계정 잠금, 자격 증명 만료, 계정 비활성화을 뜻한다. true가 긍정의 의미(문제 없음)를 나타내기 위해 메서드명을 약간 꼬은 것을 볼 수 있다.

## GrantedAuthority

권한은 사용자가 애플리케이션에서 수행할 수 있는 작업을 뜻한다. 권한을 이해하고 구분하기 위해 스프링 시큐리티는 권한을 GrantedAuthority 인터페이스로 정의한다.&#x20;

```java

public interface GrantedAuthority extneds Serializable{
    String getAuthority();// 권한이름을 스트링 형식으로 반환하는 추상메서드 딱 하나만 있다.
}
```

실제로 GrantedAuthority를 구현해보자.&#x20;

```java
GrantedAuthority g1 = () -> "READ"; // 추상메서드가 하나만 있으므로 labmda도 가능.
GrantedAuthority g2 = new SimpleGrantedAuthority("READ"); // 권한이름을 문자열로 받으며 인스턴스 생성
```

{% hint style="info" %}
@FunctionalInterface

GrantedAuthority 인터페이스는 추상 메서드가 하나이므로 람다식으로 구현할 수 있다. 하지만 @FunctionalInterface 애노테이션이 없으므로 사용하지말자.&#x20;

@FunctionalInterface가 없다면 향후에 하나 이상의 추상 메서드가 추가 될 수 있기 때문이다.
{% endhint %}



## User 클래스 책임 분리&#x20;

만약 JPA에서 사용하는 엔티티 클래스가 있는데 여기에 스프링 시큐리티가 이해할 수 있게 UserDetails 인터페이스도 구현하도록 하는 것은 좋지  않다. JPA와 시큐리티 관련된 책임을 한 클래스에서 다 하기 때문이다. 따라서 클래스를 나누어 구현하는 것이 좋다.&#x20;

```java
public class SecurityUSer implements UserDetails{
    private final User user;
    
    public SecurityUser(User user){
        this.user = user;
    }
    

    @Overried
    public String getUsername(){
        return user.getUserName();
    }
    
    // 나머지 UserDetails 명세에 맞게 구현 
    
}
```

