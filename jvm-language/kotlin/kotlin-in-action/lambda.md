---
description: 다른 함수에 넘길 수 있는 작은 코드 조각
---

# Lambda

## 람다: 코드 블록을 함수 인자로 넘기기&#x20;

예전 자바 8 이전 버전에서는 함수를 인자로 넘기기 위해서는 무명 내부 클래스를 통해야만 했다. 자바 8부터 람다가 도입이 되어 함수형 프로그래밍을 할 수 있다. 함수형 프로그래밍에서는 함수 또한 값처럼 다뤄 인자로 넘겨줄 수 있다.&#x20;

버튼 리스너 구현에서 무명 내부 클래스 방식으로 한 것과 람다 방식으로 한 것을 비교해보자.

```java
// JAVA 
button.setOnClickListener(new OnClickListener(){
    @Override
    public void onClick(View view){
        // 수행할 동작
    }
})
```

```kotlin
// Kotlin
button.setOnClickListener{ 
    // 수행할 동작
} 
```

자바8부터도 가능한 람다는 무명 내부 클래스 방식과 달리 불필요한 코드가 제거 되어 훨씬 가독성이 좋다.&#x20;



## 컬렉션에서의 람다&#x20;

기존 람다가 없던 시절은 자바 컬렉션에서 편하게 사용할 수 있는 라이브러리가 적었다. 만약 이름과 나이를 가지는 Person 클래스가 있고, 그 클래스로 이루어진 리스트가 있다고 하자. 이 리스트에서 가장 연장자를 찾으려면 반복문을 통해 직접 찾을 수 있다. 하지만 코틀린은 람다 방식을 이용해 라이브러리 함수를 제공한다. 다음과 같이 말이다.&#x20;

```kotlin
val peoples = listOf(Person("Alice", 24), Person("BB", 30))
// maxBy 함수, 가장 큰 원소를 찾기 위해 기준이 되는 값을 반환하는 함수를 인자로 받는다.
// 반환 문법의 이해가 잘 안 될수 있다. 아래 람다 문법을 보자.
println(people.maxBy{it.age}) // Person의 나이 프로퍼티를 넘겨 그 기준으로 가장 높은 값을 찾아준다.
```

{% hint style="info" %}
단지 함수나 프로퍼티를 반환하는 역할을 수행하는 람다는 **멤버 참조로** 대치 가능하다.&#x20;

`people.maxBy(Person::age)`
{% endhint %}

## 람다 식의 문법

람다는 값처럼 여기저기 전달할 수 있는 동작의 모음이다. 따라서 변수에 저장할 수도 있지만, 바로 함수 인자에 람다를 정의해 넘기는 경우가 대부분이다.&#x20;

```kotlin
{ x: Int, y: Int -> x + y }
```

람다는 항상 중괄호 {}에 둘러쌓여 있다. 화살표가 인자와 본문을 구분해준다.  만약 위의 maxBy함수에 넘긴 람다를 정식으로 표현하면 이렇게 된다. 위에서 봤던 것과 달리 가독성이 떨어지는 것을 볼 수 있다.&#x20;

```kotlin
people.maxBy({p : Person -> p.age})
```



### 코틀린스러운 람다 함수

이제 단계별로 코틀린스러운 람다함수를 작성해보자.



1. **중괄호 없애기**

코틀린은 마지막 인자가 람다라면 인자 괄호 에서 벗어나 밖으로 뺄 수 있다는 관습이 있다.&#x20;

```kotlin
people.maxBy(){p : Person -> p.age}
```

더 나아가서 람다가 함수의 유일한 인자이고 위 처럼 괄호 뒤에 람다를 썼다면 빈 괄호(인자)를 제거해도 된다.

```kotlin
people.maxBy{p : Person -> p.age}
```

joinToString()이라는 표준 라이브러리 함수를 보면 맨 마지막 인자로 함수를 받는다.&#x20;

이름 붙인 인자를 통해 람다를 넘겨보는 것과 람다를 괄호 밖으로 전달하는 방식을 봐보자.

```
people.joinToString(seperator=" ", transform={p: Person -> p.name})
```

```
people.joinToString(seperator=" "){p:Person -> p.name}
```

이름 붙인 인자 방식은 코드가 더 많지만 명확하고, 괄호 밖으로 전달하는 방식은 간결하지만 메서드를 잘 모르는 사람이 봤을 때는 람다의 용도를 이해하기 힘든 단점이 있다.



2. **파라미터 타입 제거**

컴파일러는 람다 파라미터의 타입도 추론 할 수 있다. 따라서 파라미터 타입을 굳이 명시할 필요가 없다.&#x20;

컴파일러가 타입을 추론하지 못하는 경우도 있는데 그건 나중에 생각하고 일단 컴파일러가 경고할 때만 타입을 명시하자.

