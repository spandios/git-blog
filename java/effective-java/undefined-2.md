# 클래스와 인터페이스

#### 15. 클래스와 멤버의 접근 권한을 최소화하라

잘 설계된 컴퍼넌트는 내부의 데이터와 구현을 숨기고 API만을 통해 소통하는데 이를 캡슐화라고 한다.

캡슐화는 시스템을 구성하는 컴퍼넌트를 서로 독립시키므로 개발, 테스트, 분석에 도움이 된다.

컴퍼넌트 간의 독립성이 높아지므로 더 유연하며 재사용성이 높아지고 테스트하기가 더 쉬워진다.

**가장 기본 원칙은 모든 클래스와 멤버의 접근성을 가능한 좁히는 것이다.**

pulbic일 필요가 없는 것들은 package-private(default)로 좁히자.

package-private은 패키지 안에서는 접근 가능하므로 package의 내부 구현을 뜻한다.

*   접근 수준 종류

    private < package-private < protected < public 순으로 접근 권한이 넓다

    * private : 해당 클래스 안에서만 접근 가능하다
    * package-private( default): 같은 패키지 내에서만 접근 가능하다.
    * protected : 같은 패키지와 상속 받은 하위클래스도 접근 가능하다.
    * public : 어느 곳에서든지 접근 가능하다.
* 접근권한 순서

1. 공개 API를 세심히 설계한다.
2. 그 외의 모든 멤버는 priavte으로 만든다.
3. 다른 클래스가 접근해야하는 멤버에 한해 package-private으로 풀어주자

* 상속 Overiding에서 접근 제한 한계

상위 클래스의 메서드를 재정의할 때는 그 접근 수준을 상위 클래스에서보다 좁게 설정할 수 없다.

* public 클래스에선 public 필드를 가지게 하지 말자.
  * 그 필드와 관련된 것은 불변식을 보장할 수 없게 되기 때문이다. 또 스레드 또한 안전하지 않다.
  * 상수용인 public static final 필드는 허용 가능하는데 반드시 기본 타입 값이나 불변 객체를 참조해야 한다.

16. public 클래스에는 pulbic 필드가 아닌 접근자 메서드를 사용하라

가변 필드를 public로 노출하면 캡슐화 이점이 없다.

```java
class Point{
private double x;
private double y;

public Point(double x, double y){
	this.x = x;
	this.y = y;
}

public double getX(){ return x; }
public double getY(){ return y; }

public void setX(double x){this.x = x;}
public void setY(double y){this.y = y;}
}
```

#### 17. 변경 가능성을 최소화하라 (불변객체)

불변 클래스란 그 인스턴스의 내부 값을 수정할 수 없는 클래스이다. String, 기본 타입의 박싱 클래스들, BigInteger, BigDecimal이 그 예의다. 이런 불변 객체들이 간직한 정보는 객체가 파괴되는 순간까지 절대 변경되지 않는다.

불변 클래스는 가변 클래스보다 설계, 구현, 사용이 더 쉽고 안전하기 때문에 사용된다.

**불변 클래스를 만들기 위한 5가지 규칙**

* 객체의 상태를 변경하는 메서드를 제공하지 않음
* 클래스를 확장할 수 없도록 한다

하위 클래스가 객체의 상태를 변하게 막는 사태를 막아줌

* 모든 필드를 final로 선언한다

시스템이 강제하는 수단을 이용해 설계자의 의도를 명확히 드러낸다

* 모든 필드를 private으로 선언한다

필드가 참조하는 가벽 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아준다.

* 자신 외에는 내부의 가변 컴퍼넌트에 접근할 수 없도록 한다.

클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에게서 그 객체의 참조를 얻을 수 없게 한다.

**함수형 프로그래밍**

피연산자에 함수를 적용해 새로운 결과를 반환하지만 피연산자 그 자체는 수정되지 않고 그대로인 프로그래밍 패턴. 그와 달리 명령형 프로그래밍에서의 메서드에서는 피연산자인 자신을 수정해 상태가 변하게 된다.

**불변객체의 장점**

* 단순한다

가변 객체는 복잡한 상태에 놓일 수 있지만 불변객체는 데이터가 불변하기 때문에 단순한다

