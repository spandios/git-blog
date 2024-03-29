# 클래스 위임

* 상속을 허용하지 않는 클래스에 새로운 동작을 추가해야할 때 해결하는 일반적인 방법은 데코레이터 패턴이다.&#x20;
* 이 패턴의 핵심은 데코레이터 클래스를 만들고 그 내부 필드에 기존 클래스를 넣되 기존 클래스의 인터페이스를 그대로 구현하게 하는 것이다. 이 때 새로운 기능은 데코레이터 클래스에서 새로 정의하거나 변경이 필요하지 않은 기능은 기존 클래스에게 요청을 전달하면 된다.&#x20;
* 하지만 이 방법의 단점은 코드가 상당히 필요하다는 것이다. 변경이 필요하지 않은 기능이라도 기존 클래스에게 위임하는 코드가 필요하기 때문이다.

```kotlin
class DelegatingCollection<T>: Collection<T>{
    private val innerList = arrayListOf<T>()
    override val size: Int get() = innerList.size
    override fun isEmpty(): Bollean = innerList.isEmpty()
    // ~~ 이외의 구현이 필요한 모든 메서드
}
```

* 이 때 코틀린은 **by**라는 키워드를 통해 그 인터페이스에 대한 구현을 다른 객체에 위임한다는 사실을 명시할 수 있다.&#x20;

```kotlin
// 컴파일러가 전달 메서드를 자동으로 생성해주기 때문에 코드가 매우 깔끔해진 것을 볼 수 있다.
class DelegatingCollection<T>(innerList: Collection<T> = ArrayList<T>()
): Collection<T> by innerList{}
```
