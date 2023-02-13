# 의존관계 자동주입

## Autowired?

저번 글에서 `@Configuration` 방식은 직접 코드로 빈을 생성하는 함수를 작성했고, 필요하다면 그 안에서 의존성 주입까지 직접 넣어줬다.

```java
// @Configuration을 통한 빈 설정
@Configuration
public class AppConfig{

   @Bean
   public AppService appService(){
      return new AppService(memberService()); // 의존성 주입을 해준다.
    } 
    
   @Bean
   public MemberService memberService(){
     return new MemberService();
   } 
}
```

하지만 `@ComponentScan`을 사용한다면 `@Configuration` 방식에서처럼 빈을 생성하는 코드를 작성하지 않기 때문에 의존성 주입할 수 있는 방법이 없다.

이런 문제를 해결하기 위해 `@Autowired` 애너테이션을 지원해주는데, 단어 뜻대로 자동으로 의존 관계를 주입해준다.&#x20;



## 생성자를 통한 Autowired&#x20;

우리가 평소에도 자주 사용하는 생성자에 `@Autowired`만 추가하면 생성자 인수에 명시된 타입을 기반으로 스프링이 자동으로 빈을 찾고 주입 해준다.&#x20;

아래 예시를 보면 빈으로 등록된 `MemberService`를 받는 생성자와 `@Autowired`가 명시 되어 있는 것을 확인할 수 있다.&#x20;

```java
@Component
public class AppService{
    private final MemberService memberService;
    
    // MemberSerivce 빈을 자동으로 찾아 주입해준다.
    @Autowired
    public AppService(MemberService memberService) {
        this.memberService = memberService;
    }
}
```

## 생성자 방식에서의 자동주입을 권장하는 이유&#x20;

생성자 이외에도 의존관계를 자동 주입하는 방법은 수정자, 필드 주입이 있다. 하지만 스프링을 포함한 대부분의 DI 프레임워크는 생성자 방식을 적극 권장하는데 가장 큰 이유는 생성자가 가지는 장점 때문이라고 생각한다.

* 불변&#x20;

의존관계는 한번 맺어진 이후 변경 될 일은 거의 없다. 때문에 객체가 생성될 때 딱 한번만 호출 되는 생성자와 잘 어울린다. 이 후에 의존 관계를 변경할 수 있는 방법 자체가 없기 때문이다. final 키워드를 사용할 수 있는 것도 같은 이유이다.

* 안전성

생성자의 장점 중에 하나는 객체를 사용할 때 반드시 필요한 정보를 강제로 받을 수 있도록 생성자에서 명시할 수 있다는 것이다.&#x20;

만약 수정자 주입 방식을 이용한 객체에서 실수로 의존성 주입을 하지 않았다면? 의존성을 주입 받지 못 했는데 사용하려고 하므로 오류가 날 것이다.&#x20;

* POJO

군더더기 없이 순수 자바 코드 방식이기 때문에 의존성 없이도 깔끔한 유닛 테스트가 가능하다.&#x20;



## `@Autowired` 생략&#x20;

만약 생성자가 하나 뿐이라면 `@Autowired`를 생략 가능하기 때문에 더 깔끔한 코드를 작성할 수 있다.&#x20;

```java
@Component
public class AppService{
    private final MemberService memberService;
    
    // MemberSerivce 빈을 자동으로 찾아 주입해준다.
    // 생성자가 하나이기 때문에 @Autowired를 생략해도 좋다.
    public AppService(MemberService memberService) {
        this.memberService = memberService;
    }
}
```



## 자동 (@ComponentScan & Autowired ) vs 수동 (@Configuration)&#x20;

* 자동
  * 정형화된 패턴에 대해 가독성을 높여주고 반복적인 작업을 편하게 도와주는 자동 방식을 기본적으로 사용하자
* 수동
  * 비지니스 로직를 도와주는 역할인 기술적인 문제나 aop 같은 곳에서 주로 사용된다.&#x20;
  * 앱에서 공통적으로 사용되는 만큼 설정 정보를 수등 등록해서 명확히 보여주는 것이 좋다.

