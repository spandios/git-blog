# 이벤트

## 시스템 간 강결합 문제

쇼핑몰에서 구매를 취소하면 환불 처리를 해야한다. 이 때 환불 기능을 실행하는 곳은 주문 도메인 엔티티 또는 응용 서비스가 될 수 있다. 먼저 주문 도메인 엔티티에서 환불을 처리하는 예를 보자.

```java
public class Order{
    ... 
    public void cancel(RefundServcie refundService){
        verifyNotYetShipped();
        this.state = OrderState.CANCLED; // 주문 로직
        
        this.refundStatus = State.REFUND_STARTED; // 결제 로직 
        refundService.refund(getPaymentdId());    
        this.refundStatus = State.REFUND_COMPLETED;
    }
    ...
}
```

`cancel` 함수는 결제 도메인의 환불 서비스인 `RefundService`를 전달받고 실행하고 있다. Order는 주문을 표현하는 도메인인데, 다른 도메인의 환불 로직에 직접 관심을 갖고 구현하고 있다. 문제는 만약 환불 로직이 변경 되면 관심을 갖은 만큼 주문 엔티티의 코드도 변경되어야 하는 것이다.&#x20;

또 다른 문제는 주문 취소가 환불뿐만 아니라 취소한 내용을 통지하는 기능이 추가 됐을 때이다. 다른 로직이 섞여 복잡해지고 트랜잭션 처리가 복잡해진다.&#x20;

```java
public class Order{
    ... 
    public void cancel(RefundServcie refundService, NotiService notiSvc){
        verifyNotYetShipped();
        this.state = OrderState.CANCLED; // 주문 로직
        
        this.refundStatus = State.REFUND_STARTED; // 결제 로직 
        notiSvc.~~  // 통지 로직
            
    }
    ...
}
```



이제 응용 서비스에서 환불 기능을 실행해보자.

```java
public class CancelOrderService{
    private RefundService refundService;
    
    @Transacational
    public void cancel(OrderNo orderNo){
        Order order = findOrder(orderNo);
        order.cancel();
        
        order.refundStart();
        refundService.refund(order.getPaymentId());
        order.refundCompleted();
    
    }
}
```

위의 도메인 엔티티의 케이스와는 다르게 각자의 관심사 대로 행동하도록 하고 있다.&#x20;

하지만 문제는 여전히 존재하는데, 외부 시스템인 RefundService가 정상이 아닐 경우 트랜잭션 처리를 어떻게 해야할지 애매하다는 것이다. 트랜잭션 롤백을 수행하지 않고 주문 취소는 반영하되 문제가 발생한 환불만 나중에 retry할 수도 있기 때문이다.&#x20;

또 다른 문제는 성능이다. 환불 처리하는 외부 시스템 RefundService의 환불 처리가 길어지면 길어질 수록 전체 주문 취소 기능의 작업도 길어진다. 프로세스가 동기적이기 때문이다.&#x20;



앞서 알아본 문제의 이유는 주문 바운디드 컨텍스트와 결제 바운디드 컨텍스트 간의 강결합(high coupling)때문이다.

이런 강한 결합을 없앨 수 있는 방법 중 하나는 이벤트이다. 특히 비동기 이벤트를 사용하면 두 컨텍스트 간의 결합을 크게 낮출 수 있다.



## 이벤트 개요&#x20;

이벤트가 발생한다는 것은 상태가 변경됐다는 것을 뜻한다. 이벤트 발생은 반드시 그 이벤트에 대응하는 반응을 구현하고 수행한다.&#x20;

도메인 모델에서도 유사하게 도메인의 상태 변경을 이벤트로 표현할 수 있다.&#x20;

보통 "\~할 때", "\~가 발생하면", "만약 \~ 하면 "과 같은 요구사항은 도메인의 상태 변경과 관련된 경우가 많고 이 때 이벤트를 이용해서 구현할 수 있다. 예를들어, '주문 취소가 발생하면 이메일을 보낸다'라는 요구사항에서 주문 최소는 도메인의 상태 변경이며 '주문이 취소 됨' 이라는 이벤트를 활용할 수 있다.



