# Component Scan

## Component Scan이란?

스프링 빈 등록은 귀찮기도하고 점점 의존관계가 복잡해지면 실수가 많아질게 뻔하다.

스프링은 이런 문제점을 해결하기 위해 자동으로 빈을 등록해주는 기능을 지원한다. &#x20;

이 기능은 @ComponentScan이라는 애너테이션을 이용하면 되는데, @Component 애너테이션이 붙은 모든 클래스를 스캔해 빈으로 자동 등록한다.&#x20;

```java
// 1. ComponentScan
@ComponentScan() 
public class AutoAppConfig {
}

// 2.Component
@Component() 
public class MemoryMemberRepository implements MemberRepository{

}

```

위의 코드로 예를들면 MemoryMemberRepository class에 @Component을 적용했으며 이 클래스는 @ComponentScan에 의해 자동으로 빈으로 등록 된다.



## Component 스캔 기본 대상

@Service, @Repository, @Configuration, @Controller 애너테이션들은 기본적으로 @Component를 포함하고 있기 때문에 컴퍼넌트 스캔에서 스캔 대상이 된다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-04 오후 1.40.27.png" alt=""><figcaption></figcaption></figure>

## Spring boot에서는?

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-03 오후 4.48.28.png" alt=""><figcaption></figcaption></figure>

기본적인 설정을 해주는 편리한 @SpringBootApplication 애너테이션에도 이미 @ComponentScan이 명시되어 있는 것을 알 수 있다.&#x20;



#### Filter

위의 예시를 보면 컴퍼넌트스캔 애너테이션에 excludedFilter를 사용하는 것을 볼 수 있다.

필터 설정에는 includeFilter와 excludeFilter가 있는데 각각 포함과 제외를 필터해준다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-04 오후 1.54.30.png" alt=""><figcaption></figcaption></figure>

#### BasePacakges

탐색할 패키지의 시작위치를 지정해준다. 만약 지정하지 않으면 @ComponentScan이 붙은 설정 정보 클래스의 패키지가 시작 위치가 된다.&#x20;

스프링 부트는 기본적으로 따로 설정하지 않고 설정 정보 클래스를 프로젝트 최상단에 위치하게 한다.&#x20;

