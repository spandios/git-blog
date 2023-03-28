---
description: 프로퍼티 접근자 로직 재활용
---

# 위임 프로퍼티

* 위임 프로퍼티는 코틀린이 제공하는 관례에 의존하는 특성 중에 독특하면서 강력한 기능이다.
* 위임 프로퍼티를 사용하면 **값을 뒷받침 하는 필드에 단순히 저장하는 것보다 더 복잡한 방식으로 동작하는** 프로퍼티를 쉽게 구현할 수 있다.&#x20;
* 또한 그 과정에서 접근자 로직을 매번 재구현할 필요도 없다.  예를 들어 프로퍼티는 위임을 사용해 자신의 값을 필드가 아니라 데이터베이스 **테이블이나 브라우저 세션, 맵 등에 저장** 할 수 있다.&#x20;
* 이런 특성 기반에는 위임이 있다. 위임은 객체가 직접 작업을 수행하지 않고 다른 도우미 객체가 그 작업을 처리하게 맡기는 디자인 패턴을 말한다. 이 때 작업을 처리하는 도우미 객체를 위임 객체라고 부른다.&#x20;



## 위임 프로퍼티 소개&#x20;

위임 프로퍼티의 일반적인 문법은 다음과 같다.

```kotlin
class Foo{
    var p: Type by Delegate()
}
```

* p 프로퍼티는 접근자 로직을 다른 객체에게 위임한다. 이 예제에서 위임 객체는 Delegate 클래스의 인스턴스이다.
* 프로퍼티 위임 객체가 지켜야하는 관례를 지킨 모든 객체는 위임에 사용할 수 있다.
* 프로퍼티 위임 관례를 따르는 Delegate 클래스는 **getValue와 setValue 메서드**를 제공해야 한다. 컴파일러가 접근자 메서드에서 위임 객체의 setValue와 getValue를 호출하기 때문이다.&#x20;

```kotlin
class Foo{
    private val delegate = Delegate()
    var p : Type
    set(value: Type) = delegate.setValue(~)
    get() = deletegate.getValue(~)
}

class Delegate{
    operator fun getValue(~){~}
    operator fun setValue(~){~}
}
```



## by lazy()를 사용한 프로퍼티 초기화 지연&#x20;

* 위임 프로퍼티 기능의 강력한 예로 프로퍼티 초기화를 지연 시켜주는 lazy로 둘 수있다.
* 지연 초기화는 객체의 일부분을 초기화 하지 않고 남겨뒀다가 실제로 그 부분의 값이 필요할 경우 초기화할 때 흔히 쓰이는 패턴이다. 초기화 과정에서 자원을 많이 사용하거나 객체를 사용할 때 꼭 초기화하지 않아도 되는 프로퍼티에 대해 지연 초기화 패턴을 사용하면 좋다.&#x20;

먼저 뒷받침하는 프로퍼티를 통해 지연 초기화를 구현해보자.

```kotlin
fun loadEmails(person:Person) : List<Email>{
    return listOf(~)    
}

class Person(val name: String){
    private val _emails: List<Email>? = null // 뒷받침 하는 프로퍼티
    val emails: List<Email>
        get(){
            if(_email == null){
                _emails = loadEmails(this)
            }else{
                return _emails!!
            }
        }
}
```

`_emails는` 뒷받침 하는 프로퍼티로 값을 저장하는데 사용된다. 다른 프로퍼티인 emails는 실제로 프로퍼티에 대한 읽기 연산을 클라이언트에게 제공한다. emails 프로퍼티에 직접 조회할 때서야 뒷받침하는 프로퍼티에 값이 없으면 초기화하는 지연 초기화 패턴을 구현했다. 하지만 이 구현은 번거럽기도 하고 스레드 안전하지 않기도 하다.&#x20;

이 때 코틀린은 더 나은 해법인 위임 프로퍼티를 제공한다. 아래 코드를 보자

```kotlin
class Person(val name:String){
    val emails by lazy {loadEmails(this)}
}
```

훨씬 간결하다. 위임 프로퍼티는 뒷받침 하는 프로퍼티와 오직 한번만 초기화 되는 게터 로직을 함께 캡슐화 해준다.&#x20;

lazy 함수는 코틀린 관례에 맞는 getValue를 포함한 객체를 반환한다. by lazy 이 두 키워드를 풀어서 해석해보자.&#x20;

lazy 함수는 초기화 로직을 람다로 받으며 위임 객체 관례인 getValue를 시그니처를 만족하는 객체를 반환하고 있다. emails 위임 프로퍼티는 by 키워드를 통해 lazy가 반환한 객체를 위임 객체로 지정하고 있다.&#x20;



```kotlin
// lazy
public actual fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer)

//lazy getValue 확장함수
@kotlin.internal.InlineOnly
public inline operator fun <T> Lazy<T>.getValue(thisRef: Any?, property: KProperty<*>): T = value
```



## 위임 프로퍼티 컴파일 규칙

컴파일러는 위임 객체를 감춰진 프로퍼티에 저장하며 그 감춰진 프로퍼티를 \<delegate>라는 이름으로 부른다. 또한 컴파일러는 프로퍼티를 표현하기 위해 KProperty 타입의 객체를 사용하고 \<property>라고 부른다.&#x20;

컴파일러는 모든 프로퍼티 접근자 안에 getValue와 setValue 호출 코드를 생성해준다.

<img src="../../../../../.gitbook/assets/file.excalidraw (11).svg" alt="" class="gitbook-drawing">

단순하지만 프로퍼티 값이 데이터베이스처럼 저장될 장소를 바꿀 수 있고, 프로퍼티를 읽거나 쓸 때 값 검증이나 변경 탐지 처럼 벌어질 일도 변경할 수도 있다.&#x20;

값을 맵에 저장하는 프로퍼티를 저장하는 사례를 보자. 모든 데이터를 map으로 저장하되 이름만 변수로 따로 빼내어 필수 정보로 관리하고 있다. map 인터페이스는 getValue와 setValue 확장 함수를 이미 제공하고 있어 위임 객체에 map을 그대로 사용하고 있다.&#x20;

```kotlin
class Person{
    val _attributes = hashMapOf<String, String>()
    fun setAttribute(attrName: String, value: String){
        _attributes[attrName] = value
    }
    val name: String by _attributes
}
```





