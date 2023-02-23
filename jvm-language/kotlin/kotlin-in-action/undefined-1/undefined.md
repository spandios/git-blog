---
description: 자바에서 가장 많이 볼 수 있는 오류 중 하나는 NPE 오류이다.
---

# 널 가능성

널 가능성(nullability)은 NPE를 피할 수 있게 돕는 코틀린 타입 시스템의 특성이다.

코틀린을 비롯한 최신 언어에서 NPE를 보완하는 접근 방법은 오류가 나더라도 가능한 컴파일 단계에서 나도록 한다. null이 될 수 있는지 여부를 타입 시스템에 추가함으로써 런타임에서 발견되기 전에 컴파일 단계에서 미리 오류를 감지하는 것이다.&#x20;



## 널이 될 수 있는 타입

코틀린과 자바의 가장 중요한 차이는 코틀린 널이 될 수 있는 타입을 명시적으로 지원한다는 점이다.&#x20;

인자에 널이 올 수 없다는 것을 코틀린에서는 다음 함수와 같이 정의할 수 있다.

```kotlin
fun strLen(s:String) = s.length
```

null이거나 널이 될 수 있는 것을 인자로 넘길 수 없다. 만약 그런 값을 넘긴다면 컴파일 때부터 오류가 발생한다.&#x20;

String 타입의 s 인자는 절대 null이 오지 못하기 때문에 실행 시점에 NPE 에러가 날 일이 없다.&#x20;



이제 널이 될 수 있는 타입으로 정의하고자 한다면 타입 이름 뒤에 ?를 명시한다.&#x20;

```kotlin
fun strLen(s:String?) = s.length
```

널이 될 수 있는 타입의 변수가 있다면 수행할 수 있는 연산이 제한된다. 예를 들어 변수.메서드() 처럼 바로 메서드를 호출할 수 없다.&#x20;

또 널 가능성 타입이 다르면 변수 대입과 인자에 넘기지 못한다.

```kotlin
val x: String? = null

val y: String = x // 오류 서로 nullable 타입이 다른 변수에 대입하려 한다.
strLen(y) // 오류서로 nullable 타입이 다른데 인자로 넘기려 한다. 
```

이렇게 제한이 있는 널이 될 수 있는 타입은 어떤 용도로 사용할까? 가장 중요한 일은 null과 비교하는 것이다.&#x20;

일단 null이 아님을 체크하면 그 다음부터는 널이 될 수 없는 타입처럼 사용할 수 있다.



자바는 변수에 선언된 타입이 있어도 널 여부를 추가로 하지 않으면 그 변수에 어떤 연산을 수행할 수 있을지 알 수 없다. 만약 널이 오지 않을거라 생각하고 널 체크를 안하다가 널이 들어오게 된다면 NPE이 발생시킬 것이다.&#x20;

코틀린은 이런 문제에 대해 종합적인 해법을 제공한다.&#x20;



## 안전한 호출 연산자 ?.

코틀린이 제공하는 가장 유용한 도구 중 하나가 안전한 호출 연산자인 ?.이다.&#x20;

?.은 null 검사와 메서드 호출을 한번의 연산으로 수행한다. 예를 들어 `s?.toUpperCase()`는  `if(s != null) s.toUppseCase() else null와 같다.`

호출 하려는 값이 null이 아니라면 일반 메서드처럼 호출되며 null이라면 null 값이 반환 된다. 안전한 호출의 결과도 널이 될 수 있다는 것에 유의하자.&#x20;



### 프로퍼티&#x20;

안전한 호출 연산자는 메서드뿐 아니라 프로퍼티를 읽거나 쓸 때도 사용 가능하다.&#x20;

```kotlin
class Employee(val name: String, val manager: Employee?)
fun managerName(employee : Employee?) : String? = employee.manager?.name
```



### 안전한 호출 연쇄

안전한 호출을 연쇄해서 간결하게 널 검사를 할 수 있다.&#x20;

