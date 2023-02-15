# Pessimistic Lock(비관적 잠금)

{% hint style="info" %}
Pessimistic 비관주의

염세주의이나 페시미즘이라고도 하며, 세계는 원래 불합리하여 비애로 가득찬 곳으로서 행복이나 희열도 덧없는 일시적인 것에 불과하다고 보는 세계관이다.

\-위키백과-
{% endhint %}



## Pessimistic Lock

비관적 잠금은 여러 사용자들이 한 데이터에 대해 접근하고 수정할 것이라고 생각한다. 다시말해 트랜잭션 충돌이 분명히 발생할 거라는 비관적인 예측을 토대로 해결책을 제시하는 것이다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2023-02-13 오후 12.05.12 (1).png" alt=""><figcaption><p>비관적 잠금</p></figcaption></figure>

비관적 잠금은 먼저 애그리거트를 조회한 스레드가 사용이 끝날 때까지 다른 스레드의 접근을 막는 방법으로 동시성 문제를 해결하는 방법이다.&#x20;

위의 그림에서 스레드1을 보면 스레드2보다 먼저 애그리거트를 조회하고 해당 row에 lock을 걸었다.그러면 스레드2 같이 다른 트랜잭션은 해당 애그리거트에 조회하기 위해 스레드1의 작업이 끝나고 트랜잭션이 해제할 때까지 기다린다. 비관적 잠금은 이런 식으로 동시성 문제를 해결하며, 보통 DBMS가 제공하는 행단위 잠금을 사용해서 구현한다.

실제 비관적 잠금을 사용하려면 스프링을 예로 Hibernate의 경우 PESSIMISTIC\_WRITE타입을 명시하면 `for update` 쿼리를 이용해 잠금을 구현한다.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE) 
@Query("select m from Member m where m.id := id") 
Optional<Member> findByIdForUpdate(@Param("id") MemberId memberId);
```



## DeadLock

비관적 잠금으로 단점 중 하나는 데드락이 발생할 수 있는 것이다. 데드락이 발생하는 이유는 무엇일까?

<img src="../../../../.gitbook/assets/file.excalidraw (1).svg" alt="DeadLock" class="gitbook-drawing">

위에 예시에선 스레드1이 주문 애그리거트 락을 보유하고 있고 스레드2가 멤버 애그리거트에 락을 보유하고 있는 것을 볼 수 있다.

문제는 스레드1이 스레드2가 이미 선점하고 있는 멤버에 조회하고 있고, 스레드2 또한 스레드1가 선점한 주문에 조회하려는 것이다. 둘은 서로가 락을 해제할 때까지 기다리지만, 누가 먼저 락을 해제하는 조건 없이 무한정 기다리기 때문에 무한 루프에 빠지며 이것을 데드 락 상태에 빠진 것이다.

스프링 데이터 JPA는 아래의 쿼리 힌트를 통해 해결하는데, 만약 지정한 시간 이상으로 lock을 보유하면 익셉션을 발생시킨다.&#x20;

```java
@Lock(LockModeType.PESSIMISTIC_WRITE) 
@QueryHints({@QueryHint(name = "javax.persistence.lock.timeout", value="2000")})
@Query("select m from Member m where m.id := id") 
Optional<Member> findByIdForUpdate(@Param("id") MemberId memberId);
```



## 장단점

### 장점

* 충돌이 자주 발생하는 경우에는 데이터를 다시 일관되게 하려는 작업인 롤백 횟수를 줄여 성능을 최적화 할 수 있다.
* 접근 자체를 막는 엄격한 조건인만큼 데이터 무결성을 보장하는 수준이 높다.

### 단점&#x20;

* 엄격한 조건 때문에 동시성은 떨어질 수 밖에 없어 성능은 손해를 볼 수 밖에 없다.
* 데드락이 발생할 수 있으니 미리 대비책을 강구해야 한다.



따라서 비관적 잠금은 데이터 충돌이 자주 발생하며 데이터 무결성이 중요할 때 사용하는 것이 좋다.



### 참고

[https://unluckyjung.github.io/db/2022/03/07/Optimistic-vs-Pessimistic-Lock/](https://unluckyjung.github.io/db/2022/03/07/Optimistic-vs-Pessimistic-Lock/)