## 이벤트 구성요소&#x20;

도메인 모델에 이벤트를 도입하려면 네 개의 구성요소인 이벤트, 이벤트 생성 주체, 이벤트 디스패처(퍼블리셔), 이벤트 핸들러(구독자)를 구현해야 한다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2023-02-15 오후 12.11.28.png" alt=""><figcaption></figcaption></figure>

* 이벤트 생성 주체&#x20;

도메인 모델에서 이벤트 생성 주체는 엔티티, 밸류, 도메인 서비스와 같은 도메인 객체이다. 이들 도메인 객체는 도메인 로직을 실행해서 상태가 바뀌면 관련 이벤트를 발생시킨다.&#x20;

* 이벤트 핸들러&#x20;

이벤트 생성 주체가 발생한 이벤트를 구독하고 있다가 이벤트가 발생 시 반응한다. 이벤트 주체가 발생한 이벤트를 전달받아 이벤트에 담긴 데이터를 이용해서 원하능 기능을 실행한다.

* 이벤트 디스패처

이벤트 생성 주체와 이벤트 핸들러를 연결해준다. 디스패처는 이벤트를 자신을 구독하고 있는 이벤트 구독자들에게 전달한다.&#x20;



## 이벤트의 구성&#x20;

이벤트는 발생한 이벤트에 대한 정보를 담고 있다. 그 정보는 대략 다음과 같다.

* 이벤트 종류: 클래스 이름으로 표현&#x20;
* 이벤트 발생 시간
* 추가 데이터: 이베트 처리에 필요한 정보들

아래 이벤트는 배송지 변경 이벤트를 나타낸 것이다. 클래스 명으로 어떤 이벤트인지 표현하고 있고 이벤트 발생 시간과 이벤트 처리에 필요한 정보를 가지고 있다.

```java
public class ShippingInfoChangedEvent{ 
    private long timestamp;
    private String orderNumber;
    private ShippingInfo newShippingInfo;
}
```



배송지 변경 이벤트를 생성하는 주체는 Order 애그리거트이다. 애그리거트의 배송지 변경 기능을 구현한 메서드는 다음 코드처럼 배송지 정보를 변경 후 이벤트를 발생시킬 것이다.

다음 코드는 Events.raise()라는 디스패처를 통해 이벤트를 전파하고 있다.&#x20;

```java
public class Order{
    public void changeShippingInfo(ShippingInfo newShippingInfo){
        setShippinfgInfo(newShippingInfo);
        Event.raise(new ShippingChangedEvent(number,newShippingInfo))
    }
}
```



이렇게 전파된 이벤트를 처리하는 핸들러는 다음 코드와 같이 구현할 수 있다.

```java
public class ShippingInfoChnageHandler{
  @EventListener(ShippingInfoChangedEvent.class)
  public void handle(ShippingInfoChangedEvent evt){
    shippingInfoSynchronizer.sync(evt.getOrderNumber(), evt.getNewShippingInfo());
    // 물론 이벤트에 필요한 데이터가 없다면 다른 리소스에서 조회해야 한다
  }
}

```



## 이벤트 장점

### 관심사 분리&#x20;

이벤트를 이용하면 서로 다른 도메인 로직이 한 곳에서 섞이는 것을 방지할 수 있다.

다음은 처음에 소개한 주문 취소 코드에 이벤트를 이용해서 결합을 낮춘 코드이다.

```java
public class Order{
    public void cacnel(){
        this.state = OrderState.CANCELD;
        Events.raise(new OrderCancelEvent(number.getNumber());
    }
}
```

### 기능 확장 용이

구매 취소와 함께 이메일로 취소 내용을 보내고 싶다면 같은 이메일 발송을 처리하는 핸들러를 만들고 주문 취소 이벤트를 구독하기만 하면 된다.&#x20;



<img src="../../../../.gitbook/assets/file.excalidraw (4).svg" alt="" class="gitbook-drawing">

