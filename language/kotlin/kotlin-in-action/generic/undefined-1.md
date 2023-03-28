---
description: 소거된 타입 파라미터와 셀체화 된 타입 파라미터
---

# 제네릭스의 동작

## 실행 시점의 제네릭: 타입 검사와 캐스트&#x20;

* JVM의 제네릭스는 보통 타입 소거를 사용해 구현된다. 이는 실행 시점에 제네릭 클래스의 인스턴스에 타입 인자 정보가 들어있지 않다는 뜻이다.&#x20;
* 코틀린도 마찬가지로 제네릭 타입 인자 정보는 런타임에 지워진다. 예를 들어 List\<String> 객체를 만들고 그 안에 문자열을 넣더라고 List가 어떤 타입의 원소를 저장하는지 실행 시점에는 알 수 없다.&#x20;

<img src="../../../../.gitbook/assets/file.excalidraw (19).svg" alt="" class="gitbook-drawing">

* 컴파일러는 두 리스트를 서로 다른 타입으로 인식하지만 실행 시점에는 완전히 같은 타입의 객체이다.&#x20;
* 컴파일러가 타입 인자를 알고 올바른 타입의 값만 각 리스트에 넣도록 보장해주기 때문에 각 타입에 맞는 원소가 들어갈 것이라고 가정할 수 있다.&#x20;
* 저장해야하는 타입의 정보의 크기가 줄어들어 전반적인 메모리 사용량이 줄어드는 나름의 장점이 있다.&#x20;



### 타입 인자 한계

타입 인자를 따로 저장하지 않기 때문에 실행 시점에서 타입 인자를 검사할 수 없다. 예를 들어 어떤 리스트가 문자열로 이뤄진 리스트인지 다른 객체로 이뤄진 리스트인지를 실행 시점에 검사할 수 없다. (is 검사에서 타입 인자로 지정한 타입 검사 불가능)

```kotlin
if (value is List<String>) {} // ERROR: Cannot check for instance of earsed type
```

List인 것은 알지만 요소가 String인지 Int인지에 대한 것은 타입 인자가 실행 시점에 없어지기 때문에 모른다.



### 스타 프로젝션 (star projection)

코틀린에서는 타입 인자를 명시하지 않고는 제네릭 타입을 사용할 수 없다. 위에서 다른 컬렉션이 아니라 리스트라는 사실만을 확인하고 싶다면 스타 프로젝션을 사용하면 된다.&#x20;

```kotlin
if( value is List<*>){}
```

타입 파라미터가 2개 이상이면 모든 타입 파라미터에 \*를 붙여야한다. 자바의 List\<?>와 비슷하다.&#x20;



### 제네릭 타입으로 타입 캐스팅하기&#x20;

as로 제네릭 타입을 캐스팅 할 수 있다. 하지만 실행 시점에는 제네릭 타입의 타입 인자를 알 수 없기 때문에 타입 인자가 다른 타입으로 캐스팅해도 문제 없이 성공한다는 점을 조심해야 한다. 다음 예를 보자&#x20;

```kotlin
fun printSum(c: Collection<*>){ // 컴파일러가 Unchcked cast 경고를 발생해주긴 한다. 
    val intList = c as? List<Int> ?: throw IllegalArgumentException("List is expected")
}
```

printSum 함수 파라미터에 set 컬렉션을 넘겨보자 as 캐스팅에 실패하기 때문에 IllegalArgumentException이 발생한다. 하지만 잘못된 타입 원소인 String 리스트를 전달하면 as 캐스팅은 성공하지만 나중에 ClassCastException이 발생한다. 컴파일러는 겉에 컬렉션은 어떤 타입인지는 알지만 제네릭 타입 인자는 알지 못해 경고만 보내주고 실제 런타임에서 오류가 나는 것 까지 막아주지 못한다.&#x20;



## 실체화한 타입 파라미터를 사용한 함수 선언&#x20;

앞에서 말했듯이 코틀린 제네릭 타입의 타입 인자 정보는 실행 시점에 지워진다. 이것은 제네릭 함수의 타입 인자도 마찬가지이다. 제네릭 함수가 호출되도 그 함수의 본문에서는 호출 시 쓰인 타입 인자를 알 수 없다.&#x20;

```kotlin
fun <T> isA(value: Any) = value is T // Error cannot check for instance of erased type : T
```

