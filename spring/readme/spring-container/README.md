# Spring Container

## Spring Container란

스프링은 IoC/DI의 개념을 실제 코드에서 적용하기 위해 Container를 지원한다.&#x20;

스프링의 컨테이너는 미리 오브젝트를 생성해서 가지고 있다가 필요한 곳에 적절히 제공하는 기술이다.&#x20;

핵심은 컨테이너가 주도적으로 생성, 관계 설정, 제거 등 **오브젝트에 대해 제어권을** 코드 대신에 가지고 있는 것이다. 이는 저번 글에서 살펴봤던 IoC와 같기 때문에 IoC Container라고도 부르기도 한다.



## 두 인터페이스

컨테이너는 BeanFactory Interface와 ApplicationContext Interface를 토대로 구성된다.&#x20;

### BeanFactory

스프링 컨테이너의 최상위 인터페이스로 스프링 빈을 관리하는 핵심 역할을 한다.&#x20;

{% hint style="info" %}
### Spring Bean이란?

스프링 컨테이너(BeanFactory)에 의해 관리되는 객체를 뜻한다.&#x20;

* 빈 이름은 기본적으로 메서드 이름을 사용할 수 있으며, 직접 부여할 수도 있다.&#x20;
* 타입으로 조회 시 같은 타입이 둘 이상이면 오류가 발생한다. 이 때는 이름을 명시해 가져오자.
{% endhint %}

### Application Context

BeanFactory를 비롯해 여러가지 추가적인 부가 기능의 인터페이스를 상속한 인터페이스다.&#x20;

따라서 보통 이 인터페이스를 이용하지 BeanFactory를 단독으로 사용하지 않는다.

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-03 오전 11.19.24.png" alt=""><figcaption></figcaption></figure>

<div align="center">

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-03 오후 6.50.39.png" alt=""><figcaption></figcaption></figure>

</div>

* #### Application Context의 부가기능
  * 메시지소스를 활용한 국제화 기능
  *   환경변수

      로컬,운영 등의 환경변수를 구분해서 처리 가능&#x20;
  *   애플리케이션 이벤트

      이벤트를 발행하고 구독하는 모델을 지원
  *   편리한 리소스 조회

      파일,클래스 패스 등 리소스를 조회 가능&#x20;



*   #### 다양한 설정 형식 지원

    ApplicationContext의 구현을 Annotation 기반의 코드, XML 등 다양한 설정 형식으로 함으로써 Container를 구성할 수 있다. 하지만 실무에서는 보통 애너테이션 기반의 코드 방식을 주로 하기 때문에 앞으로의 설명은 애너테이션 기반으로 한다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-03 오전 11.29.09.png" alt=""><figcaption></figcaption></figure>

```java
# Annotation 기반
ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

# Xml 기반
ApplicationContext ac = new GernericXmlApplicationContext("appConfig.xml");

# Bean 조회
AppService memberService = ac.getBean("appService", AppService.class);
```



## Bean Definition

위에서 다양한 환경에서의 설정을 지원할 수 있을 수 있었던 것은 BeanDefinition라는 추상화를 사용했기 때문이다.&#x20;

아래 그림처럼 스프링 컨테이너는 구체화된 저수준 모듈을 의존하는 것이 아닌 BeanDefinition 인터페이스에 의존하며, 각 모듈들이  의존하고 있기 때문에 DIP을 지킨 것을 볼 수 있다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-03 오후 2.44.14.png" alt=""><figcaption></figcaption></figure>

참고로 생성은 BeanDefinitionReader 추상화를 이용한다. 예를들어, 애노테이션 기반이라면 AnnotatedBeanDefinitionReader를 사용해 BeanDefinition을 생성할 것이다.&#x20;



## 애노테이션 기반의 빈 설정

```java
@Configuration
class AppConfig{
   
   @Bean
   public MemberService memberService(){
     return new MemberService();
   } 
   
   @Bean
   public AppService appService(){
      // 위의 함수로부터 memberService 의존성을 직접 주입한다.
      return new AppService(memberService()); 
    } 
    
}
```

### @Configuration

한 개 이상의 Bean 메서드를 가지는 클래스라는 것을 명시하는 애노테이션. 즉, 빈 설정 정보를 가지는 클래스라는 것을 명시한다.

### @Bean

스프링 컨테이너가 관리되는 객체로 설정 하겠다는 애노테이션이다.&#x20;



## Singleton Container

웹 서비스는 보통 많은 유저들이 동시에 접속하고 사용한다. 만약 한 유저가 접속할 때마다 어떤 객체를 생성해야 한다면 많은 리소스가 낭비된다. 이 때 빈을 싱글톤 패턴으로 유지한다면 문제를 해결 할 수 있을 것이다.&#x20;

{% hint style="info" %}
Singleton Pattern

객체 인스턴스를 딱 하나만 생성하고 지속적으로 사용하는 패턴이다.
{% endhint %}

하지만 싱글톤 패턴은 안티 패턴이라고 할 정도로 단점이 꽤 많고 치명적이다.



### 싱글톤 패턴의 단점

* 좋지 못한 객체지향 할 가능성이 커지고, 테스트가 용이 하지 않다.
  * 구체 클래스 성격의 싱글톤 객체를 바로 의존하기 때문에(전역변수처럼 사용되기 때문에) 결합도가 커지며 테스트를 어렵게 만든다.
  * 싱글톤의 객체는 private 생성자를 두기 때문에 mocking이나 , 자식 클래스를 만들기 힘들다.&#x20;
* 기존 순수 객체에 여러 부가적인 코드가 포함된다.
* 멀티 쓰레드 환경이라면 여러 인스턴스가 생성 될 수도 있다.



### Singleton Registry

스프링 컨테이너는 앞서 말한 싱글톤의 장점만을 취하고 단점을 없애 빈을 유지할 수 있도록 하는 기능이 있는데 이를 싱글톤 레지스트리라 한다.

```java
public class Pattern{
	private Pattern(){} 

	private static class PatternHolder{
		private static final Pattern INSTANCE = new Pattern();
	}

	public static Pattern getInstance(){
		return PatternHolder.INSTNACE;
	}
	
}
```

위의 코드는 싱글톤 패턴의 예시이다. private 생성자라던지 정적 class를 사용하는 등 따로 싱글톤에 필요한 코드를 객체에 반영하지 않아도 자동으로 싱글톤으로 동작하도록 도와준다.

###

### CGLIB

어떻게 우리가 따로 싱글톤에 대한 처리를 하지 않아도 싱글톤처럼 동작할 수 있는 것일까?&#x20;

바로 CGLIB라는 라이브러리를 이용해 우리가 기존 원본 객체의 바이트 코드를 수정한 것을 상속하는 객체를 만들고, 싱글톤처럼 동작하도록 부가 기능을 추가하며, 이렇게 생성된 객체를 빈으로 등록하여 사용하는 방식이기 때문이다.&#x20;

{% hint style="info" %}
싱글톤 패턴에서의 주의점&#x20;

여러 클라이언트에서 접근하는 방식이기 때문에 생글톤 객체에서는 절대 상태를 유지하면 안된다.

애초에 수정할 수 있는 로직을 만들지 말자.
{% endhint %}

{% hint style="info" %}
@Configuration 없이 @Bean만 선언하게 된다면?

싱글톤을 보장하지 않는다.&#x20;
{% endhint %}

##

## 참고&#x20;

스프링 핵심 원리 - 기본편 (김영한)

토비의 스프링



