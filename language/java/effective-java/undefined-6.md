# 일반적인 프로그래밍 원칙

## 57. 지역변수 범위를 최소화하라&#x20;

지역변수의 유효 범위를 최소로 줄이면 코드 가독성과 유지보수성이 높아진다.

**방법:**

* 처음 쓰일 때 가깝게 선언하자. 사용하려면 멀었는데 멀리 선언하면 가독성이 떨어지고 기억이 나지 않는다.
* 지역변수는 선언과 동시에 초기화해야한다.
* for문은 딱 본체까지가 반복 변수의 범위이기 때문에 범위가 최소화 된다.
* 메서드를 작게 유지학 한 가지 기능에 집중한다.



## 58. 전통적인 for문 보다는 for-each문을 사용하라&#x20;

```java
for (Element e : elements){
    ~
}
```

* for-each는 컬렉션이든 배열이든 반복문을 처리할 수 있고 반복자와 인덱스 변수를 사용하지 않아 코드가 깔끔해진다.&#x20;
* for-each는 Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다.&#x20;
* for문에 비해 성능저하도 없다.

```java
public interface Iterable<E>{
    Iterator<E> iterator();
}
```

**사용할 수 없는상황**&#x20;

* 파괴: 컬렉션을 순회하면서 선택된 원소를 제거해야한다면 반복자의 remove 메서드를 사용해야함(자바 8부터 Collection의 removeIf메서드를 사용하면 명시적으로 순회하는 일을 피할 수 있음)
* 변형: 변형하려면 반복자나 배열의 인덱스를 사용해야함&#x20;
* 병렬: 여러 컬렉션을 병렬로 순회하려면 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야함&#x20;



## 59. 라이브러리를 익히고 사용하라&#x20;

* 라이브러리가 워낙 방대하지만 적어도 java.lang, java.util, java.io와 그 하위패키지들에 익숙해져야한다.&#x20;
* 컬렉션 프레임워크와 스트림 라이브러리, java.utiil.concurrent의 동시성 기능도 알아두자.
* 만약 표준 라이브러리에도 원하는 기능이 없다면 고품질의 서드파티 (예 구아바)를 찾아보고 진짜 없으면 직접 구현하자.&#x20;



## 60. 정확한 답이 필요하다면 float와 dobule은 피하라&#x20;

* float과 dobule이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 "근사치"로 계산하도록 설계되었다.&#x20;
* 따라서 정확한 결과에서는 사용하면 안된다.&#x20;
* 금융 계산에는 BigDecimal, int 혹은 long을 사용하자
* BigDecimal은 무거우니 만약 성능이 중요하고 소수점을 직접 추적할 수 있으며 숫자가 크지 않다면 int와 long을 사용할 수도 있다.&#x20;



## 61. 박싱된 기본 타입보다는 기본 타입을 사용하라&#x20;

* 자바의 데이터 타입은 int,double, boolean 같은 기본 타입과 String, List같은 참조 타입이 있다.&#x20;
* 각 기본 타입에는 대응하는 참조 타입이 있는데 이를 박싱된 기본 타입이라고 한다.&#x20;
* 컬렉션의 원소나 타입 매개변수일 때는 기본 타입을 지원하지 않으니 기본 타입을 원소로 하려면 박싱된 기본 타입을 사용하자.&#x20;
* 박싱된 기본 타입을 사용한다면
  * &#x20;언박싱할 떄 null을 가지면 NPE가 발생한다.&#x20;
  * 언박싱할 때 성능을 꼭 고려하자&#x20;
  * \== 연산자를 주의해서 사용하자

{% hint style="info" %}
**박싱 타입과 기본 타입의 주된차이**&#x20;

* 박싱된 기본 타입은 같은 값을 가져도 서로 다르다고 식별 될 수 있다. ==&#x20;
* 박싱된 기본 타입은 null을 가질 수 있다.&#x20;
* 기본 타입은 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율 적이다.&#x20;
{% endhint %}



## 62. 다른 타입이 적절하다면 문자열 사용을 피하라&#x20;

* 기본 타입을 문자열로 사용하지 말자&#x20;
* 열거 타입 대신 문자열로 사용하지 말자&#x20;
* 여로 요소가 혼합된 데이터를 하나의 문자열로 표현하는 것은 좋지 않다.&#x20;

번거럽고, 덜 유연하고, 느리고, 오류 가능성이 크기 때문이다.&#x20;



## 63. 문자열 연결은 느리니 주의하라&#x20;

StringBuilder를 사용하자&#x20;

StringBuffer는 멀티 쓰레드에 안전하지만 그만큼 성능은 StringBuilder보다 낮다.

## 64. 객체는 인터페이스를 사용해 참조하라&#x20;

* 적합한 인터페이스만 있다면 매개변수뿐 아니라 반환값, 변수, 필드를 전부 인터페이스로 선언하자.&#x20;
* 이런 습관은 프로그램이 훨씬 유연해진다.&#x20;
* 물론 적합한 인터페이스가 없다면 클래스를 참조하면 된다. 클래스가 계층구조를 가지고 있다면 가장 덜 구체적인 상위 타입을 참조하자&#x20;



## 65. 리플렉션 보다는 인터페이스를 사용하라&#x20;

