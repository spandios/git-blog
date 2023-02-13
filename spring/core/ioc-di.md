# IoC / DI

### DI (Dependency Injection)

DI는 어렵게 생각할 필요 없이 의존성을 인수로 취해 주입 받는 것으로 생각하자.

그럼 왜 직접 의존성을 생성하지 않고 주입받는 것일까? 바로 좋은 객체를 설계하는데 도움이 되는 객체지향 원칙을 지키기 위해서이다.

DI가 객체지향 원칙 위반한 코드를 어떻게 해결하는지 살펴보도록하자.

#### 1. 피자가게

```kotlin
class PizzaStore : IPizzaStore{
    private val pizzas : List<KimchiPizza> = listOf(
        KimchiPizza(),
        KimchiPizza(),
        KimchiPizza()
    ) // 3개로 고정되어 있다.

    override fun sell(pizzaName:String) {
        // TODO: Implement this method
    }
}
```

* OCP 위반

피자 가게는 피자가 3개를 직접 생성하고 참조하고 있다. 만약 판매할 피자의 개수가 변경 된다면 PizzaStore자체를 수정해야하므로 변경에 열려 있으니 OCP를 위반한다.

아래는 판매할 피자 리스트를 인수로 받아 의존성을 주입 받게 끔 개선된 코드다.

이제 피자 리스트를 직접 참조하지 않게 되어 PizzaStore 변경은 닫혀 있어 OCP 위반 문제를 해결할 수 있다.

```kotlin
// 의존성 주입이 된다면 판매할 피자 리스트를 직접 참조하지 않아 OCP 위반문제를 해결 할 수 있다.
class PizzaStore(private val pizzas : List<KimchiPizza>){ 

   
}

val pizzaStore = PizzaStore(listOf(KimchiPizza(),KimchiPizza()))
```

이제 피자 리스트를 직접 참조하지 않게 되어 PizzaStore 변경은 닫혀 있어 OCP 위반 문제를 해결할 수 있다.

\


* DIP 위반

OCP 위반을 해결해도 여전히 문제가 남아있는데, 현재 김치피자만 판매할 수 밖에 없고 다른 피자를 바꾼다 해도 PizzaStore의 코드의 수정은 불가피하다.

상위 모듈인 피자가게가 하위 모듈이자 구체화 모듈 김치피자에 직접적으로 의존하고 있기 때문이다. 다형성을 취하지도 못할 뿐 아니라 수정에 열려 있어 좋지 못한 구조이다.

아래는 Pizza Interface를 도입해 추상화 된 의존성을 주입 받게 끔 개선된 코드다.

이제 피자가게가 구체화된 모듈에 의존하는게 아닌 추상화(interface)에 의존하고 있어 DIP 위반 문제를 해결 할 수 있다.

```kotlin
interface Pizza {}

class KimchiPizza : Pizza{

}

class CheesePizza : Pizza{

}

class PizzaStore(private val pizzas: List<Pizza>){

}

val pizzaStore = PizzaStore(listOf(KimchiPizza(),ChessePizza()))
```

#### 2. OrderService

먼저 DI 없이 OrderService를 작성해본다. OrderService는 할인 정책을 가지고 있다. 현재 할인 정책은 고정된 할인과 퍼센트 할인이 있다.

```kotlin
class OrderServiceImpl : OrderService {
    private val discountPolicy: DiscountPolicy = FixDiscountPolicy()

    fun order(memberId:Int, itemId:Int, price:Int): Order {
        val discountPrice: Int = discountPolicy.discount(price)
        return Order(memberId, itemId, price, discountPrice)
    }
}
```

discountPolicy변수를 보면 구현체가 아닌 DiscountPolicy Interface을 타입으로 받고 있으니 DIP를 지켰다고 할 수 있다. 또, 할인정책과 주문 서비스를 각각 분리해 역할을 나눴기 때문에, 할인정책 변경이 주문 서비스 코드를 변경하지 않으니,  OCP 또한 지켰다고 기대할 수 있다고 생각할 수 있지만, 실제로는 DIP, OCP 둘다 위반이다.

DiscountPolicy 인터페이스를 사용하고 있지만 실제로는 구체 클래스 FixDiscountPoliy를 의존하고 있으므로 DIP를 위반한다. 만약 고정 할인 대신 퍼센트 할인으로 정책이 변경된다면 주문 서비스의 코드 변경이 일어나므로 OCP 또한 위반이다.

```kotlin
// private val discountPolicy: DiscountPolicy = FixDiscountPolicy()  코드 수정이 일어남
private val discountPolicy: DiscountPolicy = RateDiscountPolicy()
```

이 문제를 해결하려면 OrderServiceImpl에서 구체 클래스를 자신이 직접 생성하지 않고 단지 인터페이스만을 의존해야 한다.

```kotlin
class OrderServiceImpl : OrderService {
    private val discountPolicy: DiscountPolicy // 직접 구현체를 생성하지 않음 

    fun order(memberId:Int, itemId:Int, price:Int): Order {
        val discountPrice: Int = discountPolicy.discount(price)
        return Order(memberId, itemId, price, discountPrice)
    }
}
```

하지만 지금 이대로 실행하면 discountPolicy에 대한 실제 구체 클래스에 대한 정보가 없으므로 당연히 null pointer exception이 발생할 것이다.

