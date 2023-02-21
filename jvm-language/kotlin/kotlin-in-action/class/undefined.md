# 데이터 클래스

## 컴파일러가 생성한 메서드

자바 플랫폼에서는 equals, hashCode, toString 메서드 처럼 클래스가 반드시 정의해야하는 메서드가 있다.

앞서 코틀린이 생성자와 접근자를 알아서 만들어 준 것처럼 이번에도 이런 메서드들을 보이지 않는 곳에서 미리 만들어서 제공해주기 때문에 편리하고 코드 가독성을 높여준다. 각각 어떤 메서드인지 알아보자.

### equals

자바에서는 ==의 동작이 피연산자에 따라 다르다. 만약 원시타입에서 사용 된다면 동등성 연산으로 값을 비교할 것이고, 참조 타입이라면 참조 비교 연산으로 두 참조의 주소를 비교 할 것이다.

{% hint style="info" %}
동등성 (equality) vs 참조 비교(reference comparision)

동등성 연산은 두 피연산자의 값을 비교하는 것이고, 참조 비교 연산은 두 피연산자의 주소가 같은지를 비교한다.
{% endhint %}

만약 서로 다른 객체이지만 어떤 변수의 값이 같다면 동등하다는 로직을 추가하려면 어떻게해야할까? eqauls 메서드를 오버라이드 하면 된다.&#x20;

```kotlin
class Client(val id: String, val name: String){
    override fun eqauls(ohter: Any?) : Boolean{
        if(ohter == null || other !is Client) false // !is를 통해 해당 타입이 아닌지를 체크했다.
        return id == other.id // id만 같으면 서로 같은 객체라는 로직을 따로 정의했다.
    }
}
```

코틀린에서의 ==는 내부적으로 eqauls를 호출한다. 만약 참조 비교를 하고 싶다면 ===를 사용하면 된다.&#x20;



### hashCode

jvm 플랫폼에서는 eqauls 결과가 true로 반환하는 두 객체는 반드시 서로 같은 hashCode를 반환 해야하는 규칙이 있다. 이 규칙을 준수할 것이라고 기대하는 HashSet은 자신의 원소를 비교할 때 hash를 먼저 비교하고 같을 때만 실제 값을 비교한다. 따라서 eqauls를 재정의 했다면 반드시 hashCode 또한 재정의를 해야한다.

```kotlin
class Client(val id: String, val name: String){
    override fun hashCode(): Int = id.hashCode() + name.hashCode()
}

```

### toString&#x20;

기본으로 제공되는 객체의 문자열 표현은 주소를 뜻하기 때문에 유용하지 않다. toString()을 재정의하면 디버깅이나 로깅 시 도움이 된다.&#x20;

```kotlin
class Client(val id: String, val name: String){
    override fun toString(): String = "Client(id=$id, name=$name)"
}

```

## 데이터 클래스&#x20;

클래스 앞에 data 변경자를 붙인 것을 데이터 클래스라고 하며, 데이터 클래스는 위의 모든 클래스가 정의해야하는 메서드를 자동으로 생성해 준다.

```kotlin
data class Client(val id: String, val name:String)
```

* 자동으로 생성된 **eqauls**: 주 생성자에 나열된 모든 프로퍼티 값의 동등성을 확인한다.&#x20;
* 자동으로 생성된 **hashCode**: 모든 프로퍼티의 해시 값을 바탕으로 계산된 해시 값을 반환한다.
* 자동으로 생성된 **toString**: 클래스의 각 필드 순서대로 표시하는 문자열을 반환한다.&#x20;

코틀린은 데이터 클래스에게 이외에도 몇 가지 유용한 메서드를 더 생성해준다.&#x20;



## 데이터 클래스와 불변성&#x20;

데이터 클래스의 모든 프로퍼티는 가급적 읽기전용인 불변 클래스로 만들기를 권장한다. 불변 객체는 더 간단하고 멀티 쓰레드에서 더 안전하고 성능이 좋다.&#x20;

데이터 클래스 인스턴스를 불변 객체로 유지하면서 일부 프로퍼티를 변경 할 수 있는 copy 메서드란게 있다.

직접 값을 변경하지 않고 복사본을 생성하고 그것의 프로퍼티를 변경함으로써 원본 객체를 변하지 않게 하는 것이다.&#x20;

```kotlin
val heo = CLient("3","허주영)
print(heo.copy(name="알파카")) // Client(id="3", name="알파카")
```



