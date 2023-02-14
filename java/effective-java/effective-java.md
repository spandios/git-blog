# effective java

##

## 2장 모든 객체의 공통 메서드

equals, hashCode, toString, clone는 모두 재정의를 염두에 두고 설계된 것이다.

따라서 해당 메서드를 오버라이딩할 때는 규약에 맞게 사용해야한다.

### 10. equals는 일반 규약을 지켜 재정의하자

equals 문제를 회피하는 가장 쉬운길은 아예 재정의 하지 않는 것이다.

재정의 해야할 때는 객체 식별성이아닌 논리적 동치성을 확인해야할 때이다.

이는 주로 값 클래스에서 나타난다. (**값 클래스:** Integer나 String처럼 값을 표현하는 클래스)

즉, 클래스가 아닌 **값**을 비교하고 확인할 때 사용된다.

**동치관계**

* 반사성 : x.equlas(x)는 true
* 대칭성 : x.equlas(y)면 y.equlas(x)도 true다
* 추이성 : x.equals(y)가 true이고 y.equls(z)가 true면 x.equlas(z)도 true이다.
* 일관성 : x.equlas(y)를 반복해서 사용하면 항상 true거나 false이다.
* null-아님 : x.eauls(null)은 항상 false이다.

#### 11. equlas를 재정의한다면 hashCode도 재정의하자

* hashmap 같이 해시테이블을 사용한다면 equal이 true라도 hash code가 달라 예상한 값을 얻을 수 없다.
* 귀찮은 작업이 있으니, ide의 도움을 받자.

### 12. toString을 항상 재정의하자

재정의하지 않으면 단지 클래스 이름과@해시코드의 스트링을 보여준다.

클래스를 사용하기 즐겁고 디버깅하기 쉽게 하위 객체까지 toString을 재정의하도록하자.

객체에 명확하고 읽기 좋은 형태로 반환하도록 하자.

#### 13. clone 재정의는 주의하자

되도록 clone을 쓰지말고 복제기능은 생정자와 팩토리를 사용하자.

#### 14. Comparable을 구현할지 고려하라

순서를 고려해야하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하자.

Comparable interface의 추상 메서드인 compareTo에서 필드의 값을 비교할 때는 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자. 왜? 자바 7부터는 박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드인 compare메서드가 추가되었기 때문이다.

```java
public class Human implements Comparable<Human>{

        Integer age;

        Human(Integer age){
            this.age = age;
        }

        @Override
        public int compareTo(Human anotherHuman) {
            return age.compareTo(anotherHuman.age); // or Integer.compare(age, o.age);
        }
    }

			Human human = new Human(8);
      Human human2 = new Human(10);
      Human human3 = new Human(12);
      Human human4 = new Human(31);
      List<Human> humans = Arrays.asList(human, human2, human3, human4);
      Collections.shuffle(humans);
      List<Human> naturalOrder = humans.stream().sorted().toList();
      List<Human> reverseOrder = humans.stream().sorted(Comparator.reverseOrder()).toList();
```

1개 이상의 필드로 정렬하기

0이 아니면 순서가 정해졌으니 return 하면 되며, 가장 중요한 기준 점부터 체크해가면서 진행하면 된다.

```java
public int comareTo(PhoneNumber pn){
	int result = Short.compare(areaCode, pn.areaCode);
  if( result == 0 ){
		result = Short.compare(prefix, pn.prefix);
		if( result == 0 ){
			result = Short.compare(lineNum, pn.lineNum);
		}
	}
	return result;
}

// lambda 방식
public class Human implements Comparable<Human>{

        Integer age;

        Integer hairCnt;

        Human(Integer age, Integer hairCnt){
            this.age = age;
            this.hairCnt = hairCnt;
        }

        private static final Comparator<Human> COMPARATOR = 
				Comparator.comparingInt((Human human)->human.age)
				.thenComparingInt(human -> human.hairCnt);

        @Override
        public int compareTo(Human o) {
            return COMPARATOR.compare(this, o);
        }
    }
```

## 3장 클래스와 인터페이스

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

반면, AbstractList 골격 구현 추상 클래스를 이용하면, 우리가 원하는 구현만 재정의하면서 손쉽게 List 인터페이스를 구현할 수 있다.

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

