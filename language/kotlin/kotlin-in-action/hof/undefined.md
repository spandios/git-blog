---
description: 람다의 부가 비용 없애기
---

# 인라인 함수

우리는 이미 보통 람다를 호출한다면 처음에는 새로운 무명 객체를 생성 후 변수에 담아놓고 이후 호출되면 이 변수를 계속 사용하지만 변수를 포획한 람다는 호출될 때마다 무명 클래스 객체를 생성한다는 것을 알아보았다.

그렇다면 반복되는 코드를 별도의 라이브러리 함수로 빼내어 컴파일러가 자바의 일반 명령문만큼 효율적인 코드를 생성하게 만들면 어떨까? 코틀린은 inline 변경자를 어떤 함수에 붙이면 컴파일러는 이 함수를 호출하는 문장을 inline 함수 본문에 해당하는 바이트코드로 바꿔치기 해준다.



## inline이 작동하는 방식

어떤 함수를 inline으로 선언하면 그 함수의 본문이 inline된다. 다시 말해 인라인 함수를 호출하는 코드에 inline 함수 본문을 번역한 바이트 코드가 그대로 대체되고 컴파일 된다는 것이다.

아래 inline 함수는 다중 스레드 환경에서 어떤 공유 자원에 대한 동시 접근을 막기 위한 것이다.&#x20;

```kotlin
// 명시적인 락을 사용해 더 신뢰할 수 있고 관리하기 쉬운 코드
inline fun <T> synchronized(lock: Lock, action: () -> T): T {
    lock.lock()
    try {
        return action()
    } finally {
        lock.unlock()
    }
}
```

이제 위의 인라인 함수를 호출하는 코드가 어떻게 컴파일 되는지 보자

```kotlin
// 컴파일 전 
fun foo(l: Lock){
    println("Before sync")
    synchronized(lock){
        println("Action")
    }
    println("After sync")
}
```

```kotlin
// 컴파일 후 자바코드
  public static final void foo(@NotNull Lock lock) {
      System.out.println("Before sync"); // foo
      lock.lock(); // inline synchronized
      try { 
         // 인라인 함수에 전달된 람다의 본문도 함께 인라닝이 된다.
         System.out.println("Action"); // inline, 람다 본문도 inline 됨
      } finally {
         lock.unlock(); // synchronized
      }
      System.out.println("After sync"); //foo 
   }
```

람다의 본문에 의해 만들어지는 바이트코드는 그 람다를 호출하는 코드 정의의 일부분으로 간주되기 때문에 코틀린 컴파일러는 그 람다를 함수 인터페이스를 구현하는 **무명 클래스로 감싸지 않는다. 따라서 성능 이슈가 발생하지 않는다.**&#x20;

인라인 함수를 호출하면서 람다를 넘기는 대신 함수 타입의 변수도 넘길 수도 있다.&#x20;

```kotlin
class LockOwner(val lock:Lock){
    fun runUnderLock(boy: ()-> Unit){
        synchrnized(lock, body)
    }
}
```

하지만 이런 경우 인라인 함수를 호출하는 코드 위치에서는 **지정된 람다의 코드를 알 수 없다**. 따라서 람다 본문은 인라이닝되지 않고 synchronized 함수의 본문만 인라인 될 것이다. 따라서 일반 람다 방식처럼 호출된다. 이 코드를 컴파일한 바이트코드는 다음 함수와 비슷하다.&#x20;

```kotlin
class LockOwner(val lock:Lock){
    fun __runUnderLock__(body: ()->Unit){
        lock.lock() // synchronized
        try{
            body() // 람다 본문은 인라인 되지 않았다.
        }
        finally{
            lock.unlock() // synchronized
        }
    }
}
```



## 인라인 함수의 한계

* 모든 함수를 인라인 함수로 정의할 순 없고 함수가 파라미터로 전달받은 람다를 본분에 사용하는 방식으로 한정된다.
* 아래 함수는 파라미터로 전달받은 람다를 바로 호출하지 않고 객체 생성자에 넘기고 있다.
* 생성된 객체는 전달 받은 람다를 프로퍼티로 저장한다. 이럴 때는 transform 인자를 인라인 함수를 받을 수 없다. 즉, 인터페이스를 구현하는 무명 클래스 인스턴스로 만들어야하는 것이다. 프로퍼티에 람다 바이트코드를 넣을 순 없기 때문이다.&#x20;

```kotlin
fun <T, R> Sequenece<T>.map(transfrom: (T) -> R): Sequence<R>{
    return TransformingSequence(this, transform) // 람다를 객체 생성자에 넘기고 있다.
}
```

{% hint style="info" %}
noline

둘 이상의 람다를 인자로 받는 함수에서 일부 람다만 인라인 하고 싶지 않다면 noline 변경자를 사용하자.
{% endhint %}

##

## 컬렉션 연산 인라이닝

