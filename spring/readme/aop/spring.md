# Spring의 프록시

## Proxy Factory&#x20;

스프링은 앞서 소개한 두 개의 기술을 아래에서 소개할 기능을 통합 하면서도 편하게 사용하도록 ProxyFactory를 제공한다.&#x20;



1. **최적의 프록시 기술 선택**

Spring은 인터페이스의 유무에 따라 위에서 소개한 JDK 동적프록시와 CGLIB 기술을 알아서 선택하고 프록시를 동적으로 생성해준다.&#x20;

보통 인터페이스가 없다면 CGLIB를 사용하고 있다면 JDK 동적 프록시를 사용할 것이다. 따로 옵션을 통해 인터페이스가 있어도 cglib를 사용할 수 있다. (`proxyTargetClass=true)`

<img src="../../../.gitbook/assets/file.excalidraw (4).svg" alt="" class="gitbook-drawing">

2. **Advice**

두 프록시 기술을 사용하기 위해 jdk 동적 프록시의 invocation handler와 cglib의 method interceptor를 둘 다 구현해야한다면 번거러울 것이다.&#x20;

스프링은 두 기술에 의존하지 않도록 서비스 추상화한 기술을 제공한다. jdk proxy와 cglib 두 구현체 모두 공통적인 advice를 호출하도록 하고 있다. 이렇게 추상화된 부가 기능 로직을 advice라고 한다.&#x20;

<img src="../../../.gitbook/assets/file.excalidraw (16).svg" alt="" class="gitbook-drawing">

3. **PointCut**

특정 조건에 맞는 대상 클래스에서만 부가 기능을 적용하는 기능을 통합하기 위해 Pointcut이라는 개념을 도입했다.&#x20;

포인트컷은 크게 대상의 클래스가 맞는지 메서드가 맞는지 확인하는 작업으로 이루어져있다.

```java
public interface Pointcut {
      ClassFilter getClassFilter();
      MethodMatcher getMethodMatcher();
}
```

실무에선 스프링이 제공하는 많은 포인트컷 중 AspectJExpressionPointcut를 가장 많이 사용된다.&#x20;



4. **Advisor**

하나의 포인트 컷과 하나의 어드바이스를 가지는 것

<img src="../../../.gitbook/assets/file.excalidraw (33).svg" alt="" class="gitbook-drawing">

어드바이저는 하나의 타겟에 여러 어드바이저를 등록할 수 있다. 때문에 타겟에 여러 어드바이스를 적용하기 위해 매번 프록시를 등록하지 않아도 된다.&#x20;

<img src="../../../.gitbook/assets/file.excalidraw (20).svg" alt="" class="gitbook-drawing">





### 전체 구조

<img src="../../../.gitbook/assets/file.excalidraw (1).svg" alt="" class="gitbook-drawing">