하지만 이런 제약을 피할 수 있는 경우가 하나 있는데, 바로 **인라인 함수의 타입 파라미터는 실체화**가(함수 본문을 구현한 바이트코드가 호출되는 지점에 삽입 되기 때문에 정확한 타입 인자를 이미 안다) 되기 때문에 실행 시점에 인라인 함수의 타입 인자를 알 수 있다.  방금 만든 isA 메서드를 인라인 함수로 만들고 타입 파라미터를 reified(구체화)로 지정하면 value의 타입이 T의 인스턴스인지를 시행 시점에도 검사할 수 있다.&#x20;

```kotlin
inline fun <reified T> isA(value:Any) = value is T 
isA<String>("abc") => true
isA<String>(123) => false 

```

코틀린 표준 라이브러리 함수인 filterIsInstance에서 실체화한 타입 파라미터를 사용한 예를 살펴보자. 이 함수는 인자로 받은 컬렉션의 원소 중에서 타입 인자로 지정한 클래스의 인스턴스만을 모아준다. 인라인 함수이기 때문에 타입 인자를 실행 시점에 알 수 있어 타입이 일치하는 원소만을 추려낼 수 있다.&#x20;

```kotlin
inline fun <refied T> Iterable<*>.filterIsInstance() : List<T>{
    val destination = mutableListOf<T>()
    for (element in this){
        if(element is T){
            destination.add(element)
        }
    }
}
```

위의 함수는 람다를 받지 않은데도 inline으로 구현했다. 저번엔 inline은 함수의 파라미터 중에 함수형 타입인 파라미터가 있어 그 파라미터에 해당하는 람다를 함께 인라이닝하면서 얻는 이익이 컸을 경우만 inline 함수를 만들라고 했다.&#x20;

하지만 이 경우는 성능 향상이 목적이 아니라 실체화한 타입 파라미터를 위함이다.&#x20;



## 실체화한 타입 파라미터로 클래스 참조 대신&#x20;

java.lang.Class 타입 인자를 파라미터로 받는 API에 대한 코틀린 어댑터를 구축하는 경우 실체화한 타입 파라미터를 자주 사용한다. java.lang.Class를 사용한 API의 예로는 JDK의 ServiceLoader가 있다. ServiceLoader는 어떤 추상 클래스나 인터페이스를 표현하는 java.lang.Class를 받아서 그 클래스나 인스턴스를 구현한 인스턴스를 반환한다.&#x20;

원래 표준 자바 API인 ServiceLoader를 사용해 서비스를 읽어 들이려면 다음 코드처럼 호출해야 한다.

`val serviceImpl = ServiceLoader.load(Service::class.java)` ::class.java 코드는 Service.class 라는 자바 코드와 완전히 같다.&#x20;

이제 이 예제를 구체화한 타입 파라미터로 사용해 다시 작성해보자.&#x20;

```kotlin
inline fun <reified T> loadService(){ // 타입 파라미터를 reified로 표시한다
    return ServiceLoader.load(T::class.java) // T::class 타입 파라미터의 클래스를 가져옴 
}
val serviceImpl = loadService<Service>()
```

훨씬 짧아졌다. 구체화 했기 때문에 지정된 타입 파라미터로 지정된 java.lang.Class 정보를 알 수 있고, 이것을 제네릭임에도 불구하고 보통 코드처럼 사용할 수 있다.&#x20;



## 실체화한 타입 파라미터의 제약&#x20;

실체화한 타입 파라미터는 유용하지만 몇 가지 제약이 있다.&#x20;

* 실체화한 타입 파라미터 사용 가능&#x20;
  * 타입 검사와 캐스팅 (is, as)
  * 10장에서 설명할 코틀린 리플렉션 API
  * 코틀린 타입에 대응하는 java.lang.Class를 얻기
  * 다른 함수를 호출할 때 타입 인자로 사용&#x20;
* 불가능&#x20;
  * 타입 파라미터 클래스의 인스턴스 생성&#x20;
  * 타입 파라미터 클래스의 동반 객체 메서드 호출
  * 실체화한 타입 파라미터를 요구하는 함수를 호출하면서 실체화하지 않은 타입 파라미터로 받은 타입을 타입인자로 넘기기&#x20;
  * 클래스,프로퍼티,인라인 함수가 아닌 함수의 타입 파라미터를 reified로 지정하기





