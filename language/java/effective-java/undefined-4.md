# 람다와 스트림

## 42. 익명 클래스보다는 람다를 사용하라&#x20;

예전 자바에서는 함수 타입을 이용할 때 추상 메서드를 하나만 담은 인터페이스를 사용 했고, 이런 인터페이스의 인스턴스를 함수 객체라고 한다. 함수 객체를 만드는 주요 수단은 **익명 클래스**를 주로 이용했다. 하지만 이 방식은 **가독성이 좋지 않아** 함수형 프로그래밍에는 적합하지 않았다.&#x20;

```java
// 익명 클래스를 Comparator interface를 구현하는 인스턴스를 생성한다. 
Collections.sort(words, new Comparator<String>(){
    public int compare(String s1, String s2){
        return Integer.compare(s1.lenght(), s2.lenght());
    }
})
```

자바 8부터 이제 추상 메서드가 하나만 있는 인터페이스는 **함수형 인터페이스**이라는 특별한 의미를 가지게 되었으며, 이런 인터페이스 인스턴스를 **람다**를 사용해 만들 수 있게 되었다. 람다는 함수나 익명 클래스와 개념은 비슷하지만 코드가 **훨씬 간결**하다는 장점이 있다.&#x20;

```java
// 람다,매개 변수, 반환 값의 타입들은 꼭 타입을 명시해야 명확하지 않은 이상 생략하는게 좋다. 
Collections.sort(words, (s1, s2) -> 
Integer.compare(s1.lenght(), s2.length()); 
// 또는 word.sort(comparingInt(String::length));
```

람다는 이름이 없고 문서화도 하지 못한다. 따라서 코드 자체로 동작이 명확히 설명 되지 않거나 코드가 많아진다면 람다를 사용하지 말아야한다. &#x20;

다음과 같은 사항일 때는 람다보다는 익명 클래스를 쓰는게 좋다.

* 추상 클래스의 인스턴스를 생성할 때
* 추상 메서드가 여러 개인 인터페이스의 인스턴스 생성할 때
* 자신을 참조하는 this가 필요할 때



## 43. 람다보다는 메서드 참조를 사용하라&#x20;

* &#x20;메서드 참조는 함수 객체를 람다보다 더 간결하게 만드는 방법이다. 메서드 참조를 사용하면  아래 람다 코드처럼 더 간결하게 수정할 수 있다. &#x20;

```java
map.merge(key, 1, (count, incr) -> count+ incr); // 일반 람다 

map.merge(key, 1, Integer::sum); // Integer에 있는 정적 메서드를 메서드 참조한다. 
```

* 메서드 참조는 람다보다 간결하지만 그게 항상 좋은 점은 아니다. 람다는 매개변수의 이름 같은 것이 프로그래머에게 좋은 가이드가 될 수 있고, 메서드 참조는 경우에 따라 람다보다 더 가독성이 좋지 않은 경우도 있다.&#x20;



### 메서드 참조의 유형들

| 메서드 참조 유형   | 예시                     | 같은 코드                                                         |
| ----------- | ---------------------- | ------------------------------------------------------------- |
| 정적          | Integer::parseInt      | str -> Integer.parseInt(str)                                  |
| 한정적(인스턴스)   | Instnat.now()::isAfter | <p>Instant then = Instant.now();<br>t -> then.isAfter(t);</p> |
|  비한정적(인스턴스) | String::toLowerCase    | str -> str.toLowerCase();                                     |
| 클래스 생성자     | TreeMap\<K,V>::new     | () -> new TreeMap\<K,V>()                                     |
| 배열 생성자      | int\[]::new            | len -> new int\[len]                                          |



## 44. 표준 함수형 인터페이스를 사용하라&#x20;

자바 표준 라이브러리는 대부분 우리가 직접 작성하지 않을정도로 일반적인 표준 함수형 인터페이스를 이미 제공하고 있기 때문에 직접 만들기전에 사용할 수 있는게 있는지 찾아보자. 아래 6개만 잘 숙지하면 나머지 37개의 인터페이스도 충분히 유추할 수 있을 것이다.&#x20;

