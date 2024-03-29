# 원시 타입

자바는 원시 타입과 참조 타입을 구분한다. 원시 타입은 변수에 그 값이 직접 들어가지만 참조 타입은 메모리상의 객체 위치가 들어간다. 널 참조를 사용 해야하거나 컬렉션에 원시타입을 담을 수 없는 사례처럼 원시 타입을 사용할 수 없을 때 자바는 래퍼 타입으로 원시 값을 담아 사용한다.

반면 코틀린은 원시 타입과 래퍼 타입을 구분하지 않기 때문에 편하며 자동으로 가장 효율적인 방식을 선택해준다. 원시타입으로는 컴파일이 불가능한 경우를 제외하면 자바의 원시타입으로 컴파일해 값을 효율적으로 저장하고 사용한다.



## 널이 될 수 있는 원시 타입&#x20;

* 자바의 원시 타입은 널 참조를 가질 수 없고 참조 타입 변수만 가질 수 있다.&#x20;
* 따라서 코틀린에서 널이 될 수 있는 원시 타입을 사용하면 그 타입은 자바의 래퍼 타입으로 컴파일 된다.&#x20;
* 제네릭 클래스의 경우도 래퍼 타입을 사용한다. `val listOfInts = listOf(1,2,3)`이 코드를 컴파일 하면 리스트는 래퍼 Integer 타입으로 이루어진 리스트이다. &#x20;
* 왜 nullable 타입을 사용하지 않았는데 박싱 타입을 사용한 것일까? 바로 JVM에서 제네릭을 구현하는 방법 때문이다.  JVM은 타입 인자로 원시 타입을 허용하지 않는다. 따라서 제네릭 클래스는 항상 박스 타입을 사용해야 하는 것이다.



## 숫자 변환&#x20;

* 코틀린과 자바의 가장 큰 차이점 중 하나는 숫자를 변환하는 방식이다.&#x20;
* 코틀린은 한 타입의 숫자를 다른 타입의 숫자로 자동 변환해주지 않는다. 대신 명시적으로 직접 변환해줘야 한다. 이 결정은 박스 타입 간의 eqauls가 그 안에 들어 있는 값이 아니라, 박스 타입 객체를 비교하는 등의 혼란을 피하기 위해서다. &#x20;
* 코틀린은 toInt(), toChar()과 같은 모든 원시 타입에 대한 변환 함수를 제공한다.

{% hint style="info" %}
원시 타입 리터럴

* L 접시가가 붙은 Long 타입 리터럴: 123L
* 표준 부동소수점 표기법을 사용한 Double 타입 리터럴: 0.12, 2.0, 1.2e10
* f나 F접미사가 붙은 Float 타입 리터럴 : 123.4f
* 0x나 oX 접두사가 붙은 16진 리터럴: 0xCAFEBABE
* 0b나 0B 접두사가 붙은 2진 리터럴: 0b000000101

숫자 리터럴 중간에 밑줄을 넣을 수 있다. (1\_234)&#x20;

문자 리터럴: 작은따옴표 안에 문자를 넣으면 되며 이스케이프 시퀸스를 사용할 수 있다 : '1' , '\t'
{% endhint %}



## Any: 최상위 타입&#x20;

* 자바에서 Object가 클래스 계층의 최상위 타입이듯 코틀린에서는 Any 타입이 non-nullable 타입의 조상 타입이다.&#x20;
* 자바에서는 참조 타입만 Object가 최상위 타입인 반면 코틀린에서 Any는 원시 타입을 포함한 모든 타입의 조상 타입이다.&#x20;
* Any타입은 널이 될 수 없는 타입이다. 따라서 널을 포함하는 모든 값을 저장할 수 있는 변수를 선언하려면 Any? 타입을 사용해야 한다.&#x20;
* 내부에서 Any타입은 java.lang.Object에 대응한다. 자바에서 Object는 코틀린에서는 Any!로 취급되며, 코틀린 함수가 Any를 사용하면 자바에서는 Object로 컴파일 된다.&#x20;
* 모든 코틀린 클래스는 toString, eqauls, hashCode 함수라는 메서드를 가지고 있다. 이 세 메서드는 Any에 정의된 메서드를 상속한 것이다. 하지만 java.lang.Object에 있는 wait, notify처럼 다른 메서드들은 Any에서 사용할 수 없다. 그런 메서드를 호출하려면 java.lang.Object 타입으로 캐스트해야한다.&#x20;



## Unit 타입: 코틀린의 void

Unit 타입은 자바 void와 같은 기능을 한다.&#x20;

```kotlin
fun f(): Unit {} == fun f(){} // Unit 반환 타입은 생략 가능
```

코틀린은 void와 다르게 일반적인 타입이며 따라서 타입 인자로 사용할 수 있다. Unit 타입에 속한 값은 단 하나이다. Unit타입의 함수는 Unit 값을 묵시적으로 반환할 것이다.&#x20;

```kotlin
interface Processor<T>{
    fun process(): T
}

class NoResultProcessor: Processor<Unit>{
    override fun process(){ // Unit 타입 함수는 묵시적으로 Unit값을 반환 
        ~~ 
    }
}

```



## Nothing : 결코 정상적으로 끝나지 않음

코틀린에서는 성공적으로 값을 돌려주는 일이 없으므로 반환 값 자체가 의미 없는 함수가 일부 존재한다.&#x20;

예를 들어 테스트 라이브러리에서 fail 함수 같이 예외를 던져 실패하는게 정상인 함수같은 것들이 있다.&#x20;

이런 함수를 분석할 때 정상적으로 끝나지 않는다는 것을 알면 유용하다. 그렇기 때문에 코틀린은 Nothing이라는 타입으로 표현한다.&#x20;