* 스레드에 안전하다

값이 변하지 않기 때문에 어느 스레드에나 안심하고 공유할 수 있다. 때문에 한번만든 인스턴스는 최대한 재활용하자. ex) public static final Complex ZERO = new Comple(0,0); , ex2) 자주 사용되는 인스턴스를 캐싱해 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩토리를 제공할 수 있다. 박싱된 타입 클래스 or BigInteger

* 불변객체는 그 자체로 실패 원자성을 제공한다

상태가 절대 변하지 않으니 불일치 상태에 빠질 가능성이 없다.

**불변객체의 단점**

* 값이 다르면 반드시 독립된 객체로 만들어야하기 때문에 비용이 듬.

이를 보완하는 방법은 다단계 연산들을 예측해 기본 기능으로 제공하는 것이다.

예컨대 BigIntegre는 가변 동반 클래스를 private-package에 두고 있다. String은 StringBuilder를 가변 동반 클래스로 두고 있다 (or 구닥다리 StringBuffer)

**final를 이용해 확장을 막는 방법보다 더 유연한 다른 방법**

생성자에 private을 두고 정적 팩터리를 통해 생성하게 한다. 이렇게 하면 다수의 구현 클래스를 활용한 유연성을 제공한다.

```java
private Complex(double re, double im){
	this.re = re;
	this.im = im;
}

public static Complex valueOf(double re, double im){
	return new Complex(re, im);
}
```

**정리**

1. 클래스는 꼭 필요한 경우가 아니라면 불면이여야 한다.
2. 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.
3. 마땅한 이유가 없다면 모든 필드는 private final이여야 한다.

#### 18. 상속보다는 컴포지션을 사용하라

상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다.

**안전한 사례**

* 상위 클래스와 하위 클래스 모두 같은 프로그래머가 통제하는 패키지 안일 때
* 초기부터 확장할 목적으로 설계되었고 문서화가 잘 된 클래스

안전하지 않음

* 다른 패키지의 구체 클래스를 상속은 위험하다

상속은 캡슐화를 깨트릴 수 있다\*\*.\*\* 다시 말하면 상위 클래스가 내부 구현이 변경됨에 따라 잘 동작하던 하위클래스에 문제가 생길 수 있고, 상위 클래스의 내부 데이터들을 노출 시킨다.

상속은 반드시 하위 클래스와 상위 클래스의 관계가 is-a 관계일 때만 하도록한다.

예를들어 자바 컬렉션 프레임워크에서 ArrayList는 List이기 때문에 상속이 어울리지만, Stack은 Vector가 아니기 때문에 상속하면 안됐다.

**composition (구성)**

기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 해서

구성요소로써 사용하는 방법이다. composition은 has-a로 생각하자. 예를들어 점과 삼각형은 상속 이나 composition 어떤게 어울릴까? is a로 생각해보면 “삼각형은 점이다”인데 말이 안된다. 이번엔 has a 로 생각해보자 “삼각형은 점을 가지고 있다”는 맞는 말이다. 따라서 이런관계일 때는 composition을 사용하자. 구현해야할 인터페이스가 있고 이를 재사용할 수 있도록 전달 클래스를 만들 수 있다면 더욱 어울릴 것이다.

