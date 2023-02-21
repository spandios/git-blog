# 데이터 클래스

## 컴파일러가 생성한 메서드

자바 플랫폼에서는 equals, hashCode, toString 메서드 처럼 클래스가 반드시 정의해야하는 메서드가 있다.

코틀린은 이런 메서드들을 기계적으로 보이지 않는 곳에서 미리 만들어서 제공해주기 때문에 코드 가독성을 높여준다.

각각 어떤 메서드인지 알아보자.

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



### toString