| 인터페이스              | 함수 시그니처             | 예                   | 설명                      |
| ------------------ | ------------------- | ------------------- | ----------------------- |
| UnaryOperator\<T>  | T apply(T t)        | String::toLowerCase | 인수가 1개고 인수와 반환 타입이 같음   |
| BinaryOperator\<T> | T apply(T t1, T t2) | BigInteger::add     | 인수가 2개고 인수와 반환 타입이 같음   |
| Predicate\<T>      | boolean test(T t)   | Collection::isEmpty | 인수 하나를 받고 boolean 값을 반환 |
| Function\<T, R>    | R apply(T t)        | Arrays::asList      | 인수와 반환 타입이 다르다.         |
| Supplier\<T>       | T get()             | Instant::now        | 인수를 받지 않고 결과를 반환        |
| Consumer\<T>       | void accept(T t)    | System.out::println | 인수를 하나 받고 반환값은 없음       |

* 만약 참조 타입이 아니라 기본 타입인 int, long, double을 사용하고 싶다면 이름 앞에 해당 기본 타입 이름을 붙인 인터페이스가 존재한다. 예를들어 int를 받는 Predicate는 IntPredicate가 된다.&#x20;
* Function 인터페이스는 입력과 결과의 타입이 다른데 이 때 기본 타입을 위한 인터페이스 변형은 SrcToResult 형식이다. 예를 들어 long을 받아 int를 반환하면 LongToIntFunction이 된다.&#x20;
* 인수를 2개씩 받는 변형은 BiPredicate\<T, U>, BiFunction\<T,U,R>, BiConsumer\<T,U>가 있다.&#x20;
* BooleanSuppier는 boolean을 반환하도록 하는 Supplier의 변형이다.
* 표준 함수형 인터페이스 대부분은 기본 타입만 지원한다. 그렇다고 기본 함수형 인터페이스에 박싱된 **기본 타입을 넣어 사용하면 안된다.**&#x20;

### 전용 함수형 인터페이스를 작성&#x20;

아래의 특성이 하나 이상을 가진 인터페이스라면 전용 함수형 인터페이스로 구현할지를 고려해야한다.

* 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.&#x20;
* 반드시 따라야하는 규약이 있다.&#x20;
* 유용한 디폴트 메서드를 제공할 수 있다.&#x20;

전용 함수형 인터페이스를 작성했다면 **@FunctionalInterface 애너테이션**을 잊지 말고 명시해야한다. 그 이유는 다음과 같다.&#x20;

* 이 인터페이스가 람다용으로 설계된 것을 알려준다.
* 추상 메서드가 오직 하나만 가지고 있어야 컴파일 된다.
* 때문에 누군가 메서드를 더 추가하는 것을 방지해준다.&#x20;



### 유의&#x20;

서로 다른 함수형 인터페이스를 인수로 받는 메서드들을 만들지 말자. 사용에 혼란을 일으킨다.



## 45. 스트림은 주의해서 사용하라&#x20;

### **스트림의 핵심 개념**

* 데이터 원소의 유한 혹은 무한 시퀸스
* 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현
* 소스 스트림에서 시작해 하나 이상의 중간 연산 후 종단 연산으로 끝난다.&#x20;
* 스트림 파이프라인은 지연 평가된다. 종단 연산 호출 없이는 아무 일도 하지 않을 것이다.&#x20;
* 메서드 연쇄를 지원해 여러 계산을 하나의 표현식으로 표현가능하다.
* 기본적으로 순차적으로 수행하지만 병렬도 가능하다.

### 과도한 stream

다음 코드는 반복문과 자바 8에서 추가된 computeIfAbsent 메서드를 사용한 아나그램 집합 로직이다.

```java
try (Scanner s = new Scanner(dictionary)) {
    Map<String, Set<String>> m = new HashMap<>();
    while (s.hasNext()) {
        String word = s.next();
        m.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>())
        .add(word);
    }

    for (Set<String> group : m.values()) {
        if (group.size() >= minGroupSize) {
            System.out.println(group.size() + ": " + group);
        }
    }
}
```

이 부분을 stream을 과하게 활용해보자. 분명 코드는 짧아지지만 읽기가 많이 힘들어졌고 결국 유지보수하기 어려워진다.&#x20;

