# 제네릭 타입 파라미터

* 제니릭을 사용하면 타입 파라미터를 받는 타입을 정의할 수 있다.&#x20;
* 제네릭 타입의 인스턴스를 만들려면 타입 파라미터를 구체적인 타입 인자로 치환해야 한다. 코틀린 컴파일러는 보통 타입과 마찬가지로 타입 인자도 추론할 수 있다.
* listOf에 전달된 값이 문자열 타입이기 때문에 컴파일러는 List\<String>임을 추론한다. &#x20;

`val authors = listOf("HEO","Dmitry")`&#x20;

* 반면 빈 리스트를 만들 때는 타입 인자를 추론할 수 없기 때문에 직접 타인 인자를 명시해야 한다.&#x20;

`val aturhos = listOf<String>() || val autrhos : List<String> = listOf()`

{% hint style="info" %}
코틀린은 반드시 제네릭 타입의 타입 인자를 명시하거나 컴파일러가 추론할 수 있어야한다.

그와 반면 자바는 제네릭이 없던 버전과 호환하기 위해 타입 인자가 없는 제네릭 타입(raw) 타입을 허용하지만, 절대 사용하지 말아야한다.&#x20;
{% endhint %}



## 제네릭 함수와 프로퍼티&#x20;

* 제네릭 함수를 사용하려면 반드시 구체적 타입으로 타입 인자를 넘겨야 한다. 아래 slice 함수의 정의를 살펴보자.&#x20;

<img src="../../../../.gitbook/assets/file.excalidraw (26).svg" alt="" class="gitbook-drawing">

```kotlin
// 타입 인자를 명시할 수 있지만 컴파일러가 추론 가능하기 때문에 생략할 수 있다.
val letters = ('a' .. 'z').toList()
letters.slice(10 .. 13) // 컴파일러가 T가 Char라는 사실을 추론가능하다.
letters.slice<Char>(10..13) // 직접 명시할수도 있다.
```

* 제네릭은 고차함수에서도 사용 가능하다. 컴파일러는 filter가 List\<T> 타입의 리스트에서 호출될 수 있다는 사실과 filter의 수신 객체인 List의 타입인자를 확인해 T가 어떤 타입인지 추론할 것이다.&#x20;

```kotlin
fun <T> List<T>.filter(predicate: (T) -> Boolean): List<T>
```

* 클래스나 인터페이스 안에 정의된 메서드, 확장 함수 또는 최상위 함수에서 타입 파라미터를 선언할 수 있다.&#x20;
* 제네릭 확장 프로퍼티도 선언할 수 있다.&#x20;

```kotlin
val <T> List<T>.penultimate: T 
    get() = this[size - 2]
```



## 제네릭 클래스 선언&#x20;

* 자바와 마찬가지로 코틀린에서도 타입 파라미터를 넣은 꺽쇠 기호를 이름뒤에 붙이면 클래스와 인터페이스를 제네릭하게 만들 수 있다.&#x20;

```kotlin
interface List<T>{
    operator fun get(index: Int): T 
}
```

* 제네릭 클래스를 확장하는 클래스를 정의하려면 기반 타입의 제네릭 파라미터에 대해 타입 인자를 정해야 한다.
* 구체적인 타입을 지정하기

```kotlin
class StringList: List<String>{
    override fun get(index: Int):String = ...
}
```

* 타입 파라미터로 받은 타입을 그대로 넘길 수도 있다.&#x20;

```kotlin
class ArrayList<T>: List<T>{
    override fun get(index:Int): T = ...
}
```

* 자기 자신을 타입 인자로 참조할 수도 있다. Comparable 인터페이스를 구현하는 클래스가 이런 패턴의 예이다.

```kotlin
interface Comparable<T>{
    fun compareTo(other: T): Int
}

class String: Comparable<String>{ // 자기 자신의 타입을 참조한다. 
    override fun compareTo(other:String): Int = ...
}
```



## 타입 파라미터 제약&#x20;

타입 파라미터 제약은 클래스나 함수에 사용할 수 있는 타입 인자를 제한하는 기능이다.&#x20;

### 상한 (Upper bound)

어떤 제네릭 타입 파라미터를 **상한(upper bound)**으로 지정하면 그 제네릭 타입을 인스턴스화 할 때 사용하는 타입인자는 반드시 그 상한 타입이거나 그 상한 타입의 하위 타입(하위 클래스)이여야 한다.&#x20;

모든 원소의 합인 sum 함수는 리스트 원소를 숫자 타입만 받아야 한다. 다음 코드를 통해 해결할 수 있다.

```kotlin
fun <T : Number> List<T>.sum() : T 

// java 
<T extends Number> T sum(List<T> list)
```

타입 파라미터 T 에대한 상한을 정하고 나면 T 타입의 값을 그 상한 타입의 값으로 취급할 수 있다.&#x20;

다음은 두 파라미터 사이에서 더 큰 값을 찾는 제네릭 함수이다.&#x20;

```kotlin
fun <T: Comparable<T>> max(first:T, second: T) : T{
    return if (first > second) first else second
}

max("kotlin", "java") => kotlin
max("kotlin", 42) => error // string과 숫자를 비교할 순 없다.
```

* 타입 파라미터에 대해 둘 이상의 제약을 가해야 하는 경우 아래와 같이 where을 쓴다.&#x20;

```kotlin
fun <T> ensureTrailingPeriod(seq: T) where T: CharSequence, T: Appendable{

}    
```



### 타입 파라미터를 널이 될 수 없는 타입으로 한정

아무런 상한을 정하지 않은 타입 파라미터는 결과적으로 Any?를 상한으로 정한 파라미터와 같기 때문에 널이 될 수 있는 타입을 받을 수 있다. 항상 널이 될 수 없는 타입만 타입 인자로 받게 만드려면 Any? 대신 Any를 상한으로 하자.&#x20;



###