* 리플렉션은 임의의 클래스에 접근해서 정보를 얻을 수 있고 조작할 수 있으며 메서드도 호출할 수 있다.&#x20;
* 컴파일 타임에는 알 수 없는 클래스를 사용하려면 리플렉션을 사용해야 한다.
* 단점
  * 컴파일타임 타입 검사가 주는 이점을 누리지 못한다.
  * 리플렉션을 이용하면 가독성이 좋지 않아 진다.&#x20;
  * 성능이 떨어진다.



## 66. 네이티브 메서드는 신중히 사용하라

* 네이티브 메서드란 C나 C++ 같은 네이티브 프로그래밍 언어로 작성된 메서드를 말한다.
* 과거에는 성능 최적화나 플랫폼 특화된 기능을 사용하기 위해 목적으로 네이티브 메서드를 사용하긴 했지만 현재는 JVM 성능과 기능이  많이 발전했기 때문에 네이티브 메서드를 사용할 일은 거의 없다.



## 67.최적화는 신중히 하라

최적화할 때는 다음 규칙을 따르자&#x20;

1. 하지 마라
2. 자그마한 효율성은 모두 잊자, 섣부른 최적화가 만원의 근원이다.&#x20;

* 성능 때문에 견고한 구조를 희생하지 말자. 빠른 프로그램보다는 좋은 프로그램을 작성하자&#x20;
* 좋은 프로그램이지만 성능이 나오지 않는다면 그 아키텍처 자체가 최적화할 수 있는 길을 안내해준다.&#x20;
* 좋은 프로그램은 정보 은닉 원칙을 따르고 결합도가 낮기 때문에 각 요소를 다시 설계할 수 있다.
* 성능을 제한하는 설계를 피하라. 완성 후 변경하기가 가장 어려운 설계 요소는 컴퍼넌트 끼리의 소통 방식이다. ex) API 네트워크 프로토콜)&#x20;
* API를 설계할 때 성능에 주는 영향을 고려하라.&#x20;
  * public 타입을 가변으로 만들면 불필요한 방어적 복사를 해야한다.
  * 컴포지션으로 해결할 수 있는 것을 상속으로 설계에 성능도 상위 클래스에 제약받음&#x20;
  * 특정 구현 타입에 의존해 더 빠른 구현체가 있어도 쉽게 이용하지 못함&#x20;
* 성능을 위해 API를 왜곡하지 말자.&#x20;
* 최적화를 한다면 성능 측정도 해보자. 실질적으로 크게 개선되는 점은 별로 없다고 한다.&#x20;
* 프로파일링 도구를 통해 최적화 노력을 어디에 집중할지 찾아보자. jmh는 자바 코드의 상세한 성능을 알기 쉽게 보여주는 벤치마크 프레임워크다.&#x20;



## 68. 일반적으로 통용되는 명명 규칙을 따르라&#x20;

* &#x20;자바의 명명 규칙은 크게 철자와 문법으로 나뉜다.
* 철자 규칙&#x20;
  * **패키지**와 모듈 이름은 각 요소를 점으로 구분하여 계층적으로 짓는다.&#x20;
  * 패키지는 조직의 인터넷 도메인 이름을 역순으로 사용한다. com.google.
  * 패키지 이름은 되도록 8자 이하의 짧은 단어로 한다.&#x20;
  * **클래스와 인터페이스**의 이름은 하나 이상의 단어로 이뤄지며, 각 단어는 대문자로 시작한다. 약자의 경우도 첫 글자만 대문자로 하는 쪽이 많다.
  * **메서드와 필드** 이름은 첫 글자를 소문자로 하는 것만 빼면 클래스 명명과 같다.&#x20;
  * **상수 필드**는 모두 대문자로 쓰며 단어 사이는 밑줄로 구분한다. **static final**
  * **타입 매개변수 이름**은 보통 한 문자로 표현한다. 대부분은 다음의 다섯 가지 중 하나다. 임의의 타입 T, 컬렉션 원소 타입 E, 맵의 키와 값은 K,V, 예외에는 X, 메서드 반환 타입에는 R을 사용한다.&#x20;
* 문법 규칙
  * 객체를 생성할 수 있는 클래스는 보통 단수 명사나 명사구를 사용한다.&#x20;
  * 객체를 생설 할 수 없는 클래스는 보통 복수형 명사로 짓는다.&#x20;
  * 인터페이스 이름은 클래스와 똑같이 짓거나 able ible처럼 형용사로 짓는다.&#x20;
  * 애너테이션은 규칙 없이 다양하게 사용한다.
  * 메서드의 이름은 동사나 동사구로 짓는다.&#x20;
  * boolean 값을 반환하는 메서드는 보통 is로 시작한다&#x20;
  * 반환 타입이 boolean이 아니고 해당 인스턴스의 속성을 반환하는 메서드의 이름은 보통 명사, 명사구, 혹은 get으로 시작하는 동사구로 짓는다.&#x20;
  * 객체의 타입을 바꿔서 다른 타입의 또 다른 객체를 반환하는 인스턴스 메서드의 이름은 보통 toType으로 짓는다.  (toString)
  * 객체의 내용을 다른 타입으로 바꿔 주는 것은 asType으로 짓는다 (asList)
  * 정적 팩터리의 이름은 from, of, valueOf, instance, getType 등을 사용한다.&#x20;