```java
try(Stream<String> worlds = Files.lines(dictionary)){
            worlds.collect(
                groupingBy(world -> world.chars().sorted().collect(StringBuilder::new, (sb, c) -> sb.append((char) c), StringBuilder::append).toString())
            ).values().stream()
                .filter(group -> group.size() >= minGroupSize)
                .map(group -> group.size() + ": " + group)
                .forEach(System.out::println);
}

```

반면 이 코드는 더 명확하고 읽기 쉽다. 특히 도우미 메서드인 alphabetize를 밖으로 빼내어 가독성을 높였다.

```java
try(Stream<String> worlds = Files.lines(dictionary)){
    worlds.collect(
        groupingBy(world -> alphabetize(world))
    ).values().stream()
        .filter(group -> group.size() >= minGroupSize)
        .forEach(group -> System.out.printf(group.size() + ": " + group));
}
```

스트림 강박에 갇히지말고 반복문과 스트림을 적절히 사용해야 한다.&#x20;



### 스트림에 알맞은 로직들&#x20;

* 시퀀스를 일관되게 변환&#x20;
* 시퀀스를 필터링
* 시퀀스를 하나의 연산을 사용해 결합
* 시퀀스를 컬렉션에 모음&#x20;
* 시퀀스에서 특정 조건을 만족하는 원소를 찾음&#x20;

### 스트림에 안맞는 로직들&#x20;

* 코드 블록은 범위 안의 지역변수를 읽고 수정할 수 있다. 반면, 람다는 final 변수만 읽을 수 있고, 지역변수를 수정할 수 없다.&#x20;
* 코드 블록은 return문을 이용해 메서드에 빠져가거나 for문에서 여러 제어를 할 수 있다. 반면, 람다는 불가능하다.&#x20;
* 스트림은 각 연산 단계가 순차적으로 이루어지고 그 전의 값은 잃기 때문에 각 연산 단계마다 동시에 접근이 필요할 때는 적절하지 않다.&#x20;



## 46. 스트림에서는 부작용(side effect) 없는 함수를 사용하라&#x20;

* 순수함수란 오직 입력만이 결과에 영향을 주는 함수, 다른 상태를 참조하지 않고 변경 시키지 않음
* 스트림의 핵심은 계산을 일련의 변환으로 재구성하는 것이다. 이 때 각 변환에서는 이전 단계의 결과를 받아 처리하는 순수함수여야 한다.&#x20;
* 순수 함수를 사용하지 않아 여러 영향을 주는 함수를 사용한다면 유지보수하기 어려운 부작용이 생긴다.&#x20;
* collector라는 수집기는 스트림 사용에 있어 필수적이다. collector는 여러 스트림의 원소들을 객체 하나에 취합해 컬렉션으로 만들어준다.&#x20;
  * toList(): Stream to List
  * toSet(): Stream to Set
  * toMap(): Stream to Map, 여러 오버로딩 함수들이 있다는 것 알아두기&#x20;
  * groupingBy(): 입력으로 분류함수를 받고 이를 통해 원소들을 분류해 맵으로 담음&#x20;
  * joining(): CharSequenece 인스턴스의 스트림에만 적용 가능 스트림 원소에 특정 문자는 추가시켜 원하는 문자열을 만들 수 있다.



## 47. 반환 타입으로는 스트림보다 컬렉션이 낫다

* Collection 인터페이스는 Iterable의 하위 타입이며, Stream 메서드도 제공한다. 반복과 스트림을 동시에 지원하는 것이다.
* 따라서 원소 시퀀스를 반환하는 공개 API의 반환 타입은 Collection이 좋다.&#x20;



## 48. 스트림 병렬하는 주의해서 적용하라&#x20;

* 스트림에서의 병렬화는 성능이 오히려 더 나빠지거나 예상치 못한 결과를 발생시킬 수 있다.
* 스트림 소스가 ArrayList, HashMap, HashSet, 배열, int 범위일 때 가장 병렬화의 효과가 크다. 자료구조가 데이터들을 원하는 크기로 정확하고 쉽게 나눌 수 있고 이웃한 원소의 참조들이 메모리에 연속해서 저장되는 참조 지역성이 뛰어나기 때문이다.&#x20;
* 정말 큰 데이터가 아닌 이상 병렬화는 사용하지 않는게 좋을 것 같다.





