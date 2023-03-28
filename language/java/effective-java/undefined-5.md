# 메서드

## 49. 매개변수가 유효한지 검사하라

* 잘못된 값으로 메서드 본체가 실행되기 전에 매개변수를 제대로 검사함으로써 오류를 뒤늦게 파악하지 않는 것이 중요하다.&#x20;
* public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화하자. (@throws 자바독 태그)
* 자바 7에 추가된 java.util.Objects.requireNonNull을 사용해 널 검사를 편하게 하자&#x20;
* 공개되지 않은 private 메서드라면 유효한 값이 넘겨질것이라는 것을 확신해야한다. 이럴 땐 assert를 통해 유효성 검사를 코드 안에 내포할 수 있다.&#x20;



## 50.적시에 방어적 복사본을 만들라&#x20;

* 클라이언트가 우리 코드의 불변식을 깨트릴거라 가정하고 방어적으로 프로그래밍하는 습관을 가지자.&#x20;
* 인스턴스 내부의 객체를 바꾸려는 외부의 공격을 막으려면 매개변수 각각을 방어적으로 복사해야한다.
* 다음과 같이 복사본을 만들고 필요하다면 이후에 유효성 검사를 한다. 이 순서가 부자연스러워보여도 멀티스레딩환경이라면 원본 객체를 수정할 위험이 있기 때문이다.

```java
public Period(Date start , Date end){
    this.start = new Date(stat.getTime());
    this.end = new Date(end.getTime());
    
    if(this.start.compareTo(this.end) > 0){
        throw new IllegalArgumentException();
    }
}
```

* 위에서 clone메서드를 이용해서 복사하지 않았는데, 그 이유는 복사할 인스턴스가 final이 아니라면 이를 확장하는 하위 클래스가 생성될 수 있다. 이 때 clone이 악의를 가진 하위 클래스를 반환할 수 있기 때문이다.&#x20;
* 접근자 메서드가 가변 인스턴스를 반환하면 이 또한 방어적 복사본을 반환하면 된다.&#x20;

```java
public Date start(){
    return new Date(start.getTime());
}
```

* 이런 복사가 불변 객체를 만들기위해서만 아니다. 클라이언트가 제공한 객체 참조를 내부에서 보관할 때면 그 객체가 잠재적으로 변경될 수 있는지 고민해야한다.&#x20;
* 내부 객체를 클라이언트에게 건내줄 때 방어적 복사본을 만드는 이유도 위와 같다. 항상 심사숙고하는 습관을 갖자.&#x20;
* 복사 비용이 너무 크거나 클라이언트가 잘못 수정할일이 없다면 복사본을 만드는 대신 문서화를 통해 수정에 대한 잘못된 결과는 클라이언트에게 책임이 있다는 것을 알려주자.



## 51. 메서드 시그니처를 신중히 설계하라&#x20;

* 메서드 이름을 신중히 짓기: 이해할 수 있고, 같은 패키지에 속한 다른 이름들과 일관되게 짓도록 하자.&#x20;
* 편의 메서드를 너무 많이 만들지 않기: 모든 메서드는 각각의 자신의 소임을 다해야 한다. 편의를 위해 자신의 소임보다 너무 많은 메서드를 가진다면 사용하기 힘들고 이해하기 힘들게 된다.
* 매개변수 목록은 짧게 유지하자: 4개 이하로 가독성 좋게 만들자. 특히 같은 타입의 매개변수가 연달아 나오는 경우 실수를 많이 유발한다. 다음은 매개변수 목록을 짧게 줄여주는 방법이다.
  * 한번에 많은 기능을 담은 메서드 보다는 여러 메서드로 쪼갠다. 기능을 원자적으로 쪼개어 이를 조합해가면서 복잡한 기능까지 구현하는 것이 오히려 메서드를 적게 만들고 테스트하기도 좋다.&#x20;
  * 매개변수 여러개를 묶어주는 도우미 클래스를 만든다. 이런 클래스들은 일반적으로 정적 멤버 클래스로 둔다.
  * 모든 매개변수를 갖는 추상화된 객체를 갖고 필요한 매개변수만 세터로 설정하고 넘겨주는 방법.
* 매개변수 타입은 클래스보다는 인터페이스가 확장성이 좋다.&#x20;
* 정적 팩토리 메서드처럼 어떤 값에 따라 다른 객체를 반환하는 메서드는 boolean보다는 열거타입을 사용하는 것이 가독성이 더 좋고 확장성도 좋다.&#x20;



## 52. 다중정의(overloading)는 신중히 사용하라&#x20;