```kotlin
class Address(val streetAddress: String, val zipCode: Int, val city: String, val country: String)

class Company(val name: String, val address: Address?)

class Person(val name: String, val company: Company?)

fun Person.countryName(): String {
    val country = this.company?.address?.country // 연쇄
    return if (country != null) country else "Unknown"
}

```



## 엘비스 연산자 ?:&#x20;

코틀린은 null 대신 사용할 디폴트 값을 지정할 때 편하게 사용할 수 있는 엘비스 연산자를 제공한다.

엘비스 연산자를 이용해 위의 countryName 확장 함수를 더 간결하게 수정할 수 있다.

```kotlin
fun Person.countryName(): String {
    return this.company?.address?.country ?: "Unknown"
}
```

코틀린에서는 return이나 throw 등의 연산도 식이다. 따라서 엘비스 연산자의 우항에 return, throw 등의 연산을 넣을 수 있다.  엘비스 연산자의 좌항이 null이면 return 하여 어떤 값을 즉시 반환하거나 throw를 호출해 예외를 던질 수 있는 것이다.

```kotlin
fun test2(str: String?) : Int{
    val r = str ?: throw IllegalArgumentException("str null") // throw를 호출함
    return r.length
}
```



## 안전한 캐스트 as?

as는 코틀린 타입 캐스트 연산자이다. 자바 타입 캐스트와 마찬가지로 대상 값을 지정한 타입으로 바꿀 수 없다면 ClassCastException이 발생한다. 코틀린은 더 안전한 방법인 as? 연산자를 제공한다. as? 연산자는 지정한 타입으로  변환할 수 없다면 오류를 발생 시키지 않고 null을 반환할 것이다.&#x20;

안전한 캐스트를 사용할 때 일반적인 패턴은 캐스트를 수행한 뒤에 엘비스 연산자를 사용하는 것이다.

```kotlin
class Person(val firstName:String, val lastName:String){
    override fun equals(other: Any?): Boolean {
        val otherPerson = other as? Person ?: return false // 캐스팅 실패하면 false 반환
        return otherPerson.firstName == firstName && otherPerson.lastName == lastName
    }
}
```



## 널 아님 단언 (not null assertion)

코틀린에서 널이 될 수 있는 타입의 값을 다룰 때 사용할 수 있는 도구 중 가장 무딘 도구다. !!를 사용하면 어떤 값이든 널이 될 수 없는 타입으로 강제로 바꾼다. !!는 컴파일러에게 값이 null이 아님을 잘 알고 있으니 예외가 발생해도 감수할 것을 표한다.&#x20;

널 아님 단언문이 적절한 경우는 다음과 같다. 어떤 함수가 값이 널인지 검사한 다음에 다른 함수를 호출한다고 해도, 컴파일러는 그 사실을 알지 못해 null 처리를 해줘야한다. 호출된 함수가 다른 함수에서 널이 아닌 값을 전달 받아 실행된다는 사실이 분명하다면 굳이 널 검사를 수행하고 싶지 않을 것이다. 이 때 널 아님 단언문을 사용하면 좋다.&#x20;

!!에서 NPE가 발생하면 스택 트레이스에는 어떤 값이 널이어서 오류가 났는지 알 수 없다. 따라서 !!단언문을 한 줄에 연쇄하여 사용하지말자.



## let 함수&#x20;

let함수는 자신의 수신 객체를 람다에게 넘긴다. 널이 아니라면 널이 아닌 타입의 값을 람다로 전달한다. 만약 널이라면 아무 행동도 하지 않을 것이다.&#x20;

만약 `sendEmailTo`함수는 널이 될 수 없는 String 타입 email을 받는다고 하자.

만약 let을 사용하지 않는다면 `if(email != null) sendEmailTo(email)` 처럼  email에 널이 아니라고 체크 후에 함수를 호출 해야한다. 체크하지 않는다면 컴파일 조차 되지 않을 것이다. 이제 let을 사용해보자.

