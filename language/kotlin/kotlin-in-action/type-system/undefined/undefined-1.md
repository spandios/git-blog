---
description: >-
  컬렉션을 다룰 때 가장 많이 쓰는 연산은 인덱스를 사용해 원소를 읽거나 쓰는 연산과 어떤 값이 컬렉션에 속해있는지 검사하는 연산이다. 이런
  연산 모두 연산자 구문으로 사용할 수 있다.
---

# 관례

## 컬렉션 범위에 대해 쓸 수 있는 관례

**인덱스 연산자:** 인덱스를 사용해 원소를 설정하거나 가져오고 싶을 때 a\[index]라는 식으로 사용할 수 있다.

### 인덱스로 원소에 접근: get과 set

`val value = map[key], value[key] = newValue`에서 사용된 인덱스 연산자도 관례를 따르고 있다.&#x20;

원소를 읽는 연산은 get 연산자 메서드로 변환되고, 원소를 쓰는 연산은 set 연산자 메서드로 변환된다.&#x20;

```kotlin
operator fun Point.get(index: Int): Int{ // operator가 있는 get 이름의 확장함수
    return when(index){
        0 -> x
        1 -> y
        else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}
val p = Point(1,2)
p[0] == 1
p[1] == 2
```

```kotlin
operator fun Point.set(index: Int, value: Int){ // operator가 있는 set 이름의 확장함수
    when(index){
        0 -> x = value
        1 -> y = value
        else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}
p[0] = 2 => p[0] == 2 
p[1] = 1 => p[1] == 1
```

## in 관례

in연산자는 객체가 컬렉션에 들어 있는지 검사한다. 이런 in 연산자와 대응하는 함수는 **contains**이다.

아래 코드는 **contains**의 예시로 어떤 점이 사각형 영역에 들어가는지 판단하는 코드이다.

```kotlin
data class Rectangle(val upperLeft: Point, val lowerRight: Point)

operator fun Rectangle.contains(p: Point): Boolean{
    return p.x in upperLeft.x until lowerRight.x && // until 함수로 열린 범위를 만듬 
            p.y in upperLeft.y until lowerRight.y
}
```

{% hint style="info" %}
열린 범위 (until) vs 닫힌 범위 (..)&#x20;

* 열린 범위는 끝 값을 포함하지 않는다. 10 until 20 => 10 \~ 19
* 닫힌 범위는 끝 값을 포함한다. 10 .. 20 => 10 \~ 20&#x20;
{% endhint %}



### rangeTo 관례

범위를 만들려면 .. 구문을 사용해야 한다. 이 연산자는 **rangeTo** 함수를 간략하게 표현한다.&#x20;



### for 루프를 위한 iterator 관례

* for 루프도 범위 검사와 같이 in 연산자를 사용하지만 의미는 다르다. `for ( x in list){}`와 같은 문장은 list.iterator()를 호출해서 이터레이터를 얻은 다음, 자바와 마찬가지로 그 이터레이터에 대해 hasNext와 next호출을 반복하는 식으로 변환된다.&#x20;
* 하지만 코틀린에서는 이 또한 관례이므로 iterator 메서드를 확장 함수로 정의할 수 있다. 이런 성질로 인해 일반 자바 문자열에 대해 for 루프가 가능하다. 코틀린 표준 라이브러리가 String 상위 클래스인 CharSequence에 대해 iterator 확장 함수를 제공하고 있기 때문이다. `operator fun CharSequenece.iterator(): CharIterator`
* 아래 코드는 닫힌 범위에 iterator를 정의해 for 루프에 사용할 수 있게 한다.

```kotlin
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
    object : Iterator<LocalDate> {
        var current = start
        override fun hasNext() = current <= endInclusive
        override fun next() = current.apply { current = plusDays(1) }
    }
    
val newYear = LocalDate.ofYearDay(2023,1)
val daysOff =newYAer.minusDays(1) .. newYear
for (dayOff in daysOff) {println(dayOff)}  // 2022-12-31 , 2023-01-01
```



## 구조 분해 선언

구조 분해 선언을 사용하면 복합적인 값을 분해해서 여러 다른 변수를 한꺼번에 초기화 할 수 있다. 다음과 같이 말이다.

```kotlin
class Point(val x : Int, val y : Int)
val p = Point(10,20)
val (x, y) = p  
x => 10, y => 20
```

* 구조 분해 선언은 관례를 사용한다.&#x20;
* 구조 분해 선언의 각 변수를 초기화하기 위해 componentN이라는 함수를 호출한다. 여기서 N은 구조 분해 선언에 있는 변수 위치에 따라 붙은 번호이다. 앞의 예제 코드는 다음과 같이 컴파일 될 것이다.

```kotlin
val x = p.component1() 
val y = p.component2()
```

* data class의 주 생성자에 들어있는 프로퍼티에 대해서는 컴파일러가 자동으로 componentN 함수를 만들어준다.&#x20;
* 데이터 타입이 아닌 클래스라면 다음과 같이 구현하면 된다.&#x20;

```kotlin
class Point(val x : Int, val y : Int){
    operator fun component1() = x
    operator fun component2() = y
}
```

* 컬렉션에 대해서도 구조 분해가 가능하다. 코틀린 표준 라이브러리에서는 맨 앞의 **다섯 원소**에 대한 componentN을 제공한다.&#x20;
* 표준 라이브러리의 Pari나 Triple 클래스를 사용하면 함수에서 여러 값을 더 간단하게 반환할 수 있다. 이 클래스들은 그 안에 담겨있는 원소의 의미를 말해주지 않으므로 가독성이 떨어질 수 있지만, 직접 클래스를 작성할 필요가 없기 때문에 코드는 더 단순해진다.&#x20;



### 구조 분해 선언과 루프&#x20;

루프 안에서도 구조 분해 선언을 사용할 수 있는데 특히 맵의 원소에 대해 이터레이션할 때 구조 분해 선언이 유용하다.&#x20;

```kotlin
for printEntries(map: Map<String,String){
    for ((key, value) in map){
        println("$key -> $value")
    }
}
```

이 예제는 두 가지 코틀린 관례를 활용한다. 하나는 객체를 이터레이션하는 관례이고, 다른 하나는 구조 분해 선언이다.&#x20;

코틀린 표준 라이브러리에는 맵에 대한 확장 함수로 iterator가 들어있다. 그 iterator는 맵 원소에 대한 이터레이터를 반환하기 때문에 자바와 달리 코틀린에서는 맵을 이터레이션 할 수 있다.&#x20;

