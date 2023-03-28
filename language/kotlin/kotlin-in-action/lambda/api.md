# 컬렉션 함수형 API

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

<img src="../../../../.gitbook/assets/file.excalidraw (27).svg" alt="flatMap" class="gitbook-drawing">

* 먼저 문자열을 리스트로 변환(map)하면 `[[a,b,c],[d,e,f]]` 처럼 리스트 안에 리스트로 될 것이다.&#x20;
* flatten을 통해 평평하게 만들면 `[a,b,c,d,e,f]` 하나의 리스트로 평평해진다.



## flatten

* 단순히 평평하게 펼치고 싶다면 **flatten()**을 호출하자.
