---
description: 람다를 인자로 받거나 반환하는 함수인 고차함수에 대해 알아보자.
---

# 고차함수

* 고차 함수는 다른 함수를 인자로 받거나 함수를 반환하는 함수이다. 코틀린에서는 람다나 함수 참조를 사용해 함수를 값으로 표현할 수 있다. 따라서 함수를 일반 값처럼 함수 인자로 넘기거나 결과로 받을 수 있다.
* 람다가 코드 조각이라면 **고차 함수는 람다나 함수 참조를 인자로 받거나 다시 람다나 함수 참조를  반환하는 함수**이다.&#x20;
* 다음 필터 함수는 술어 함수를 인자로 받으므로 고차 함수이다. `list.filter {x > 0}`



## 함수 타입&#x20;

* 함수 타입을 정의하려면 함수 파라미터의 타입을 괄호 안에 넣고, 그 뒤에 화살표를 추가한 다음, 함수의 반환 타입을 지정하면 된다.&#x20;

<img src="../../../../.gitbook/assets/file.excalidraw (14).svg" alt="function type" class="gitbook-drawing">

* 함수 타입을 선언할 때는 반환 타입을 반드시 명시해야 하기 때문에 Unit이라도 생략하면 안된다.



{% hint style="info" %}
### 파라미터 이름 지정&#x20;

함수 타입에서 파라미터 타입만 작성해도 되지만 이름도 지정할 수 있다.&#x20;

<pre><code><strong>// 정의 
</strong><strong>val callback : (code: Int, content: String) -> Unit 
</strong><strong>
</strong><strong>// 이름 그대로 사용하기
</strong><strong>{code, content -> 본문 }
</strong><strong>
</strong><strong>// 그냥 원하는 이름으로 사용하기 
</strong><strong>{code, page -> 본문}
</strong><strong>
</strong>
</code></pre>
{% endhint %}



### 인자로 받은 함수를 호출하는 예

```kotlin
// predicate이름의 함수를 인자로 받아 호출한다
fun String.filter(predicate: (Char) -> Bollean) : String{
    val sb = StringBuilder()
    for (index in 0 until length){
        val element = get(index)
        if (predicate(element)) sb.append(element) // 파라미터로 전달받은 함수 호출 
    }
    return sb.toString()
}

"ab1c".filter{ it in 'a' .. 'z'} => "abc" // 람다를 전달한다. 

```



### 자바에서 코틀린 함수 타입 사용&#x20;

* 컴파일된 코드 안에서 함수 타입은 일반 인터페이스로 바뀐다.&#x20;
* 즉 함수 타입의 변수는 FunctionN 인터페이스를 구현하는 객체를 저장한다. 함수 인자의 개수에 따라 Function0\<R>, Function2\<P1, P2, R> 등의 인터페이스를 제공한다.&#x20;
* 각 인터페이스는 invoke 메서드 정의가 들어 있고 이것을 호출하면 함수를 실행할 수 있다.&#x20;
* 함수 타입의 변수는 인자 개수에 따라 각기 다른 FunctionN 인터페이스를 구현하는 클래스의 인스턴스를 저장하며, 그 클래스의 invoke 메서드 본문에는 람다의 본문이 들어간다.

```kotlin
// 인자를 2개로 받는 Function2 인터페이스
public interface Function2<in P1, in P2, out R> : kotlin.Function<R> {
    public abstract operator fun invoke(p1: P1, p2: P2): R
}
```



### 디폴트 값을 지정한 함수 타입 파라미터

* 맨 마지막 transform 인자를 보면 기본 동작 toString에 비해 현저히 호출 빈도가 낮음에도 호출 시 항상 lambda를 넘기는 것은 불편할 것이다.
* 이 때 디폴트 값을 지정해 lambda를 지정하지 않으면 기본 동작을 수행하도록 할 수 있다.

{% code lineNumbers="true" %}
```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: (T) -> String = { it.toString() }
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(transform(element))
    }
    result.append(postfix)
    return result.toString()
}

letters.joinToString()
letters.joinToString{ it.toLowerCase()}

```
{% endcode %}



### 널이 될 수 있는 함수 타입 파라미터

위의 joinToString을 널 가능성으로 구현할 수도 있다. 안전한 호출 ?.과 엘비스 연산자를 사용했다.

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: ((T) -> String)?
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(transform?.invoke(element) ?: element.toString())
    }
    result.append(postfix)
    return result.toString()
}

```

### 함수를 함수에서 반환&#x20;

함수가 함수를 인자로 받아야 할 필요가 더 많지만 함수를 반환하는 함수도 유용하다. 프로그램의 상태나 다른 조건에 따라 달라질 수 있는 로직이 있을 때를 생각해보자.&#x20;

```kotlin
enum class Delivery { STANDARD, EXPEDITED }

class Order(val itemCount: Int)

fun getShippingCostCalculator(
    delivery: Delivery
): (Order) -> Double {
    // 조건에 따라 다른 계산을 하는 함수를 반환했다. 
    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount }
    }
    return { order -> 1.2 * order.itemCount }
}

val calculator = getShippingCostCalculator(Delevery.EXPEDITED)
calculator(Order(3))
```



### 람다를 활용한 중복 제거&#x20;

함수 타입과 람다 식은 재활용하기 좋은 코드를 만들 때 쓸 수 있는 훌륭한 도구이다.&#x20;

```kotlin
fun List<SiteVisit>.averageDurationFor(os: Os) = 
    filter{it.os == os}
    .map(SiteVisit::duration)
    .average()
```

확장 함수를 이용해 os 별로 방문 시간의 평균을 구하는 함수를 정의했다. 만약 여기서 모바일 디바이스의 사용자만 얻고 싶다면 위의 방식대로는 가능하지 않아 다음과 같은 코드를 또 구현해야 한다.&#x20;

```kotlin
val averageMobileDuration =log.filter { it.os in setOf(OS.IOS, OS.ANDROID}
.map(SiteVisit::duration)
.average()
```

이 때 람다를 활용해 조건 부분만 뽑아내면 중복을 제거하고 재활용하기 좋은 코드를 만들어 사용할 수 있다.&#x20;

```kotlin
fun List<SiteVisit>.averageDurationFor(predicate : (SiteVisit) -> Boolean) =
filter(predicate).map(SiteVisit::duration).average()

log.averageDurationFor{it.os in setOf(OS.IOS, OS.ANDROID)}
log.averageDurationFor{it.os == OS.IOS && it.path == "/signup"}

```

