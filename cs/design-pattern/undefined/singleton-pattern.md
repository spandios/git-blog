# Singleton Pattern

## 사용이유와 장단점

### 사용이유

* 무상태 객체나 설계 상 유일해야할 때

### 장점

* 하나의 인스턴스만을 사용하기 때문에, 불필요한 인스턴스 생성 비용을 아낀다.

### 단점

* 멀티쓰레드 환경이라면 여러 인스턴스를 만들 경우도 있다.
* private 생성자이므로 상속이 불가하며 많은 테스트 프레임워크에서 사용 불가
* 전역변수와 같으므로 많은 곳에서 사용되면 결합도가 높아진다.

##

## 구현

```java
// Static Inner Class를 사용해 미리 로딩되어진 Pattern Instnace를 사용한다.
// 역직렬화 때도 대처함.
public class Pattern implements Serializeable{
	private Pattern(){} 

	private static class PatternHolder{
		private static final Pattern INSTANCE = new Pattern();
	}

	public static Pattern getInstance(){
		return PatternHolder.INSTNACE;
	}
	
  // 역직렬화 때 이 메서드를 반드시 실행함.
	protected Object readResoleve(){
		return getInstnace();
	}

}

```

### Enum을 통한 구현

* 왜? Reflection과 직렬, 역직렬화에 안전하다.
* 단점 사용하지 않더라도 미리 클래스가 로딩됨.

```java
public enum Pattern {
	INSTANCE;
}
```

##

## Singleton 구현 정리

1. 단순구현 ⇒ 멀티 쓰레드일 시 문제 발생 할 수 있음.
2. 이른 초기화 ⇒ static 변수로 미리 만들어 둠으로 써 사용, 미리 만든 것 자체가 사용 안하면 낭비일 수도 있음
3. syncronize ⇒ 사용 되어 질 때 동기적으로 만들 수 있음. 너무 복잡함
4. static inner class 사용하기 ⇒ 멀티쓰레드와 이른 초기화에 대응 함 Reflection 대응은 불가능.
5. enum class 사용하기 ⇒ 멀티 쓰레드, Reflection, 직렬, 역질화 대응하지만 이른초기화.
6. enum을 사용해 싱글톤 패턴을 구현하는 방버의 장점과 단점은?
   1. 장점 : 멀티쓰레드와 Reflection, 직렬화 역직렬화에 대응
   2. 단점 : 이른 초기화

#### 스프링에서

1. 스프링 빈에서 싱글톤 스코프.
