# CQRS

## 단일 모델의 단점&#x20;

보통 조회 기능은 여러 애그리거트에서 데이터를 가져와야 한다. 예를들어 주문 상세를 조회하려면 Order에서 주문 정보, Product에서 상품 이름, Memebr에서 회원 이름과 ID를 가져와야 한다.&#x20;

조회는 속도가 빠를수록 좋은데 이렇게 여러 애그리거트가 필요하면 JOIN 방법이나 즉시 로딩, 지연로딩 등구현 방법을 많이 고민해야 한다.&#x20;

이런 고민이 발생하는 이유는 시스템 상태를 변경할 때와 조회할 때 모두 하나의 모델을 사용하기 때문이다.&#x20;

ORM기법은 도메인 상태 변경 기능을 구현하는 데는 적합하지만 여러 애그리거트에서 데이터를 가져와 출력하는 기능을 구현하기에는 고려할게 많아서 복잡해지는 것이 원인이다.&#x20;

이런 구현 복잡도를 낮추는 방법은 바로 상태 변경을 위한 모델과 조회를 위한 모델을 분리하는 것이다.



## CQRS란&#x20;

시스템의 기능은 크게 두 가지로 나눌 수 있다. 하나는 상태 변경이다. 새로운 주문을 생성하거나, 배송지 정보를 변경하는 것 모두 상태를 변경하는 기능이다. 다른 하나는 상태 정보를 조회하는 것이다. 주문 상세, 게시글 목록 등은 모두 조회 상태를 읽고 보여주는 조회 기능이다.&#x20;

상태 변경은 주로 한 애그리거트의 상태를 변경하지만, 조회는 앞서 말했듯이 여러 애그리거트가 필요할 때가 많다.&#x20;

이렇게 상태를 변경하는 범위와 상태를 조회하는 범위가 정확하게 일치하지 않기 때문에 단일모델로는 모델이 불필요하게 복잡해진다. 이런 문제를 해결할 수 있는 것이 **CQRS**다.

**CQRS는** Command Query Responsibility Segregation의 약자로 상태를 변경하는 명령을 위한 모델과 상태를 제공하는 조회를 위한 모델을 분리하는 패턴이다.

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-15 오후 5.42.43.png" alt=""><figcaption></figcaption></figure>

위의 그림은 명령 모델과 조회 모델의 설계 예를 보여준다. 왼쪽은 상태를 변경하기 위해 객체지향 대로 충실히 모델을 설계 했고, 오른쪽은 오직 상태 조회를 위해 조회에 특화된 모델을 설계했다.&#x20;

필요하다면 명령 모델과 조회 모델이 서로 다른 DB를 사용할 수 있다.

&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-15 오후 5.46.33.png" alt=""><figcaption></figcaption></figure>

위의 그림 처럼 명령 모델은 트랜잭션을 지원하는 RDBMS를 사용하고 조회 모델은 조회 성능이 더 좋은 NoSQL를 사용하는 것처럼 말이다.

## CQRS 장단점

### 장점&#x20;

1. 조회 모델을 분리할 수 있어 덜 복잡한 도메인 설계를 할 수 있으므로 도메인 자체에 더 집중할 수 있다.&#x20;
2. 조회 성능 향상에 유리하다. 캐시나, 조회에 특화된 쿼리를 마음대로 사용할 수 있다.

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-15 오후 5.53.23.png" alt=""><figcaption></figcaption></figure>

### 단점&#x20;

1. 구현해야할 코드가 더 많아 지고 더 많은 구현 기술들이 필요하다. 도메인이 복잡하지 않거나 트래픽이 많지 않은 서비스라면 조회 전용 모델을 따로 만드는 것은 오히려 개발 비용만 늘어날 뿐이다.&#x20;
