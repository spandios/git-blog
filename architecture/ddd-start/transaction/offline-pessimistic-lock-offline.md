# Offline Pessimistic Lock(Offline 비관적 잠금)

오프라인 비관적 잠금은 트랜잭션 범위를 기준으로 잠금을 건다. 첫 번째 트랜잭션을 시작할 때 오프라인 잠금을 걸고, 마지막 트랜잭션에서 잠금을 해제한다. 때문에 다른 스레드가 실질적인 수정이 아닌 단지 수정폼 같은 조회 요청을 해도 락을 얻을 수 없어 실패한다.

예를 들어 수정 기능에서는 폼을 보여주는 트랜잭션 하나와 데이터를 수정하는 트랜잭션 둘로 볼 수 있다.&#x20;

오프라인 선점 잠금은 트랜잭션 1에서 부터 lock을 걸고 실제 수정이 완료되면 lock을 해제하는 방식을 사용하기 때문에  다른 사용자의 수정 폼 요청도 허락하지 않게 할 수 있다.&#x20;


