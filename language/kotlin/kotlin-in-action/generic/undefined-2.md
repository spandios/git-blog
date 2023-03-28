# 변성: 제네릭과 하위타입

변성개념은 List\<String>와 List\<Any>와 같이 기저 타입이 같고 타입 인자가 다른 여러 타입이 서로 어떤 관계가 있는지 설명하는 개념이다.&#x20;



## 변성이 있는 이유: 인자를 함수에 넘기기&#x20;

List\<Any> 타입의 파라미터를 받는 함수에 List\<String>을 넘기면 안전할까?&#x20;

만약 원소 추가나 변경이 없는 경우는 안전할 것이다.&#x20;

```kotlin
fun printContents(list: List<Any>){
    println(list.joinToString())
}
printContents(listOf("abc","bac"))
printContents(listOf(1, 2))
```

반면 원소를 추가하거나 변경이 있는 경우 타입 불일치가 생길 수 있어 안전하지 않다.&#x20;

```kotlin
fun addAnswer(list:MutableList<Any>){
    list.add(42)
}
val list = mutableListOf("abc", "bac")
addAnswer(list)
list.maxBy {it.length} // 실행 시점에서 예외가 발생할 것이다.

// 안전하기 위해선 
fun addAnswer(list:MutableList<Integer>){
    list.add(42)
}

```

코틀린에서는 읽기 전용 인터페이스와 변경 가능한 인터페이스를 선택해 안전하지 못한 함수 호출을 막을 수 있다.&#x20;