```java
// 전달 클래스
public class ForwardingSet<E> implements Set<E>{
	private final Set<E> s;
	public ForwardingSet(Set<E> s) {this.s = s;}

	public void clear(){s.clear();}
	public boolean add(E e){s.add(e);}
	public boolean addAll(Collection<? extens E> c){s.addAll(c);}
	//~~ 
}

//래퍼 클래스(기존 객체를 감싸고 있기 때문에), 데코레이터 패턴이라고도 할 수 있다.
public class InstrumentedSet extends ForwardingSet{
	private int addCount = 0;	
	public InstrumentedSet(Set<E> s){
		super(s);
	}
	
	@Overried public boolean add(E e){
		addCount++;
		return super.add(e);
	}

	@Overried public boolean addAll(Collection<? extens E> c){
		addCount += c.size();
		return super.addAll(c);
	}

}

// 실제 사용하기
Set<instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

### 19. 상속용 클래스를 설계할 때는 상속을 잘 고려하고 문서화를 하자, 그렇지 않으면 금지하자

* 문서화

상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야한다.

이런 설명은 메서드 주석에 @implSpec 태그를 달아주면 자바doc이 자동으로 생성해줄 것이다.

* protected

효율적인 하위 클래스를 만드려면 클래스의 내부 동작 과정 중간에 끼어들 수 있는 hook을 잘 선별하여 protected 메서드 형태로 공개하자.

어떤 메서드가 protected로 노출해야하는지의 공식은 없다. 미리 예측해보고 직접 하위클래스를 만들어봐서 테스트 해야한다.

* 상속용 클래스 생성자에서 재정의 메서드를 호출하면 안된다.

상위 클래스의 생성자가 하위 클래스보다 먼저 호출된 상황에 재정의된 메서드를 호출하니 당연히 잘못된 결과가 나올 것이다.

* 상속용 클래스를 만드는 데는 많은 노력이 드니, 차라리 상속을 금지하는 것은 어떨까?

핵심 기능을 정의한 인터페이스가 있고, 클래스가 그 인터페이스를 구현한다면 상속을 금지해도 개발하는데 아무런 어려움이 없기 때문이다.

상속을 금지하려면 class를 final로선언하는 것과 생성자를 private이나 default-pacakge로 두고 정적 팩터리를 만들어주는 2가지 방법이 잇다.

### 20. 추상 클래스보다는 인터페이스를 우선하자

인터페이스는 추상 메서드들의 집합이고 추상 클래스는 하나 이상의 추상 메서드를 가진 클래스로 주로 상속용 클래스를 설계할 때 사용한다.

이 둘의 가장 큰 차이점은 추상 클래스는 구현한 메서드를 제공할 수 있는 점인데, 자바8부터는 default method를 제공해 기본 구현한 메서드를 제공할 수 있게 됐다.

때문에 이제 추상 클래스와 인터페이스의 가장 큰 차이점은 추상 클래스가 정의한 추상 메서드를 구현하는 클래스는 반드시 하위 클래스가 되어야 한다는 것이다.

#### 인터페이스의 장점

* 다중상속

자바는 단일 상속만 가능하다. 추상 클래스는 어찌됐든 클래스이므로 다중 상속이 허용되지 않는다.

반면, 인터페이스는 다중상속이 가능하다. 때문에 통해 새로운 타입 정의에도 아무 제약이 없다.

* 손쉽게 기존 클래스에도 추가 가능하다.

기존 클래스에 implements만 추가하여 구현하면 된다.

* 믹스인이 가능하다

믹스인이란 주된 클래스 기능과 더불어 선택적으로 가능한 기능을 명시하는 것을 뜻한다. 대표적으로 Comparable이 믹스인이라 할 수 있다. comparable을 구현한 클래스는 자신의 주된 기능과 더불어 인스턴스 끼리 순서를 정할 수 있는 부가적인 기능을 가지고 있다는 것을 알 수 있다.

추상 클래스는 다중상속이 안 되기때문에 믹스인을 정의하기에 힘들다.

* 계층 구조가 없는 타입 프레임워크를 만들 수 있다.

인터페이스는 상위 클래스의 생성자나 내부 내용을 신경 안 쓰고 단지 정해진 규약만 지키면 되기 때문에 구현 상 계층 구조가 없다고 볼 수 있다.

반면 클래스는 항상 상위 클래스와 연관되어 있으므로 계층 구조를 나타낼 수 밖에 없다.

* 래퍼클래스가 상속보다 더 활용성이 높다.

추상 클래스는 그 타입에 기능을 추가하는 방법은 상속 뿐이지만 인터페이스는 래퍼 클래스(컴포지션)를 통해 결합도가 낮은 구조를 만들 수 있다.

* interface도 디폴트 메서드를 제공할 수 있다.

공통되게 미리 구현해놓으면 좋은 메서드는 디폴트 메서드를 이용하자.

* 제약
  * eqauls와 hashCode는 정의하지 말아야한다.
  * 인스턴스 필드는 가지지 못한다.
  * public이 아닌 정적 멤버도 만들 수 없다. (private 정적 메서드는 예외적으로 가능)
* 인터페이스와 추상 골격 구현 클래스를 함께 제공해 인터페이스와 추상 클래스의 장점을 취할 수도 있다.

인터페이스를 통해 타입을 정의하고 필요에 따라 디폴트 메서드를 구현하며, 나머지 메서드는 골격 구현 클래스가 구현하게 두자.

가장 대표적인 예로 AbstractList가 있다. 만약 추상 골격 구현 클래스 없이 int 배열을 List로 반환하는 정적 메서드를 만드려면 아래와 같이 구현해야할 모든 메서드를 구현해야한다.

![스크린샷 2023-01-31 오후 8.38.33.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c14965db-187a-484a-bdf0-b43f4f24a223/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2023-01-31\_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE\_8.38.33.png)

![스크린샷 2023-01-31 오후 8.39.08.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/956c1c5d-5b4d-4ccf-8a08-da367cbcd028/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2023-01-31\_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE\_8.39.08.png)

반면, AbstractList 골격 구현 추상 클래스를 이용하면, 우리가 원하는 구현만 재정의하면서 손쉽게 List 인터페이스를 구현할 수 있다.

![스크린샷 2023-01-31 오후 8.41.17.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/188349ff-b02a-4ec2-a6dd-ed6ebb2673dc/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2023-01-31\_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE\_8.41.17.png)

![스크린샷 2023-01-31 오후 8.41.50.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/19eaf4ae-2aa4-43ff-aea8-13f335432ea5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2023-01-31\_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE\_8.41.50.png)

#### 추상화 클래스 장점

정말 상속이 어울리는 곳에서는 충분히 사용하면 좋다.

* 미리 공통 사항을 만들어 하위 클래스가 더 편하게 상속을 하도록 도와줄 수 있다.
* 기본 상속과 달리 추상 메서드에 대해 오버라이딩을 강제하기 때문에 규약을 지킬 수 있다.

#### 정리

인터페이스는 추상 클래스(상속)보다 확장성이 좋은 설계를 할 수 있도록 도와준다.

## 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

* 자바8부터 인터페이스에 디폴트 메서드가 추가 되었다.
* 이제 디폴트 메서드를 선언하면, 그 인터페이스를 구현하고 디폴트 메서드를 재정의하지 않은 모든 클래스에서는 디폴트 구현이 쓰이게 된다.
* 이제 자바도 기존 인터페이스에 메서드를 추가하는 길이 생겼지만 그렇다고 모든 구현체들과 호환이 된다는 보장은 없다.
* 따라서 기존 인터페이스에 새 함수는 디폴트 메서드라도 세심한 주의를 기울여야 한다.
* 물론 새 인터페이스 설계에는 디폴트 메서드는 인터페이스를 더 쉽게 사용하고 표준 작업에 유용한 수단이기 때문에 잘 활용하자.

#### 22.인터페이스는 타입을 정의하는 용도로만 사용하자

인터페이스를 구현한 클래스는 클라이언트에게 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에게 얘기해주는 것이다.

아래와 같이 상수 공개용으로 인터페이스를 사용하지 말자.

```jsx
public interface PhysicalConstants{
	static final double AVOGARDOS_NUMBER = 6.022_140_857e23;
}
```

상수를 공개학 목적이라면 아래 3가지 중 하나를 고려해보자.

* 특정 클래스나 인터페이스에 강하게 연관됐다면 그 자체에 추가하기
* enum 사용하기
* 상수 유틸리티클래스 사용하기

```jsx
public class PhysicalConstants{
	private PhysicalConstants(){}
	
public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

}
```

## 23. 태그 달린 클래스보다 클래스 계층구조를 활용하라

#### 태그 : 해당 클래스가 어떠한 타입인지에 대한 정보를 가지고 있음

아래는 태그 달린 클래스로 어떤 모양의 도형인지를 나누게 된다.

```java
class Figure{
	enum Shape{ RECTANGLE, CIRCLE};

