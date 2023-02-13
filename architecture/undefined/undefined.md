# 도메인 모델

### 도메인 모델

도메인 모델이란 특정 도메인을 개념적으로 표현한 것이다. 이런 표현을 통해 우리는 각기 다른 전문분야의 기여자들과 도메인 지식을 원활히 동기화 할 수 있다. 표현 방법은 UML기법, 그림, 수학 공식 등 도메인을 이해하는데 도움만 된다면 무엇이든지 가능하다.

### 도메인 모델 분류

도메인 모델은 개념모델과 구현 모델로 나누어 생각할 수 있다. 개념은 앞서 말했듯이 코드를 고려하지 않고 도메인을 개념적으로 본 것이고, 구현모델은 객체지향을 토대로 도메인의 규칙을 실제로 코드로써 구현한 것이다.&#x20;

### 엔티티와 밸류

이렇게 도출된 도메인 모델은 엔티티와 밸류로 구분할 수 있다.

* 엔티티 타입: 식별자를 가지는 테이블 객체라고 할 수 있다.

```kotlin
class Order{
	String orderNo; // 식별자이다.
	
}
```

* 밸류 타입 : 식별자 없이 개념적으로 어떤 도메인을 표현한다.

#### 밸류 타입 장점&#x20;

1. 기본형 타입을 대체해 좀 더 가독성이 좋은 코드를 작성할 수 있다.

```kotlin
public class OrderLine{
	private Product product;
	private int price; // price는 int타입의 숫자이지만 더 정확하게는 돈을 의미한다.
}

// 밸류타입
public class Money{
	private int value;
	
	public Money(int value){
		this.value = value;
	}
}

public class OrderLine{
	private Product product;
	private Money money; // 돈이란 것을 더 쉽게 알 수 있다.
}
```

2. 밸류 타입에 기능을 추가할 수 있다

```kotlin
public class Money{
	private final int value;
	
	// 단지 값을 더하는 연산이 아니라 돈 계산 중 덧셈을 명확히 나타낼 수 있다.
	Money add(Money money){
		return new Money(this.value + money.value);
	}	

	public Money(int value){
		this.value = value;
	}
}
```

3. 불변으로 사용하자

의도되지 않은 데이터 변경에 안전할 뿐 아니라, 복잡한 상태를 가지지 않기 때문에 밸류 타입에 적합하다.

```kotlin
public class Money{
	private final int value;
	
	// 단지 값을 더하는 연산이 아니라 돈 계산 중 덧셈을 명확히 나타낼 수 있다.
	Money add(Money money){
		return new Money(this.value + money.value);
	}	

	public Money(int value){
		this.value = value;
	}
}
```

### 도메인 용어와 유비쿼터스언어

실제 도메인에 사용되는 도메인 용어를 코드에 반영해야한다. 개발자가 이 용어를 개발에서 도메인으로 또는 도메인에서 개발로 변환해 생각하는 불필요한 변환 과정을 건너뛸 수 있다. 최대한 도메인 용어를 활용하면 가독성을 높일 수 있고, 코드를 분석할 때 더 쉬우며, 도메인 전문가와 소통할 때도 불필요한 커뮤니케이션 비용을 덜 수 있다.

```kotlin
// 아래와 같은 방식은 STEP1이 도메인 단계에서는 어떤 용어인지를 다시 생각해내야한다.
public enum OrderState{
	STEP1, STEP2, STEP3
}

public enum OrderState{
	PAYMENT_WAITING, PREPARING, SHIPPED
}
```

에릭 에반스는 도메인 주도 설계에서 언어의 중요함을 강조하기 위해 유비쿼터스 언어라는 용어를 사용했다. 도메인과 관련된 공통의 언어를 정의해 소통, 문서, 코드, 테스트 등 모든 곳에서 같은 용어를 사용한다. 이렇게 하면 소통 과정에서 발생하는 용어의 모호함을 줄일 수 있고 개발자는 도메인과 코드 사이에서 불필요한 해석 과정을 줄일 수 있는 것이다.

