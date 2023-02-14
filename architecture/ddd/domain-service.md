# 도메인 서비스

도메인이 복잡해질수록 여러 애그리거트가 서로 협동해서 도메인 로직을 구현하는 경우가 대부분이다.

그런데 어떤 로직은 한 애그리거트가 맡기엔 애매한 경우가 있을 것이다. 결제금액 계산을 예로 생각해보자.

결제 금액을 계산하는 로직은 아래 목록처럼 여러 애그리거트의 조합으로 최종 결제 금액을 계산해야 한다.

* 상품 애그리거트: 상품의 가격&#x20;
* 주문 애그리거트: 구매 개수
* 할인 쿠폰 애그리거트: 할인 금액이나 비율 등 로직&#x20;
* 회원 애그리거트: 회원 등급에 따라 추가 할인

그렇다면 이 계산을 하는 주체는 누구일까? 가장 먼저 쉽게 떠오르는 것은 주문 애그리거트가 필요한 데이터를 모두 알도록하며 계산 로직까지 처리하면서 권한과 책임을 무리하게 넓히는 것이다.

하지만 변경 되기 쉬운 할인 계산 로직이 아니나 다를까 후에 변경 되면 별 상관도 없는 주문 도메인을 수정해야하는 문제가 발생한다. 결합도가 높아지고 응집도가 낮아진 것이다.&#x20;

이 문제를 해결할 수 있는 방법 중 하나가 바로 별도의 서비스로 하나 분리하고 그곳에서 구현하는 것이다.



## 도메인 서비스

도메인 서비스는 도메인 로직을 구현한다. 한 애그리거트에 넣기 애매한 개념은 도메인 서비스를 이용해 다루면 된다.&#x20;

주로 다음 상황에서 도메인 서비스를 사용한다.

* 계산 로직: 여러 애그리거트가 필요한 계산 로직이나, 한 애그리거트에 넣기는 복잡한 로직이 필요할 때
* 외부 시스템 연동이 필요: 구현하기 위해 외부 시스템을 사용해야 하는 로직이 필요할 때



### 서비스 생성

{% hint style="info" %}
도메인 서비스는 상태 없이 로직만 구현한다.

따로 상태를 가질 필요가 전혀 없다. 오직 로직 구현을 책임으로 하는 서비스이기 때문이다.
{% endhint %}

```java
public class DiscountCalculationService{
    // 상태 없이 파라미터로 받은 것을 토대로 계산한다.
    public Money calculateDiscountAmounts(List<OrderLine> orderLines, List<Coupon> coupons, MemberGrade grade){
        // 계산하기 
        return 계산 값  
    }
}
```



### 사용해보기

1. 애그리거트 메서드에서 도메인 서비스를 파라미터로 받아 사용하는 사례

```java
class Order{
    ~~ 
    // Service를 전달받아 사용한다.
    public void calculateAmounts(DiscountCalculationService disCalSvc, MemberGrade grade){
        Money discountAmounts = disCalSvc.calculateDiscountAmounts(~~);
        this.paymentAmounts = totalAmounts.minus(discountAmounts);
    }
    ~~
}
```



2. 도메인 서비스 메서드에서 애그리거트를 파라미터로 받아 사용하는 사례

```java
public class TransferService{
    public void transfer(Account fromAcc, Account toAcc, Money amounts){
        fromAcc.withDraw(amounts);
        toAcc.credit(amounts);
    }    
}
```



### 응용 서비스인지 햇갈릴 때&#x20;

도메인 서비스인지 응용 서비스인지 햇갈릴 때는 해당 로직이 애그리거트의 상태를 변경하거나 애그리거트의 상태 값을 계산하는지 확인하면 된다.&#x20;

상태를 변경하거나 값을 계산 하면서 한 애그리거트에 넣기는 적합하지 않다면 도메인 서비스로 구현하자.



### 도메인 서비스 패키지 위치

도메인 서비스는 도메인 로직을 표현하므로 도메인 영역에 위치한다.

<figure><img src="../../.gitbook/assets/스크린샷 2023-02-10 오후 6.03.40.png" alt=""><figcaption></figcaption></figure>

###

### 인터페이스와 클래스 분리하기

도메인 서비스 로직이 고정되어 있지 않거나 로직이 외부 시스템을 의존한다면 인터페이스를 두고 실제 구현은 인프라 계층에서 하도록 하자.

<figure><img src="../../.gitbook/assets/스크린샷 2023-02-10 오후 6.39.41.png" alt=""><figcaption></figcaption></figure>

## &#x20;



