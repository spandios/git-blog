# 이벤트

## 시스템 간 강결합 문제

쇼핑몰에서 구매를 취소하면 환불 처리를 해야한다. 이 때 환불 기능을 실행하는 곳은 주문 도메인 엔티티 또는 응용 서비스가 될 수 있다. 먼저 주문 도메인 엔티티에서 환불을 처리하는 예를 보자.

```kotlin
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

```kotlin
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

```kotlin
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

하지만 문제는 여전히 존재하는데, 외부 시스템인 RefundService가 정상이 아닐 경우 트랜잭션 처리를 어떻게 해야할지 애매하다는 것이다. 트랜잭션을 롤백하는 로직잉 아니라, 주문 취소는 반영하되 문제된 환불만 나중에 다시 처리할 수도 있기 때문이다.&#x20;

또 다른 문제는 성능이다. 환불 처리하는 외부 시스템 RefundService의 환불 처리가 길어지면 길어질 수록 전체 주문 취소 기능의 작업도 길어진다. 프로세스가 동기적이기 때문이다.



앞서 알아본 문제의 이유는 주문 바운디드 컨텍스트와 결제 바운디드 컨텍스트 간의 강결합(high coupling) 즉, 높은 결합도 때문이다.&#x20;

이런 강한 결합을 없앨 수 있는 방법 중 하나는 이벤트이다. 특히 비동기 이벤트를 사용하면 두 컨텍스트 간의 결합을 크게 낮출 수 있다.

