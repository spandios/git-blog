---
description: 스프링은 스프링 시큐리티라는 보안 관련 프레임워크를 제공한다.
---

# Legacy

스프링 시큐리티는 주로 인증과 인가 기능에 강력하고도 매우 유연한 구조로 되어 있는 필터 기반의  보안 프레임워크이다.



{% hint style="info" %}
필터란 디스패쳐 서블릿에 도달하기 전에 어떤 작업을 할 수 있는 기능
{% endhint %}



![](../../../.gitbook/assets/FilterChain.png)

어떤 요청이 들어와 처리되기 전 인증과 인가가 됐는지 확인하는 작업이 스프링 시큐리티의 핵심이다.

실제 시큐리티에서는 클라이언트에서 요청이 들어오면 필요한 필터들이 체인으로 엮여 순서에 맞게 실행되고 끝으로 서블릿에 최종적으로 도달한다.



### DelegatingFilterProxy <a href="#servlet-delegatingfilterproxy" id="servlet-delegatingfilterproxy"></a>

Filter들은  Servlet Context에 속하므로 Application Context에 등록된 Security 관련 Bean을 사용하지 못한다.

이를 해결하기 위해 DelegatingFilterProxy 필터는 FilterChainProxy라는 BeanFilter를 보유하고 있고, 이 FilterChainProxy 필터가 각 요청에 알맞는 Security Filter Chain에게 요청을 위임한다.

SecurityFilterChain은 Application Context에 속하므로 Bean을 사용할 수 있는 Filter들로 구성할 수 있다.

이런 구성덕에 최 앞단인 Servlet 필터에서 처리하는 것 보다 커스텀 가능하고, 확장성 있게 보안 관련 처리를 할 수 있다.





![](<../../../.gitbook/assets/스크린샷 2022-06-04 오후 4.08.24.png>)



다음 글은 실제로 인증을 어떻게 처리하는지 살펴보겠다.



{% hint style="info" %}
참고

[https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-filterchainproxy](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-filterchainproxy)

[https://uchupura.tistory.com/24](https://uchupura.tistory.com/24)
{% endhint %}