	// 태그 필드 - 현재 모양을 나타낸다.
	final Shape shape;
	
	// 사각형일 때만 쓰임.
	double length;
	double width;
	
	// 원일 때만 쓰임 
	double radius;

	Figure(double radius){
		shape = Shape.CIRCLE;
		this.radius = radius;
	}

	Figure(double length, double width){
		shape = Shape.RECTANGLE;
		this.length = length;
		this.width = width;
	}
	
double area(){
	switch(area){
		~~
	}
}

}
```

#### 문제점

* 열거 타입을 선언, 태그 필드, switch 등 쓸데없는 코드가 많다.
* 한 클래스에 여러 구현이 섞여 있어 가독성이 나쁘다.
* 한 객체에 사각형과 원을 표현하고 있기 때문에 맡은 책임이 2개 이상이며 더 늘어날 수 있다.

#### 해결법

계층구조로 변경하자

```java
abstract class Figure{
	abstract double area();
}

class Circle extends Figure{
	final double radius;
	Circle(double radius) { this.radius = radius;}
	@Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure{
	final double length;
	final double width;
	
	Rectangle(dobule length, dobule width){
		this.length = length;
		this.width = width;
	}

	@Override dobule area() { return length * width; }
}
```

* 쓸데 없는 코드 사라짐
* 가독성이 좋아짐
* 타입 사이의 자연스러운 계층 관계도 반영 되어 유연성과 컴파일타임 타입 검사 능력을 높여줌
* 한 클래스에 하나의 책임을 가지고 있음

## 24. 멤버 클래스는 되도록 Static으로 만들어라

#### 중첩 클래스

* 중첩 클래스란 다른 클래스 안에 정의된 클래스를 말한다.
* 중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.
* 종류: 정적 멤버 클래스, (비정적) 멤버 클래스, 익명 클래스, 지역 클래스

#### 정적 멤버 클래스

* 바깥 클래스의 private 멤버에도 접근할 수 있다는 점만 제외하고는 일반 클래스와 똑같다.
* 다른 정적 멤버와 똑같은 접근 규칙을 갖는다. ex) private으로 선언되면 바깥 클래스에서만 접근 가능
*   흔히 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미로 사용 된다.

    ```java
    class Calculator{
    	