#### 25. 톱레벨 클래스는 한파일에 하나만 담으라

* 톱레벨 클래스는 하나의 파일에 하나씩만 포함되어야 한다.
* 파일 당 하나의 톱레벨 클래스를 담는 것은 가독성과 유지보수에 도움이 된다.
* 각 클래스에 대한 이해가 쉬워지고 코드 수정에 있어서도 효율적이다.
* 굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스를 사용하라.

```java
// HelloWorld.java
// 한 파일에 톱레벨 클래스가 두개이다.
class Hello{
}

class World{

}

// 정적 멤버 클래스를 통해 톱레벨 클래스를 하나로 두었다. 
class Hello{
	void say(){
		print("Hello " + WORLD.name);
	}	
	private static class World{
		static final String name = "world";
	}
}

```

## 5장 제네릭

제네릭은 자바5부터 사용할 수 있는 문법으로 지원하기 전까지는 객체를 꺼낼 때마다 형변환을 해야 했다.

그래서 형변환 오류가 종종 나곤 했지만, 제네렉의 등장 이후로 넣을 수 있는 타입을 컴파일러에게 알려줬다. 그래서 컴파일러는 알아서 형변환 코드를 추가할 수 있게 되고, 엉뚱한 타입의 객체를 넣으려는 시도를 원천 차단 가능해졌다.

#### 용어정리

* 제네릭 클래스: 클래스와 인터페이스 선언에 타입 매개변수가 쓰임
* 제네릭 타입: 제네릭 클래스와 인터페이스 ex) List
* unbounded wildcard : 제네릭은 사용하고 싶지만 그 타입이 정확히 무엇인지는 신경 쓰고 싶지 않을 때 ?를 쓰자. ex) List\<?>
* upper bounded wildcard : List
* lower bounded wildcard : List
* 제네릭 메서드: static List asList(E\[] a)
* recursive type bound : \<T extends Comparable>
* 타입 토큰 : ex) String.class

#### 26. 로 타입은 절대 사용하지 말자.