```
people.maxBy{p -> p.age}
```

더 나아가 파라미터가 하나이고 타입을 추론할 수 있을 경우 람다의 파라미터 이름을 default 이름인 it으로 대체해서 사용할 수 있다.

```
people.maxBy{it.age}
```



## 외부 변수에 접근 - 클로저

코틀린 람다는 바깥에 있는 로컬 변수에 접근할 수 있으며 자바와 달리 파이널 변수가 아닌 변수에도 접근 가능하다. 또한 람다 안에서 바깥의 변수를 변경해도 된다.&#x20;

```kotlin
fun printProblemCounts(response: Collection<String>){
    var clientErrors = 0
    var serverErrors = 0
    response.forEach {
        if (it.startsWith("4")){
            clientErrors++ // 람다 안에서 바깥 변수에 접근 및 변경 
        }else if (it.startsWith("5")){
            serverErrors++ // 람다 안에서 바깥 변수에 접근 및 변경
        }
    }
    println("$clientErrors client errors, $serverErrors server errors")
}
```

clientErrors, serverError 변수와 같이 람다 안에서 사용된 외부 변수를 람다가 포획한 변수라고 한다.

기본적으로 함수 안에 정의된 로컬 변수의 생명주기는 함수가 끝나면 끝난다. 하지만 람다에서는 생명주기가 달라질 수 있다. 포획한 변수가 있는 람다를 저장한 뒤 함수가 끝난 뒤에 실행해도 람다는 여전히 포획한 변수를 읽거나 쓸 수 있다. 어떻게 그런 동작이 가능할까?&#x20;

파이널 변수를 포획한 경우는 람다 코드를 변수 값과 함께 저장한다.&#x20;

파이널 변수가 아닌 경우는 특별한 래퍼로 감싸고, 그 래퍼에 대한 참조를 람다 코드와 함께 저장한다.&#x20;



비동기적으로 실행 된느 코드로 활용할 경우 함수 호출이 끝난 다음에서야 로컬 변수가 변경된다. 예를 들어 다음 코드는 버튼 클릭 횟수를 제대로 셀 수 없다.&#x20;

```kotlin
fun tryToCountButtonClick(btn:Button) : Int {
    var clicks = 0 
    button.onClick{ clicks++ }
    return clicks
}
```

이 함수는 항상 0을 리턴할 것이다. 왜냐하면 onClick handler가 실행 되면 clicks의 값을 변경하기는 하겠지만 그 전에 이미 clicks가 0일 때 리턴되어 함수가 끝나기 때문이다.&#x20;



## 멤버 참조

코틀린은 이중 콜론 `::` 을 사용해 함수를 값으로 바꾸는 식인 **멤버 참조**를 사용할 수 있다.&#x20;

멤버 참조는 프로퍼티나 메서드를 단 하나만 호출하는 함수 값을 만들어준다. 멤버 참조는 그 멤버를 호출하는 람다와 같은 타입이다. 따라서 앞서 살펴봤던 `people.maxBy{it.age}`나 `people.maxBy{p -> p.age}`와 같다.

생성자 참조를 사용하면 클래스 생성을 연기하거나 저장할 수 있다.&#x20;

```kotlin
data class Person(val name : String)
val createPerson = ::Person
val p = createPerson("heo")
```

확장 함수도 멤버 함수와 같은 방식으로 참조할 수 있다.

```kotlin
fun Person.isAdult() = age > =21
var predicate = Person::isAdult
```

## 컬렉션 함수형 API

### all

어떤 조건을 모든 원소가 다 만족하는지

```kotlin
val canBeInClub27 = {p: Person -> p.age <= 27}
val people = listOf(Person(age=36), Person(age,22))
people.all(canBeInClub27) ==> false
```

### any&#x20;

어떤 조건을 한 원소라도 만족하는지&#x20;

```kotlin
val canBeInClub27 = {p: Person -> p.age <= 27}
val people = listOf(Person(age=36), Person(age,22))
people.any(canBeInClub27) ==> true
```

### count

```kotlin
val canBeInClub27 = {p: Person -> p.age <= 27}
val people = listOf(Person(age=36), Person(age,22))
people.count(canBeInClub27) ==> 1
```

### find&#x20;

```kotlin
val canBeInClub27 = {p: Person -> p.age <= 27}
val people = listOf(Person(age=36), Person(age,22))
people.find(canBeInClub27) ==> Person(age = 22)
```



## groupBy

컬렉션의 모든 원소를 어떤 특성에 따라 여러 그룹으로 나눌 때 사용한다.  만약 문자열 첫 글자에 따라 분류하고 싶다면 아래와 같이 할 수 있다.

```kotlin
val list = listOf("a", "ab", "b")
list.groupBy(String::first)) ==> {a=[a,b], b=[a]} | Map<String, List<String>>
```

## flatMap