이때 DI를 통해 외부에서 구현체를 받으면 OCP, DIP를 지킬 수 있다.

DI를 사용하면 클라이언트 코드를 변경하지 않고, 런타임 시 실제 객체 인스턴스를 쉽게 변경해 연결 할 수 있기 때문이다.

```kotlin
// 생성자 주입 방식을 통해 DIP, OCP를 지킬 수 있다.
class OrderServiceImpl(private val discountPolicy: DiscountPolicy) : OrderService {
    fun order(memberId:Int, itemId:Int, price:Int): Order {
        val discountPrice: Int = discountPolicy.discount(price)
        return Order(memberId, itemId, price, discountPrice)
    }
}
```

**관심사의 분리**

외부에서 구현체를 DI 받아야하므로 누군가는 구현체를 생성하고 의존성 주입하는 책임을 가져야 한다. AppConfig 클래스에서 이 책임을 갖도록 해보자.

```kotlin
class AppConfig {		
    // 구현체를 생성한다.
    private fun discountPolicy() : DiscountPolicy = RateDiscountPolicy() 
    // 구현체 생성 뿐 아니라 의존성도 주입한다.
    fun orderService() : OrderService = OrderServiceImpl(discountPolicy()) 
}
```

이렇게 AppConfig는 객체를 생성과 의존성 주입에만 관심 가지도록 하고, OrderService는 어떤 구체적인 할인정책은 무시하고 오직 주문 자체에만 관심 가지도록 분리했으니 관심사를 분리했다고 할 수 있다.

### IoC (Inversion of Controll)

AppConfig를 사용하기 전에는 우리가 구현 객체를 스스로 생성하고 사용했다. 하지만 AppConfig가 등장 후 제어흐름은 구현 객체 스스로가 아닌 AppConfig가 주관하게 됐다. AppConfig는 DiscountPolicy와 OrderService의 구현체를 결정한다. 각 구현체는 자기 자신이 어디서 생성되는지 모르며 구체적인 의존 관계는 모른다. 객체 생성과 의존성은 관심이 없고 그저 자신에게 맡겨진 로직을 충실히 수행할 뿐이다.

AppConfig처럼 객체에 대한 생성 및 생명주기 관리 기능을 스프링은 IoC(DI) Container를 통해 제공한다. 더 자세한 건 다음글에서 다룬다.

이렇게 IoC는 메서드 호출 시점이나 생명주기 등 프로그램 제어 흐름을 직접관리하는 게 아닌 외부 요소가 관리하는 것이라고도 한다. 예를들어 프레임워크는 자신들의 설계 대로 프로그램이 동작하도록 규격과 흐름을 만들었고, 개발자에게는 단지 특정 규격을 지킨 부품만을 요구한다. 이렇게 받은 부품은 프레임워크가 가져다가 사용한다. 개발자가 자신이 작성한 코드의 제어를 신경쓰지 않고 프레임워크에게 위임하는 것을 볼 때 제어의 역전이 일어났다고 할 수 있다.

또 다른 예로는 템플릿 메서드 패턴을 들 수 있다. 템플릿 메서드 패턴은 변경이 없는 곳은 상위 클래스로 변경이 잦은 곳은 하위 클래스로 둔다. 하위 클래스에서 오버라이딩한 메서드는 하위 클래스가 자체가 관리하는 게 아닌 상위 클래스가 호출하고 흐름을 관리한다. 따라서 이 경우도 제어의 역전이 일어났다고 할 수 있다.

IoC는 궁극적으로는 관심사를 분리한 것이라고 생각하고 유연한 구조를 가짐으로써 그것이 주는 장점을 얻을 수 있다고 생각한다.

### IoC와 DI의 관계

<figure><img src="https://blog.kakaocdn.net/dn/eq6u5i/btrWLyyZK5F/v5eMkXemJ8yxSwm8eZtUpK/img.png" alt=""><figcaption><p>https://jwchung.github.io/DI%EB%8A%94-IoC%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EC%95%8A%EC%95%84%EB%8F%84-%EB%90%9C%EB%8B%A4</p></figcaption></figure>

\


IoC와 DI는 별도의 개념이며 spring만의 특정 기술이 아니다.

스프링은 IoC 개념을 활용해 객체지향 원칙을 지키고 편하게 객체를 관리할 수 있도록 DI Container를 적극적으로 사용하며, 이를 위해 DI를 사용한다. 서로 독립적인 기술이므로 의존성은 없지만 같이 사용하여 효율을 극대화한다.



### 마치며

지금까지 IoC / DI를 알아보았다. 이 기술들은 객체지향 원칙을 지키기 위한 것이기 때문에 스프링의 핵심 본질을 위해 꼭 필요하다고 생각된다. 왜 스프링의 핵심 기술들 중 가장 기본이자 핵심인지 알게 되었다. 다음은 실제로 IoC를 구현하기 위해 사용되는 Spring의 Container에 대해 더 알아보려고 한다.

###

### 참고

[DI는 IoC를 사용하지 않아도 된다](https://jwchung.github.io/DI%EB%8A%94-IoC%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EC%95%8A%EC%95%84%EB%8F%84-%EB%90%9C%EB%8B%A4)

스프링 핵심 원리 - 기본편 (김영한)