* raw type이란 타입 매개변수가 없는 제네릭을 뜻한다.
* 로 타입은 제네릭이 주는 안정성과 표현력을 잃는다.
* 단지 호환성 때문에 존재
*   List는 모든 타입을 허용한다는 의사이지만, 로 타입인 List는 제네릭에 완전히 발을 뺀 셈이다.

    27. 비검사 경고를 제거하라
    28. 제네릭은 컴파일러가 안전하지 않은 코드에 대해 경고를 보내준다.
    29. 할 수 있는 한 모든 경고를 제거하자. ClassCastException을 일으킬 수 있는 잠재적 가능성을 줄여준다.
    30. 경고를 제거할 수는 없지만 타입 안전하다고 확신한다면 @SuppressWarnings(”uncheck”) 애너테이션을 달아 경고를 숨기자. 이 애너테이션은 가능한 좁은 범위로 사용하자.그 후 경고를 숨기기로 한 근거를 주석에 남기자.

    #### 28. 배열 보다는 리스트를 사용하자

    제네릭은 타입 정보가 런타임에는 소거가 된다. 취지 자체가 컴파일 단계에서 컴파일러에게 타입을 알려주고 형변환과 타입 체크를 하기 위해서기 때문이다. 반면, 배열은 런타임시에도 타입을 인지하고 확인하다. 때문에 서로의 호환이 좋지 않다. 그래서 배열 보다는 리스트를 사용하는게 런타임 시 ClassCastException을 만날일이 없으니 좋다.

    1. 이왕이면 제네릭 타입으로 만들라
    2. 클라에서 직접 형변환하는 것보다 제네릭 타입이 더 안전하고 편함
    3. 제네릭 배열 생성을 금지하는 제약 우회하기

    `elements = (E[]) new Object[10];`

    1. 이왕이면 제네릭 메서드로 만들라

    ```kotlin
    public static <E> Set<E> union(Set<E> s1, Set<E> s2){
    	Set<E> result = new Hashset<>(s1);
    	result.addAll(s2);
    	return result;
    }

    // 재귀적 타입 한정, 모든 타입 E는 자기 자신과 비교할 수 있음(Comparable을 구현함)
    public static <E extends Comparable<E>> E max(Collection<E> c);

    ```

    #### 31. 한정적 와일드카드를 사용해 API 유연성을 높이자

    매개변수화 타입은 불공변이다. 즉, 서로 다른 타입 Type1과 Type2가 있을 때 List은 List의 하위 타입도 상위 타입도 아니다.

    하지만 때론 불공변 방식보다 유연할 때가 필요하다. 이 때 한정적 와일드 카드를 사용하자.

    extneds: 하위 타입, super : 상위 타입

    Iterable\<? extends E> : E를 포함한 E의 하위 타입

    Iterable\<? s uperE> : E를 포함한 E의 상위 타입

    #### 공식

    * 매개변수화 타입 T가 생산자면 \<? extends T>

    ```kotlin
    //src를 통해 사용할 E 인스턴스를 사용 중 
    public void pushAll(Iterable<? extends E> src){ 
    	for (E e : src) 
    			push(e);
    }
    ```

    * 소비자면 \<? super T>

    ```kotlin
    // dst가 이 E 인스턴스를 소비한다.
    public void popAll(Collection<? super E> dist){
    	while(!isEmpty()) dst.add(pop())
    }
    ```

    ```kotlin
    public class Chooser<T>{
    	private final List<T> choiceList;
    	public Choose(Collection<? extends T> choices){
    		choiceList = choices;
    	}
    }
    // Interger는 Number의 하위타입이다.
    List<Integer> intergers = listOf(1,2);
    Choose<Number> chooser = new Chooser(integers); // 만약 한정적 와일드카드가 아니였다면 못 넣음. 
    ```

    #### 매개변수 vs 인수

    ```kotlin
    fun add(value : Int){ // value => 매개변수 
    }
    add(10) // 실제 값 인수 

    class Set<T> => T는 타입 매개변수
    Set<Interger> => Interger는 타입 인수 
    ```

    #### Comparable, Comparator

    이 인터페이스는 언제나 소비하기 때문에 일반적으로 Comparable\<? super E>를 사용하는게 더 낫다.

    왜냐하면 이 한정적 와일드 카드를 사용하지 않는다면 만약 상위 타입이 comparable을 구현했다면 상위타입과 하위타입 둘 중 어떤 인스턴스와 Comparable를 할 지 모르기 때문이다.

    1. 제네릭과 가변인수를 함께 쓸 때는 신중하자

    가변인수와 제네릭은 궁합이 좋지 않다. 간변인수 기능은 배열을 노출해 추상화가 완벽하지 않고, 배열과 제네릭 타입의 규칙이 서로 다르기 때문이다.



### 33. 타입 안전 이종 컨테이너를 고려하자

* 일반적인 제네릭 형태는 한 컨테이너가 다룰 수 있는 타입 매개변수가 정해져 있다.
* 하지만 각 타입의 class 객체를 매개변수화한 키 역할로 사용하면 제약 없는 타입 안전 이종 컨테이너를 사용 가능하다.
* class의 클래스가 제네릭. class 리터럴 타입은 Class다. ex) String.class 타입은 Class이다.
* 한편, 컴파일타임 타입 정보와 런타임 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴을 **타입 토큰**이라 한다. ex) String.class

```kotlin
//api
public class Favorites{
	public <T> void putFavorite(Class<T> type, T instance);
	public <T> T getFavorite(Class<T> type);
}

// 클라이언트 
public static void main(String[] args){
	Favorites f = new Favorites();
	f.putFavorite(String.class, "Java");
	f.putFavroite(Class.class, Favorite.class);
	String favroiteString = f.getFavorite(String.class);
	Class<?> favoriteClass =f.getFavorite(Class.class);
}

// 구현 
public class Favorites{
 // 와일드카드로 어떤 클래스의 타입이 와도 된다. 
	private Map<Class<?>, Object> favorites = new HashMap<>();

	public <T> void putFavorite(Class<T> type, T instance){
		favorites.put(Objects.requireNonNull(type), instance);
	}
	
	public <T> T getFavorite(Class<T> type){
		return type.cast(favorites.get(type)); // 형변환 연산자의 동적 버전
	}
}

```

* ? → wildcard 제네릭은 로 타입과 다르게 어떤 원소도(컬렉션에서) 넣을 수 없다. 따라서 타입 불변성을 지킬 수 있으며 꺼낼 수 있는 타입도 전혀 알 수 없게 해준다.