* 재정의 (overriding)은 동적으로 선택되지만, 다중정의(overloading)은 정적으로 선택된다.&#x20;
* 즉, 다중정의 선택은 단순히 컴파일시점에서 매개변수의 타입에 의해 이뤄진다.
* 잘못된 결과를 내는 오버로딩 사용 사례. c의 타입은 실제 런타입 때는 각기 달라지겠지만 컴파일 시점에는 모두 Collcetion이기 때문에 "그 외"만 3번 호출된다.&#x20;

```java
public class CollectionClassifier{
    public static String classify(Set<?> s){
        return "집합";
    }
    public static String classify(List<?> list){
        return "리스트";
    }
    
    public static String classify(Collection<?> c){
        return "그 외";
    }
    
    public static void main(String[] args){
        Collection<?>[] collections = {new HashSet<String>(), new ArrayList<BigInteger>(), new HashMap<String,String>().values()};
        for (Collcetion<?> c : collections){
            System.out.println(classify(c));
        }
    }
    
}
```

* 매개변수 수가 같은 다중정의는 되도록 만들지 말자. 메서드 이름을 다르게 짓는 방법도 있다.&#x20;





## 53. 가변인수는 신중히 사용하자&#x20;

* &#x20;인수 개수가 일정하지 않은 메서드를 재정의한다면 반드시 필요하다.
* 필수 매개변수는 앞에 두고, 그 뒤에 가변인수를 취하자
* 가변인수 메서드는 호출될 때마다 배열을 하나 할당하고 초기화하는 것을 고려하자.&#x20;



## 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라&#x20;

* null 이아닌, 빈 배열이나 컬렉션을 반환해 클라이언트가 null 체크하는 번거러움을 걷어주자.&#x20;
* null이 아닌 컬렉션이나 배열을 생성한다고 성능을 걱정하지말자.&#x20;



## 55. 옵셔널 반환은 신중히 하라&#x20;

* 자바 8부터 사용가능한 Optional\<T>은 null 이 아닌 T 타입 참조를 하나 담거나 아무것도 담지 않을 수 있다.&#x20;
* 결과가 없을 수 있고 이 때 처리해야하는 상황이 있다면 Optional을 사용하면 좋다.
* Optional을 반환하는 메서드는 절대 null을 반환하지 말자.&#x20;
* 컬렉션, 스트림, 배열 같은 컨테이너 타입은 옵셔널로 감싸지 말자.&#x20;
* OptionalInt, OptionalLong처럼 기본타입용 optional도 준비했다.
* 성능에 민감하다면 null이나 예외를 던지는게 나을 수 있다.



### 56. 공개된 API 요소에는 항상 문서화 주석을 작성하자&#x20;

* 자바독은 소스코드 파일에서 문서화 주석이라는 특수한 형태로 기술된 설명을 기반으로 API 문서로 변환해준다.&#x20;
* 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 선언을 하는것이 올바른 문서화의 방법이다.&#x20;
* 어떻게 보다는 무엇을 하는지 기술하는게 좋다.&#x20;
* @param : 매개변수에 대한 설명
* @return : 리턴 값에 대한 설명
* @throws: 어떤 조건에서 어떤 예외가 발생하는지

```javadoc
   /**
     * Returns the element at the specified position in this list.
     *
     * @param index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= size()})
     */
    E get(int index);
```

* {@code}: 태그로 감싼 곳을 코드 폰트로 렌더링한다.
* @implSpec: 해당 메서드와 하위 클래스 사이의 계약을 설명, 하위 클래스들이 그 메서드를 상속하거나 super 키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지를 명확히 인지하고 사용할 수 있도록 설명한다.&#x20;
* @summary: 첫번째 문장은 이 요소의 요약설명이다.
* {@index}: 해당 요소를 검색할 수 있게 해준다.&#x20;
* 제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 달아야 한다. &#x20;

```javadoc
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
public interface Map<K, V> {
}
```

* 열거 타입도 상수들에 주석을 달아야 한다.&#x20;

```java
/**
 * An instrument section of a symphony orchestra.
**/
public enum OrchestraSection{
    /** Wodwinds ~~ */
    WOODWIND,
    /** Brass ~~ */
    BRASS,
}
```

* 애너테이션 타입의 멤버들도 주석을 달아야한다.&#x20;

```java
/**
 * Indicates that the annotated method is a test method that
 * must throw the designated exception to pass.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    /**
     * The exception that the test method is expected to throw ~
     * 
     */
    Class<? extends Throwable> value();
}

```

* 스레드 안전성과 직렬화 가능성도 문서화하자.&#x20;
* 자바독은 메서드 주석을 상속할 수 있다. 문서화 주석이 없는 API요소를 발견하면 자바독이 가장 가까운 문서화 주석을 사용할 것이다.