함수의 파라미터가 읽기 전용 인터페이스로 받는다면 더 구체적인 타입의 원소를 갖는 리스트를 (List\<String>) 그 함수에 넘길 수 있다. (List\<Any> 하지만 변경 가능한 인터페이스라면 불가능하다.&#x20;



## 클래스, 타입, 하위타입&#x20;

전에 설명에서는 변수의 타입이 그 변수에 담을 수 있는 값의 집합을 지정한다고 했다. 타입과 클래스라는 용어를 자주 혼용해왔는데 사실 둘은 같지 않다.  클래스 이름을 바로 타입으로 쓸 수 있다. 하지만 널이 될 수 있다는 타입에도 또 같은 이름을 쓴다. 이는 모든 코틀린 클래스가 적어도 둘 이상의 타입을 구성할 수 있다.&#x20;

제네릭 클래스에서는 제대로 된 타입을 얻으려면 제네릭 타입의 타입 파라미터를 구체적인 타입 인자로 바꿔줘야한다. 각각의 제네릭 클래스는 무수히 많은 타입을 만들어낼 수 있다. 타입 사이의 관계를 논하기 위해서는 **하위 타입**이라는 개념을 잘 알아야 한다. 어떤 타입 A의 값이 필요한 장소에 어떤 타입 B의 값을 넣어도 아무 문제가 없다면 타입 B는 타입 A의 하위 타입이다.&#x20;

예를 들어 Int는 Number의 하위 타입이다. 상위 타입은 하위 타입의 반대이다. A 타입이 B의 하위 타입이면 B는 A의 상위 타입이다.&#x20;

```kotlin
fun test(i: Int){
    val n: Number = i // Number는 Int의 상위 타입이기 때문에 대입 가능, Int는 하위 타입이다.
}
```

간단한 경우 하위 타입은 하위 클래스와 근본적으로 같다. Int 클래스는 Number의 하위 클래스 이므로 Int는 Number의 하위 타입이다.

널이 될수 있는 타입은 하위 타입과 하위 클래스가 같지 않다는 경우를 보여준다. 널이 될 수 없는 타입은 널이 될 수 있는 타입의 하위 타입이다. null을 널이 될 수 없는 변수에 대입할 수는 없기 때문이다.&#x20;

제네렉 타입에 대해 이야기할 때 특히 하위 클래스와 하위 타입의 차이가 중요해진다.&#x20;

제네릭 타입을 인스턴스화 할 때 타입 인자로 서로 다른 타입이 들어가 인스턴스 타입 사이의 하위 타입 관계가 성립하지 않으면 그 제네릭 타입을 무공변이라고 한다. 앞서 살펴본 MutableList를 생각해보면 각 타입 인자가 다르기만 하면 하위 관계 상관 없이 항상 하위 타입이 성립되지 않는다. 자바에서는 모든 클래스가 무공변이다.&#x20;

반면 List 읽기 전용 컬렉션 인터페이스에서 A가 B의 하위 타입이면 List\<A>는 List\<B>의 하위 타입이다.  이런 클래스나 인터페이스를 공변적이라고 한다.&#x20;





## 공변성: 하위 타입 관계를 유지&#x20;

Producer\<T>를 예로 공변성 클래스를 설명해보자. A가 B의 하위 타입일 때 Producer\<A>가 Product\<B>의 하위 타입이면 Producer는 공변적이다. 이를 하위 타입 관계가 유지된다고 말한다.&#x20;

코틀린에서는 제네릭 클래스가 타입 파라미터에 대해 공변적임을 표시하려면 타입 파라미터 이름 앞에 out을 넣어야 한다.&#x20;

```kotlin
interface Producer<out T>{ // 클래스가 T에 대해 공변적이라고 선언한다.
    fun produce(): T
}
```

클래스의 타입 파라미터를 공변적으로 만들면 함수 정의에 사용한 파라미터 타입과 타입 인자의 타입이 정확히 일치하지 않더라도, 그 클래스의 인스턴스를 함수 인자나 반환 값으로 사용할 수 있다.

먼저 무공변 클래스 예제를 보고 이것을 공변적으로 바꿔보자.&#x20;

```kotlin
// 무공변 클래스 정의 
open class Animal{
    open fun feed() { println("Feeding...") }
}

class Herd<T : Animal>{ // 이 타입 파라미터를 무공변성으로 지정한다.
    val size: Int get() = 15
    operator fun get(i: Int): T { ... } // operator get 정의로 [] 인덱스 참조 가능
}

fun feedAll(animals: Herd<Animal>) {
    for (i in 0 until animals.size) {
        animals[i].feed() // 
    }
}

// 무공변 클래스 사용
class Cat: Animal() {
    override fun feed() { println("Cat is eating...") }
}

fun takeCareOfCats(cats: Herd<Cat>) {
    for (i in 0 until cats.size) {
        cats[i].feed()
    }
    feedAll(cats) // 오류 발생
}

```

이 경우 Herd 클래스의 T 타입 파라미터에 대해 아무 변성도 지정하지 않았기 때문에 고양이 무리는 동물 무리의 하위 클래스가 아니다. 명시적으로 타입 캐스팅을 하면 가능하겠지만 코드가 장황해지고 실수 하기 쉽다. 애초에 올바른 방법도 아니기도 하다.&#x20;

Herd 클래스는 무리안의 동물을 다른 동물로 바꾸지 않는다(변경 x). 따라서 Herd를 공변적인 클래스로 만들고 호출 코드를 적절히 바꿀 수 있다.&#x20;

```kotlin
class Herd<out T : Animal>{ // T는 이제 공변적이다.
 ~~ 
}

fun takeCareOfCats(cats: Herd<Cat>) {
    for (i in 0 until cats.size) {
        cats[i].feed()
    }
    feedAll(cats) // 캐스팅 할 필요 없음
}

```

클래스 멤버를 선언할 때 타입 파라미터를 사용할 수 있는 지점은 모두 in과 out 위치로 나뉜다.&#x20;

T라는 타입 파라미터를 선언하고 T를 사용하는 함수가 멤버로 있는 클래스를 생각해보자.&#x20;

T가 함수의 반환 타입에 쓰인다면 T는 **out 위치**에 있다.그 함수는 T 타입의 값을 **생산한다.**&#x20;

T가 함수의 파라미터 타입에 쓰인다면 T는 **in** 위치에 있다. 그런 함수는 T 타입의 값을 **소비한다.**&#x20;

클래스 타입 파라미터 T 앞에 out 키워드를 붙이면 out 위치에서 밖에 사용이 불가능하다. out 키워드는 T의 사용법을 제한해 T로 인해 생기는 하위 타입 관계의 타입 안정성을 보장한다.&#x20;

Herd 클래스를 다시 보자. Herd에서 T를 사용하는 장소는 **오직 get 메서드의 반환 타입 뿐이다.**&#x20;

이 위치는 아웃 위치이다. 따라서 이 클래스를 공변적으로 선언해도 안전하다. Cat이 Animal의 하위 타입이기 때문에 Hedr\<Animal>의 get을 호출하는 모든 코드에 Cat을 반환해도 문제 없이 작동하는 것이다.&#x20;

정리하자면 T에 붙은 out은 두가지를 함께 의미한다.&#x20;

* 공변성 : 하위 타입 관계가 유지 된다.&#x20;
* 사용 제한: T를 아웃 위치에서만 사용할 수 있다.&#x20;

이제 List\<T> 인터페이스를 보자. 코틀린 List는 읽기 전용이다. 따라서 그 안에는 변경하는 메서드가 없다. 따라서 List는 T에 대해 공변적이다. 하위 타입을 반환해도 문제 없이 작동한다는 것이다. 코드를 보자&#x20;

```kotlin
interface List<out T>: Collection<T>{
    operator fun get(index: Int): T // T를 반환하는 메서드만 정의한다
}
```

반면 MutableList는 공변적인 클래스로 선언할수 없다. T를 인자로 받기도 하고(in) 값을 반환 하기도 하기 때문이다. (out)&#x20;

**변성은** 코드에서 위험할 여지가 있는 메서드를 호출할 수 없게 만듦으로써 제네렉 타입의 인스턴스 역활을 하는 클래스 인스턴스를 잘못 사용하는 일이 없게 방지하는 역할을 한다. 생성자는 한번만 호출 되니 위험할 여지가 없다.&#x20;

하지만 val이나 var 키워드를 생성자 파라미터에 적는다면 게터나 세터를 정의하는 것과 같다. 따라서 읽기 전용 프로퍼티는 out 위치, 변경 가능 프로퍼티는 아웃과 인 위치 모두에 해당한다.&#x20;



## 반공변성: 뒤집힌 하위 타입 관계

반공변성은 공변성의 반대이다. Comparator 인터페이스를 봐보자.

```kotlin
interface Compartor<in T>{
    fun compare(e1: T, e2: T): Int {} // T를 in 위치에서 사용 
}
```

compare 메서드는 T 타입의 값을 소비하기만 한다. T가 in 위치에서만 쓰인다는 뜻이다.&#x20;

어떤 타입에 대해 Comparator를 구현하기만 하면 그 타입의 하위 타입에 속하는 모든 값을 비교할 수 있다.&#x20;

예를 들어 `strings.sortedWith()` 에서 sortedWith 함수는 Comparator\<String>을  요구한다. 이때 String 보다 더 상위 타입으로 비교하는 anyComparator를 넘겨도 가능하다.&#x20;

```kotlin
val anyComparator = Comparator<Any>{
    e1,e2 -> e1.hashCode() - e2.hashCode()
}
```

이는 Comparator\<Any>가 Comparator\<String>의 하위 타입이라는 뜻인데, Any는 String의 상위 타입이다.&#x20;

따라서 서로 다른 타입 인자에 대해 Comparator의 하위 타입 관계는 타입 인자의 하위 타입 관계와 반대이다.&#x20;

Cosumer\<T>로 정리해보면 타입 B가 타입 A의 하위 타입인 경우 Consumer\<A>가 Consumer\<B>의 하위 타입인 관계가 성립되면 제네릭 클래스 Consumer\<T>는 타입 인자 T에 대해 반공변이다.&#x20;

<img src="../../../../.gitbook/assets/file.excalidraw (28).svg" alt="" class="gitbook-drawing">



in이라는 키워드는 그 키워드가 붙은 타입이 이 클래스의 메서드 안으로 전달돼 메서드에 의해 소비된다는 뜻이다.&#x20;

다음 표는 변성에 대해 요약해 보여준다.&#x20;

| 공변성                               | 반공변성                            | 무공변성                 |
| --------------------------------- | ------------------------------- | -------------------- |
| Producer\<out T>                  | Consumer\<in T>                 | MutableList\<T>      |
| 타입 인자의 하위 타입 관계가 제네릭 타입에도서도 유지된다. | 타입 인자의 하위 타입 관계가 제네릭 타입에서 뒤집힌다. | 하위 타입 관계까 성립하지 않는다.  |
| T를 아웃 위치에서만 사용 가능하다.              | T를 인 위치에서만 사용 가능하다              | T를 아무 위치에서나 사용 가능하다. |



파라미터가 하나뿐인 Fuction1 인터페이스처럼 클래스나 인터페이스가 어떤 타입 파라미터에는 공변적이면서 다른 파라미터는 반공변적일 수 있다.&#x20;

```kotlin
interface Function1<in P, out R>{
    operator fun invoke(p: P): R
}
```

동물을 인자로 받아서 정수를 반환하는 람다를 고양이에게 번호를 붙이는 고차 함수에 넘길 수 있다.&#x20;

```kotlin
fun enumerateCats(f: (Cat) -> Number){}
fun Animal.getIndex(): Int = ... 
enumerateCats(Animal::getIndex) 
```

Animal은 Cat의 상위타입이고 Int는 Number의 하위 타입이기 때문에 위의 in out 정의에 맞다.&#x20;

지금까지 살펴본 예제를 보면 클래스 정의에 변성을 직접 기술하면 그 클래스를 사용하는 모든 장소에 그 변성이 작용된다는 점을 알게 되었다. 자바는 이를 지원하지 않고 사용하는 위치에서 와일드카드를 사용해 그때그때 변성을 지정해야한다.&#x20;



## 사용 지점 변성: 타입이 언급되는 지점에서 변성 지정&#x20;

이제까지 클래스를 선언하면서 변성을 지정하면 그 클래스를 사용하는 모든 장소에 지정자가 적용되기 때문에 편하다. 이런 방식을 **선언 지점 변성**이라고 부른다.&#x20;

반면, 자바는 타입 파라미터가 있는 타입을 사용할 때마다 해당 타입 파라미터를 하위 타입이나 상위 타입 중 어떤 타입으로 대치할 수 있는지 명시해야 한다. 이런 방식을 **사용 지점 변성**이라 한다.

{% hint style="info" %}
자바는 사용 지점 변성이기 때문에 사용하는 곳에서 항상 와일드 카드를 사용해야 한다.&#x20;

위의 Function1 인터페이스를 자바에서는 아래 처럼 작성 해야한다.&#x20;

Function\<? super T, ? extedns R>
{% endhint %}

코틀린도 사용 지점 변성을 지원한다. 따라서 특정 타입 파라미터가 나타나는 지점에도 변성을 정할 수 있다.&#x20;

MutableList와 같은 상당수의 인터페이스는 타입 파라미터로 지정된 타입을 소비하는 동시에 생산할 수 있기 때문에 일반적으로 공변적이지도 반공변적이지도 않다. 하지만 그런 인터페이스 타입의 변수가 한 함수 안에서 생산자나 소비자 중 단 한가지 역할만을 담당하는 경우가 자주 있다. 데이터 복사 함수 예를 살펴보자.

```kotlin
fun <T> copyData(source: MutableList<T>, destination<T>){
    for (item in source){
        destination.add(item)
    }
}
```

둘 다 무공변 타입이지만 원본은 읽기만 하고 대상은 쓰기만 한다. 따라서 굳이 같은 타입일 필요 없고 아래 코드 처럼 source 원소타입은 대상 타입의 하위타입으로 지정할 수 있다.&#x20;

```kotlin
fun <T: R, R> copyData(source: MutableList<T>, destination<R>){
    for (item in source){
        destination.add(item)
    }
}
```

하지만 코틀린에서는 이를 더 우아하게 표현할 수 있는 방법이 있다. 함수 구현이 아웃 위치에 있는 타입 파라미터를 사용하는 메서드만 호출한다면 그런 정보를 바탕으로 함수 정의 시 타입 파라미터에 변성 변경자를 추가할 수 있다.&#x20;

```kotlin
fun <T> copyData(source: MutableList<out T>, destination<T>){
    for (item in source){
        destination.add(item)
    }
}
```

타입 선언에서 타입 파라미터를 사용하는 위치라면 어디에나 변성 변경자를 붙일 수 있다. 이때 **타입 프로젝션**이 일어난다. 즉 source를 일반적인 MutableList가 아니랄 MutableList를 프로젝션(제약)을 가한 타입으로 만든다.&#x20;

이 경우 source는 T를 아웃 위치에서만 사용하는 메서드를 호출 할 수 있으며 in 위치는 사용하지 못하게 만든다.&#x20;

비슷한 방식으로 이번엔 in으로 타입 프로젝션을 할 수 있다. destination의 타입은 T의 상위 타입으로 대치된다.

```kotlin
fun <T> copyData(source: MutableList<T>, destination<in T>){
    for (item in source){
        destination.add(item)
    }
}
```

{% hint style="info" %}
사용 지점 변성 선언은 자바의 와일드카드와 같다.&#x20;

* \<out T>는 \<? extends T>와 같다.&#x20;
* \<in T>는 \<? super T>와 같다.
{% endhint %}



## 스타 프로젝션 : 타입 인자 대신 \* 사용&#x20;

앞서 제네릭 타입 인자 정보가 없음을 표현하기 위해 스타 프로젝션을 사용했다. 예를 들어 원소 타입이 알려지지 않은 리스트는 List<\*>로 표현할 수 있다.  스타 프로젝션의 의미를 자세히 살펴보자&#x20;

MutableList<\*>는 MutableList\<Any?>와 같지 않다. Any?는 모든 타입의 원소를 담을 수 있다는 사실을 알 수 있다. 반면 <\*>는 어떤 정해진 구체적인 타입의 원소를 담는 리스트지만 그 원소의 타입을 정확히 모른다는 사실을 표현한다. 타입을 정확히 모른다고해서 아무 원소나 넣을 수 있다는 것은 아니다. 하지만 Any?  하위 타입으로 원소를 얻을 수 있는 것은 확실하다. Any?는 모든 타입의 상위 타입이기 때문이다.&#x20;

따라서 MutableList<\*>는 MutableList\<out Any?>처럼 동작한다. 어떤 리스트의 원소 타입을 모르더라도 안전하게 Any? 타입의 원소를 꺼내올 수 있지만, 타입을 모르는 리스트에 원소를 마음대로 넣을 순 없다.&#x20;

자바의 wildcard ? 처럼 타입 파라미터를 시그니처에서 언급하지 않거나 관심이 없을 때도 스타 프로젝션 구문을 사용할 수 있다.&#x20;

스타 프로젝션을 사용할 때는 값을 만들어내는 메서드만 호출할 수 있고 그 값의 타입에는 신경을 쓰지 말아야 한다.&#x20;