flat 펼치고, map 매핑한다. 리스트의 리스트가 있는데 이것을 리스트 하나로 모으고 싶다면 flatMap을 사용하자. `listOfLists.flatMap(it.property)`

문자열에 적용한 예를 보자.&#x20;

```kotlin
val strings = listOf("abc", "def") 
strings.flatMap{it.toList()} ==> [a,b,c,d,e,f]
```

<img src="../../../.gitbook/assets/file.excalidraw (4).svg" alt="flatMap" class="gitbook-drawing">

먼저 문자열을 리스트로 변환(map)하면 `[[a,b,c],[d,e,f]]` 처럼 리스트 안에 리스트로 될 것이다. 그리고 flatten을 통해 평평하게 만들면 `[a,b,c,d,e,f]` 하나의 리스트로 평평해진다.



## flatten

단순히 평평하게 펼치고 싶다면 listOfLists.flatten()을 호출하자.



## lazy 지연 컬렉션 연산

앞서 살펴본 map이나 fillter 같은 함수는 결과 컬렉션을 즉시 생성한다. 다시말해 컬렉션 함수를 연쇄하면 매 단계마다 계산 중간 결과를 새로운 임시 컬렉션에 담는다는 말이다. 이 때 시퀀스를 사용하면 중간 임시 컬렉션을 사용하지 않고 컬렉션 연산을 연쇄할 수 있다.&#x20;

만약 원소가 별로 없다면 임시 컬렉션이 생겨도 별 문제 없지만, 수백만개라면 최적화가 필요하다.&#x20;

```kotlin
people.asSequence()
    .map(Person::name)
    .filter{it.startWith("A")}
    .toList() // 리스트로 반환
```

### 중간 연산과 최종연산

시퀸스에 대한 연산은 중간(intermediate) 연산과 최종(terminal) 연산으로 나뉜다.&#x20;

중간 연산은 다른 시퀸스를 반환한다.  그 시퀸스는 최초 시퀸스의 원소를 반환하는 방법을 안다.&#x20;

최종 연산은 결과를 반환한다. 결과는 컬렉션에 일렬의 시퀀스를 적용해 얻은 결과다.

중간 연산은 항상 지연 계산되기 때문에 최종 연산이 없다면 아무 결과도 얻지 못한다.&#x20;

최종 연산을 호출할 때만 지연 됐던 모든 연산이 수행 될 것이다.

시퀀스의 연산 수행 동작은 기존 컬렉션과 다르다. map과 filter를 연쇄 연산하는 작업을 했다고 치자.&#x20;

직접 연산한다면 map을 각 원소에 수행해서 새로운 시퀀스 즉, 임시 컬렉션을 얻게 되고 그 임시 컬렉션에 다시 filter를 수행할 것이다.&#x20;

하지만 시퀀스는 다르다. 시퀀스는 모든 연산을 합쳐 각 원소에 한번에 처리할 것이다. 한 원소마다 map과 filter 작업을 수행하고, 다음 원소에 또 연쇄 작업을 수행하는 것이다. 때문에 임시 컬렉션이 필요가 없다.



## 수신 객체 지정 람다&#x20;

수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메서드를 호출할 수 있는 기능이 있다.

이것은 자바에는 없는 코틀린 람다의 독특한 기능이다.&#x20;

### with&#x20;

객체 이름을 반복하지 않고 그 객체에 대해 다양한 연산을 할 수 있다면 편할 것이다.&#x20;

```kotlin
fun alphabet(): String{
    val result = StringBuilder()
    for (letter in 'A'..'Z'){
        result.append(letter)
    }
    result.append("\nNow I know the alphabet!")
    return result.toString()
}

이제 with을 사용해보자
fun alphabet(): String{
  val stringBuilder = SringBuilder()
  return with(stringBuilder){
      for (letter in 'A'..'Z'){
        this.append(letter) // this 명시 
    }
    append("\nNow I know the alphabet!") // this 생략
    return this.toString()
  }
}
```

with가 반환하는 값은 람다 코드를 실행한 결과(람다 본문 마지막)이다.&#x20;

### apply

apply는 람다의 결과 대신 수신 객체가 필요할 때 사용한다. with과 거의 같지만 apply는 항상 자신에게 전달된 객체를 반환한다는 것이다. 주로 객체를 생성 하면서 즉시 프로퍼티를 초기화할 때 사용된다.&#x20;

```kotlin
fun createTextView(context : Context) = TextView(context).apply{
    text = "Apply Test"
    textSize = 20.0
}
```

이제 createTextView를 호출하면 with과 달리 apply의 수신객체인 TextView 인스턴스가 반환 될 것이다.&#x20;

{% hint style="info" %}
표준 라이브러리 **buildString**을 이용하면 StringBuilder 생성과 toString호출을 자동으로 해줌
{% endhint %}



