# DIP

전 글에서 계층구조를 살펴보다가 상위계층들이 하위계층(인프라스트럭처)을 의존하고 있으므로 발생하는 문제가 있다고 했다. 어떤 문제들이 발생하고 어떻게 해결해야할지 알아보자.&#x20;



## 문제

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-07 오후 5.32.34.png" alt=""><figcaption></figcaption></figure>

하위 계층을 의존한다는 것은 위의 그림처럼 가격 할인 계산이라는 고수준인 도메인 계층에서의 도메인 서비스가 저수준인 인프라스트럭처 계층의 구현 기술에 의존한다는 것이다. 코드로 좀 더 이해해보자

```java
class CalculateDiscountService{

private DroolsRuleEngine ruleEngine;

CalculateDiscountService(){
	this.ruleEngine = new DroolsRuleEngine(); // 1
}

public Money calculateDiscount(List<OrderLine> orderLines, String customerId){
	~~
	List<>> facts = Arrays.asList(~~) // 2
	ruleEngine.evalute("discountCalcucation", facts); // 3
 	 ~~ 
}

}
```

1. CalculateDiscountService가 직접 저수준 모듈인 DroolsRuleEngine을 생성하고 있다
2. 특정 저수준 모듈인 DroolsRuleEngine에만 특화된 코드를 고수준에서 관심을 가지고 있다
3. 저수준 Drools에 특화된 데이터인  “discountCalculation” 문자열에도 관심을 가지고 있다



**이로 인해 발생하는 문제는 다음과 같다.**



1. **기존 코드를 수정하는 작업이 많아진다**

구현체를 교체하거나 변경이 발생하면 그 구현체를 의존하는 모든 곳에서 수정해야할 작업이 많이 발생한다.

만약 DroolsRuleEngine의 내부 구현이 변경되어 "discountCalcucation"대신 다른 문자열이 필요하다면, 관련된 모든 코드에서 변경된 문자열으로 변경해야하는 작업이 생긴다.&#x20;

그 뿐만 아니라 다른 구현체로 변경하는 작업도 생성자에서 직접 변경해줘야 한다.&#x20;

만약 DroolsRuleEngine이 인기가 있어 아주 널리 사용 되면 될 수록 유지보수가 어려운 프로젝트가 될 것이 뻔하다.&#x20;

2. **테스트가 어렵다**

현재 테스트의 목적과 관심사가 아닌 특정 구현체에 대해 모든 기능이 동작 되며 미리 실제로 코드에 존재해야하는 등 많은 관심과 노력이 필요하다.&#x20;

그 뿐만 아니라 다른 구현체로 테스트하려면 테스트 코드가 아니라 실제 코드를 수정해야하는 작업이 또 생긴다.&#x20;



## 해결

해결법은 객체지향 원칙 중 하나인 DIP에 있다.

의존관계를 반대로 뒤집어 저수준이 고수준 모듈에 의존하도록 하는 것이다. 이를 위해서는 추상화가 필요하다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-07 오후 5.53.02.png" alt=""><figcaption></figcaption></figure>

DIP를 적용하면 위의 예시처럼 의존관계가 역전되는 것을 볼 수 있다.&#x20;

1. **RuleDiscounter**

* 룰을 이용해 할인계산을 구하는 기능을 추상화한 인터페이스이다.&#x20;
* 구체적인 구현이 아니라 추상화 했기때문에 **고수준** 모듈이라 할 수 있다.

2. **DroolsRuleDiscounter**

* **저수준** 모듈인 DroolsRuleDiscounter는 실제로 기능을 구현하는 구현체이다.
* RuleDiscounter 인터페이스를 상속받아 구현하고 있다.
* 따라서 저수준 모듈이 고수준 모듈을 의존하도록 의존관계가 역전된다.&#x20;

2. **CalculateDiscountService**

* 기존 고수준의 Service는 이제 저수준 모듈인 DroolsRuleDiscounter를 바로 의존하지 않는다.
* 같은 고수준인 RuleDiscounter를 의존하고 있다.



## 결과&#x20;

1. #### 이제 구현 기술 수정이나 변경으로 인한 코드 수정이 발생하지 않는다