    	static enum Operantion{
    		PLUS, MINUS
    	}
    }
    ```

### 비정적 멤버 클래스

*   비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다. this를 통해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져 올 수 있다.

    ```java
    public class Parent {

        String test;

        void hello(){}

        Parent(){

        }

        class Child{
            Child(){
                Parent.this.hello(); // 메서드를 호출 할 수 있다.
    						Parent.this.test // 변수에 참조를 가져올 수 있음.
            }
        }
    }

    ```
* 바깥 클래스에 의존적이다.
*   어댑터를 정의할 때 자주 쓰인다.

    즉, 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용하는 것이다.

    ```java
    // 자신의 반복자를 구현하는 어댑터 
    public class MySet<E> extends AbstractSet<E>{
    	
    	@Override public Iterator<E> iterator(){
    		return new MyIterator();
    	}
    		
    	// 비정적 멤버 클래스 
    	private class MyIterator implements Iterator<E>{
    		
    	}
    }
    ```

#### **멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.**

* 멤버 클래스가 바깥 인스턴스의 참조를 가지기 때문에 시간과 공간이 더 소비된다.
* 더 문제는 가비지 컬렉션이 바깥 인스턴스의 메모리를 회수하지 못해 메모리 누수가 발생할 수 있다.

### 익명 클래스

* 바깥 클래스의 멤버도 아니고, 쓰이는 시점에서 선언과 동시에 인스턴스가 만들어진다.
* 주로 정적 팩터리 메서드를 구현할 때 사용 된다.

![스크린샷 2023-01-31 오후 8.41.50.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/19eaf4ae-2aa4-43ff-aea8-13f335432ea5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2023-01-31\_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE\_8.41.50.png)

#### 25. 톱레벨 클래스는 한파일에 하나만 담으라

* 톱레벨 클래스는 하나의 파일에 하나씩만 포함되어야 한다.
* 파일 당 하나의 톱레벨 클래스를 담는 것은 가독성과 유지보수에 도움이 된다.
* 각 클래스에 대한 이해가 쉬워지고 코드 수정에 있어서도 효율적이다.
* 굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스를 사용하라.
