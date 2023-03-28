---
description: 루프와 명령형 코드에서의 return 동작과 람다에서 return 동작은 다르다.
---

# 고차 함수 흐름제어

## 람다 안의 return 문: 람다를 둘러싼 함수로부터 반환&#x20;

먼저 일반 루프 안에서 return을 사용해보자.

```kotlin
data class Person(val name:String, val age: Int)

val people = listOf(Person("Alice", 29), Person("Bob", 31))

fun lookForAlice(people: List<Person>){
    for (person in people){
        if (person.name == "Alice"){
            println("Found!")
            return
        }
    }
    println("Alice is not found")
}

// 결과: Founded!
```

이 코드를 forEach를 사용해 람다를 사용하고 람다 본문안에 return을 사용해보자. 결과는 위와 같다.

```kotlin
fun lookForAlice2(people: List<Person>){
    people.forEach{
        if (it.name == "Alice"){
            println("Found!")
            return
        }
    }
    println("Alice is not found")
}

```

* 람다 안에서의 return은 람다로부터만 반환되는게 아니라 그 람다를 호출하는 함수가 실행을 끝내고 반환된다.&#x20;
* 이렇게 자신을 둘러싸고 있는 블록보다 더 바깥에 있는 블록을 반환하게 만드는 return 문을 논로컬 (non-local)리턴이라 한다.&#x20;
* return이 바깥쪽 함수를 반환시킬 수 있는 때는 람다를 인자로 받는 함수가 인라인 함수인 경우 뿐이다. forEach도 인라인 함수이므로 람다 본문과 함께 인라이닝된다. 따라서 return 식이 바깥쪽 함수를 반환시키도록 쉽게 컴파일 할 수 있다.&#x20;
* 인라이닝되지 않은 함수에 전달되는 람다 안에서 return은 사용할 수 없다. 인라이닝되지 않는 함수는 람다를 변수에 저장할 수 있고, 바깥쪽 함수로부터 반환된 뒤에 저장해 둔 람다가 호출될 수도 있다. 그런 경우 람다 안의 return이 실행되는 시점이 바깥쪽 함수를 반환시키기엔 너무 늦은 시점일 수 있다.  다시 말해, 호출 되는 곳에서 인라이닝 되지 않기 때문에 그 람다 코드가 가까이 없고 외부의 다양한 환경에서 호출되다보니 호출 시점을 정확히 알지 못하기 때문이다.&#x20;

## 람다로부터 반환: 레이블을 사용한 return&#x20;

* 람다에서도 로컬 return을 사용할 수 있다. 람다 안에서 로컬 return은 for 루프의 break과 비슷한 역할을 한다. 로컬 return은 람다의 실행을 끝내고 람다를 호출했던 코드의 실행을 계속 이어나간다.&#x20;
* 로컬 return과 논로컬 return을 구분하기 위해 레이브을 사용하면 된다. 아래 예시를 보자.&#x20;
* 2 line에서 람다 식앞에 레이블을 붙이고 5 line에서 앞에서 정의한 레이블을 명시하고 있다.

{% code lineNumbers="true" %}
```kotlin
fun lookForAlice3(people: List<Person>){
    people.forEach label@{
        if (it.name == "Alice"){
            println("Found!")
            return@label
        }
    }
    println("Alice is not found") // 항상 실행 됨
}
```
{% endcode %}

* 레이블을 붙이지 않고 람다를 인자로 받는 인라인 함수의 이름을 return 뒤에 레이블로 사용해도 된다.

```kotlin
fun lookForAlice4(people: List<Person>){
    people.forEach{
        if (it.name == "Alice"){
            println("Found!")
            return@forEach
        }
    }
    println("Alice is not found") // 항상 실행 됨
}
```



## 무명 함수 리턴 : 기본적으로 로컬 return&#x20;

* 논로컬 반환문은 장황하다. 무명 함수는 코드 블록을 함수에 넘길 때 사용할 수 있는 다른 방법이다.&#x20;
* 아래 코드는 람다 식 대신 무명함수를 사용한 것을 볼 수 있다. 무명 함수에서 리턴은 가장 가까운 함수를 반환시키기 때문에 forEach만 끝낼 것이며 따라서 Yahoo를 볼 수 있다.&#x20;

```kotlin
fun lookForAlice5(people: List<Person>){
    people.forEach(fun(person){
        if (person.name == "Alice"){
            println("Found!")
            return
        }
    })
    println("Yahoo~") // 항상 실행된다. 
}
```

* 사실 return에 적용되는 규칙은 간단하다. return은 fun 키워드를 사용해 정의된 가장 안쪽 함수를 반환한다. 람다 식은 fun을 사용해 정의되지 않았기 때문에 람다 밖의 함수를 반환 시키는 것이다. 따라서 fun으로 정의된 무명함수는 밖의 다른 함수를 반환시키지 않는다. 가장 가까운 fun을 찾고 그 범위를 반환하는 것이다.

<img src="../../../../.gitbook/assets/file.excalidraw (10).svg" alt="" class="gitbook-drawing">

{% hint style="info" %}
무명함수

무명함수는 일반함수와 비슷하고 함수 이름이나 파라미터 타입을 생략하다는 차이 밖에 없다.&#x20;

무명함수는 반환 타입도 생략할 수 있다. 무명 함수는 일반 함수와 비슷하지만 실제로는 람다 식에 대한 문법적 편의일 뿐이다. 따라서 람다의 규칙들(람다 식 구현 방법, 인라인 함수에 넘길 때 어떻게 본문이 인라이닝 되는지) 을 무명 함수에도 모두 적용할 수 있다.&#x20;
{% endhint %}