```
email?.let {sendEmailTo(it)} 
```

훨씬 더 깔끔해졌다. 특히 긴 식이 있고 그 값이 널이 아닐 때 수행해야 하는 로직이 있을 때 let을 쓰면 더 편하다. 긴 식의 결과를 따로 변수에 저장할 필요가 없기 때문이다.

```kotlin
// 이런 코드를 
val person: Person? = getTheBestPersonInTheWorld()
if(person != null) sendEmailTo(person.email)

// 한 줄로 리팩토링 가능하다.
getTheBestPersonInTheWorld()?.let{sendEmailTo(it)}
```

{% hint style="info" %}
let을 중첩시켜 널 체크하는 것보다는 if를 사용해 가독성을 챙기자
{% endhint %}



## 나중에 초기화할 프로퍼티&#x20;

객체 인스턴스를 생성한 다음에 나중에 초기화하는 프레임워크가 많다. 하지만 코틀린은 널이 될 수 없는 프로퍼티를 생성자 안에서 초기화 하지 않고 다른 경로에서는 초기화할 수 없다. 일반적으로 생성자에서 모든 프로퍼티를 초기화해야하기 때문이다.&#x20;

게다가 프로퍼티 타입이 널이 될 수 없는 타입이라면 반드시 그 프로퍼티를 초기화 해야한다. 초기화 값이 애매한다면 nullable로 초기화해야 한다. 그런데 이렇게 되면 프로퍼티 접근에 항상 널 체크를 하거나 !! 연산자를 사용해야 한다.&#x20;

이 때 lateinit 변경자를 붙여 나중에 초기화하는 프로퍼티란 것을 명시할 수 있다.&#x20;

```kotlin
class MyTest{
    private lateinit var myService: MyService
    
    @before fun setUp(){
        myService = MyService()
    }
}
```

lateinit 프로퍼티는 항상 var 여야한다. val는 final이기 때문에 반드시 생성자 안에서 초기화 해야되기 때문이다.&#x20;

## 널이 될 수 있는 타입 확장&#x20;

널이 될 수 있는 타입에 대한 확장 함수를 정의하면 널 값을 다루는 강력한 도구로 활용할 수 있다.&#x20;

String? 타입의 수신 객체에 대해 호출할 수 있는 isNullOrEmpty를 살펴보자.

input 파라미터가 nullable임에도 안전한 호출을 하지 않고 함수를 호출했다. nullable 타입에 대해 확장을 정의한다면 안전한 호출 없이도 호출 가능하다.

```kotlin
fun String?.isNullOrBlank(): Bollean = this == null || this.isBlank() 

fun verifyUserInput(input: String?){
    if (input.isNullOrBlank()){ // 안전한 호출이 없음
        println("Please fill in the required fields")
    }
}
```

앞서 let도 nullable에 대해 안전한 호출 없이 호출 할 수 있지만, 널인지 검사하지는 않는다. 따라서 만약 nullable 프로퍼티에  안전한 호출 ?. 없이 let을 호출 한다면 람다의 인자는 널이 될 수 있는 타입으로 추론이 될 것이다.&#x20;



## 타입 파라미터의 널 가능성

타입 파라미터 T를 클래스나 함수 안에서 사용할 때 이름 끝에 물음표가 없더라도 nullable 타입이다.&#x20;

만약 널이 될 수 없는 것을 확실히 하기 위해서는 타입 상한을 지정해야 한다.&#x20;

```kotlin
fun <T> printHashCode(t: T){ 
    print(t?.hashCode())  //T의 타입은 Any?로 추론된다.
}
fun <T: Any> printHashCode(t: T){
    print(t.hashCode()) // 널이 될 수 없다. 
}
```



## 널 가능성과 자바&#x20;

1. 자바의 애노테이션으로 표시된 널 가능성&#x20;