이번에는 컬렉션에 대해 작용되는 코틀린 표준 라이브러리의 성능을 알아보자. 표준 라이브러리 함수들은 대부분 람다를 인자로 받는다. 그럼 표준 라이브러리 함수와 람다를 사용하지 않고 직접 연산을 구현한다면 더 효율적일까?&#x20;

```kotlin
data class Person(val name:String, val age: Int)
val people = listOf(Person("Alice", 29), Person("Bob", 31))
// filter
people.filter { it.age < 30 }

// 직접 컬렉션 거르기
val result = mutableListOf<Person>()
for (person in people){
    if(person.age<30) result.add(person)
}
```

답은 아니다이다. filter는 인라인 함수이다. 따라서 filter 함수의 바이트코드는 그 함수에 전달된 람다 본문의 바이트코드와 함께 filter를 호출한 위치에 들어간다. 그 결과 filter 함수를 사용한 바이트코드와 직접 컬렉션을 거른 바이트코드는 거의 유사하기 때문에 성능에 대해 신경쓰지 않아도 되는 것이다.

```kotlin
public inline fun <T> Iterable<T>.filter(predicate: (T) -> Boolean): List<T> {
    return filterTo(ArrayList<T>(), predicate)
}
```



컬렉션 연산을 연쇄하는 상황을 생각해보자. 이 때 처리할 원소가 많아지면 중간 리스트를 사용하는 부가 비용이 커지기 때문에 asSequence를 통해 시퀸스를 사용해 중간 리스트를 사용하는 부가 비용을 줄일 수 있다.&#x20;

그런데 앞에서 봤듯이 시퀸스는 람다를 필드에 저장하는 객체로 표현되며, 최종 연산은 중간 시퀸스에 있는 여러 람다를 연쇄 호출한다.  따라서 앞 절에서 설명한 대로 시퀸스는 람다를 저장하기 때문에 인라인을 하지 않는다.&#x20;

따라서 지연 계산을 통해 성능을 얻고자 모든 컬렉션을 시퀸스로 하면 안된다. 람다가 인라이닝이 되지 않아 크기가 작은 컬렉션을 시퀸스를 사용한다면 오히려 성능이 나빠질 것이다.&#x20;



## 함수를 인라인으로 선언해야하는 경우

inline의 성능을 이점을 생각해서 여기저기 함수를 inline으로 정의하면 안된다. inline 키워드를 사용해도 람다를 인자로 받는 함수만 성능이 좋아질 가능성이 높다. 다른 경우에는 주의 깊게 성능을 측정해보고 조사해야한다.&#x20;

일반 함수 호출의 경우 JVM이 이미 강력하게 인라이닝을 지원하고 있다. JVM은 코드 실행을 분석해 가장 이익이 되는 방향으로 호출을 인라이닝한다. 이런 과정은 바이트코드를 실제 기계어 코드로 번역하는 과정인 JIT에서 일어난다. 이런 JVM 최적화를 활용한다면 각 함수 구현이 정확히 한 번만 있으면 되고, 그 함수를 호출하는 부분에서 따로 함수 코드를 중복할 필요가 없다. 하지만 코틀린의 일반 인라인 함수는 바이트코드에서 모든 함수 호출 지점을 함수 본문으로 대치하기 때문에 코드 중복이 생긴다.

반면 람다를 인자로 받는 함수를 인라인 하는 것은 이익이 많다. 일단 JVM이 람다까지 인라이닝은 하지 못해주고 있다. 또, 인라이닝을 사용하면 함수 호출 비용을 줄이고, 람다를 표현하는 클래스나 람다 인스턴스를 만들 필요가 없어지며, 넌로컬 같이 몇 가지 기능들을 더 사용할 수 있다.&#x20;

하지만 inline 변경자를 함수에 붙일 때는 코드 크기에 주의 해야한다. 함수 본문에 해당하는 많은 바이트코드를 모든 호출 지점에 복사하기 때문이다.&#x20;



## 인라인된 람다 use 사용&#x20;

자바7 부터 try with resource문이 생겼다. 이 문은 try 문이 끝나면 자동으로 리소스를 해제해주는 식이다.&#x20;

하지만 위의 식보다 코틀린은 람다를 통해 깔끔하게 처리할 수 있기 때문에 제공하지 않고, 대신 use라는 함수를 제공한다. 다음은 use를 사용한 코드이다. use도 인라인 함수이기 때문에 성능 이슈가 없다.&#x20;

```kotlin
fun readFirstFromFile(path:String): String {
    BufferedReader(FileReader(path)).use { br ->
        return br.readLine()
    }
}
```

람다의 본문 안에서 사용한 return은 넌로컬 return이다. 이 return문은 람다가 아니라 readFirstLineFromFile함수를 끝내면서 값을 반환한다. 자세한 람다에서의 흐름제어는 다음 글에서 살펴보자.