더 이상 저수준에 의존하지 않기 때문에 만약 저수준에서 내부 구현이 변경 되었다하더라도 인터페이스 자체가 변경되지 않는 이상 CalculateDiscountService에서 코드 수정이 발생하지 않는다.&#x20;

```java
class CalculateDiscountService{
    private RuleDiscounter ruleDiscounter; // 같은 수준인 추상화 인터페이스를 사용
    
    // DI를 통해 특정 저수준 모듈을 의존하지 않게 변경한다.
    CalculateDiscountService(RuleDiscounter ruleDiscounter){
        this.ruleDiscounter = ruleDiscounter;
    }
}
```

```java
// 사용할 저수준 객체 
RuleDiscounter ruleDiscounter = new DroolsRuleDiscounter();

// 고수준 객체 CalculateDiscountService는 생성자 방식으로 저수준 객체를 주입받는다. 
// 이 객체는 같은 고수준인 RuleDiscounter 인터페이스에 의존하기 때문에 내부 구현은 변경 되지 않는다. 
CalculateDiscountService service = new CalculatediscountService(ruleDiscounter);

```

만약 저수준 객체가 변경이 필요하다면 사용할 객체만 교체만 해주면 된다.

```java
// 사용할 저수준 객체 
RuleDiscounter ruleDiscounter = new AnotherRuleDiscounter();

// 고수준 객체 CalculateDiscountService는 생성자 방식으로 저수준 객체를 주입받는다. 
// 이 객체는 같은 고수준인 RuleDiscounter 인터페이스에 의존하기 때문에 내부 구현은 변경 되지 않는다. 
CalculateDiscountService service = new CalculatediscountService(ruleDiscounter);

```

2. **테스트가 쉬워진다.**

직접 구현체를 생성해서 사용하지 않으며 인터페이스 타입으로 주입받는 형식으로 변경 된다.&#x20;

이로써 테스트의 관심 밖인 구현 객체에 대해 완벽히 준비되거나 동작 하는 것이 필수가 아니게 된다.

대역 객체를 사용해서 관심 밖인 어떤 구현체에 상세히 알 필요 없이, 테스트에 필요한 기능에 대해서 알맞게 기능을 정의할 수 있기 때문이다.&#x20;



## DIP를 적용한 아키텍처 예시&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-08 오후 12.23.35.png" alt=""><figcaption></figcaption></figure>

* Notifier (응용 <=> 인프라)

실제 구현체이자 저수준인 EmailNotifier가 고수준 응용 계층의 Notifer 인터페이스를 바라보고 있다.&#x20;

따라서 이메일 알림 서비스 대신 문자 서비스인 SMSNotifier 또는 이메일과 문자를 같이 사용하는CompositeNotifier로 대체해도 OrderService는 코드를 변경할 필요가 없다.

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-08 오후 12.27.18.png" alt=""><figcaption></figcaption></figure>



* OrderRepository ( 도메인 <=> 인프라)

실제 구현체이자 저수준인 MyBatisOrderRepository가 고수준 도메인 계층의 OrderRepository 인터페이스를 바라보고 있다.&#x20;

따라서 기존 Mybatis 대신 Jpa를 사용한 JpaOrderRepository로 대체해도 OrderService는 코드를 변경할 필요가 없다.

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-08 오후 2.25.50.png" alt=""><figcaption></figcaption></figure>

## 주의사항&#x20;

고수준 모듈이 저수준 모듈을 의존하지 않게 하는 것이 핵심이다.&#x20;

아래는 잘못된 구조인데, 단지 인터페이스로 분리할 뿐 여전히 고수준인 도메인 서비스가 실제 구현체를 담당하는 인프라 계층의 인터페이스를 의존하고 있기 때문이다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-08 오후 12.08.57.png" alt=""><figcaption></figcaption></figure>

때문에 아래 그림처럼 RuleDiscounter라는 인터페이스를 도메인 계층에 둬서 고수준 모듈을 바라보게 끔 해야한다.

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-08 오후 12.12.25.png" alt=""><figcaption></figcaption></figure>



&#x20;
