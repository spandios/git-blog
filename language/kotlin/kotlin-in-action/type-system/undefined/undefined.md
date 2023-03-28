# 연산자 오버로딩

## 이항 산술 연산자

* 자바에서는 원시 타입에 대해서만 산술 연산자를 사용할 수 있고, String에 대해서만 +를 추가적으로 지원한다.
* 코틀린은 BigInteger에 add 대신 + 연산이나 컬렉션 원소 추가에도 += 연산자를 사용할 수도 있다.&#x20;

```kotlin
data class Point(val x: Int, val y: Int){
    operator fun plus(other: Point): Point{ //operator 키워드
        return Point(x + ohter.x , y + other.y)
    }
}
// point1(1,1) + point(2,2) = point(x=3, y= 3)
```

* 연산자 오버로딩을 위해 함수 앞에 `operator` 키워드를 선언한 것을 볼 수 있다. &#x20;



### 확장 함수로 정의

* 연산자를 멤버 함수로 만드는 대신 확장 함수로도 정의 할 수 있다.&#x20;
* 보통 외부 함수의 클래스에 대한 연산자를 정의할 때는 관례를 따르는 이름의 확장 함수로 구현하는게 일반적인 패턴이다.&#x20;

```kotlin
operator fun Point.plus(ohter: Point): Point{
    return Point(x + other.x, y + other.y)
}
```

* 코틀린에서는 직접 연산자를 만들어 사용할 수 없고 언어에서 미리 정해둔 연산자만 오버로딩할 수 있으며, 관례를 따르기 위해 클래스에서 정의해야 하는 이름이 연산자별로 정해져 있다.&#x20;

| 식       | 함수 이름  |
| ------- | ------ |
| a \* b  | times  |
| a / b   | div    |
| a % b   | plus   |
| a + b   | rem    |
| a - b   | minus  |

### 다른 피연산자 타입

두 피연산자가 같은 타입일 필요는 없다.&#x20;

```kotlin
operator fun Point.times(sacle:Double) : Point{ // double 타입의 피연산자에 대해서
    return Point((x * scale).toInt(), (y * scale).toInt())
}
```

### 다른 반환 타입

반환 타입이 피연산자와 일치하지 않아도 된다.

```kotlin
operator fun Char.times(count:Int) : String{ // 반환 타입이 String이다
    return toString().repeat(count)
}
```



### 복합 대입&#x20;

\+=, -= 등의 연산자인 복합 대입 연산자도 오버로딩 가능하다. plusAssign, minusAssign 같은 이름의 함수를 정의하면 된다.&#x20;

```kotlin

operator fun <T> MutableCollection<T>.plusAssign(element: T){
    this.add(element)
}
```

* plus와 plusAssign을 모두 동시에 정의하면 안 된다.&#x20;
* \+= -=는 항상 변경 가능한 컬렉션에 작용해 메모리에 있는 객체 상태를 변화 시킨다. `list +=3` (3 원소를 추가함)
* \+와 -는 항상 새로운 컬렉션을 반환한다. `val newList = list + listOf(4,5)`



## 단항 연산자 오버로딩

이항 연산자 때와 마찬 가지로 미리 정해진 이름의 함수를 선언하고 operator 키워드를 선언하면 된다.

```kotlin
operator fun Point.unaryMinus(): Point{
    return Point(-x,-y) // 각 점에 음수를 취한 새 포인트를 반환 
}
```

| 식        | 함수 이름  |
| -------- | ------ |
| +a       | times  |
| -a       | div    |
| !a       | plus   |
| ++a, a++ | rem    |
| a - b    | minus  |

## 비교 연산자 오버로딩

코틀린에서는 == 비교 연산자를 직접 사용 할 수 있어서 비교 코드가 eqauls나 compareTo를 사용한 코드보다 더 간결하며 이해하기가 쉽다.&#x20;

### eqauls : 동등성 연산자&#x20;

전에 코틀린이 == 연산자 호출이 equals 메서드 호출로 컴파일 한다는 걸 알게 되었는데 사실 이것도 한 관례에 포함된다.&#x20;

a == b 라는 비교를 처리할 때 a가 널인지 판단해서 널이 아닌 경우에만 a.equals(b)를 호출한다. 만약 a가 널일 경우 b도 널이여야 true가 반환 될 것이다. `a?.eqauls(b) ?: b == null`

**식별자 비교 연산자 ===** 는 자바의 == 연산자와 같이 두 피연산자가 서로 같은 객체를 가리키는지 비교한다.&#x20;

eqauls에는 다른 연산자 오버로딩과 다르게 Any에 정의된 메서드이기 때문에 override가 필요하다.&#x20;

Any에는 eqauls에 operator가 붙어 있으며 그것을 오버라이드 할 때는 operator를 붙이지 않아도 자동으로 적용된다.&#x20;



### compareTo: 순서 연산자&#x20;

자바에서 정렬이나 최대값, 최솟값 등 값을 비교해야하는 알고리즘에 필요한 클래스는 Comparable 인터페이스를 구현해야한다. Comparable 인터페이스에 들어있는 compareTo 메서드는 한 객체와 다른 객체의 어떤 값을 비교해 정수로 나타내준다. 그런데 자바는 <, > 등의 연산자로는 원시 타입 값만 비교할 수 있다. 다른 모든 타입의 값들은 명시적으로 compareTo메서드를 사용해야 한다.&#x20;

코틀린도 똑같은 Comparable 인터페이스를 지원한다. 거기에 compareTo 메서드를 호출하는 관례도 같이 제공한다. 따라서 **비교 연산자 (<, >, <=, >=)는 compareTo 호출로** 컴파일 된다. eqauls와 마찬가지로 compareTo에도 operator 변경자가 붙어있으므로 오버라이드 할 때는 operator를 붙이지 않아도 된다.

compareTo메서드를 오버라이드할 때 필드를 직접 비교하는 방식은 성능은 좋지만 더 복잡하다. 언제나 처음엔 성능 보다는 이해하기 쉽고 간결한 코드를 작성한 뒤, 나중에 그 코드가 자주 호출되어 성능 이슈가 생길 때 성능 개선을 하자.

코틀린 표준 라이브러리에 **compareValuesBy**함수는 직접 필드를 비교하는 것보다 조금 느리지만 매우 가독성이 좋고 편하게 사용 가능하다.&#x20;

```kotlin
class Person(val firstName: String, val lastName:String) : Comparable{
    override fun compareTo(other:Person): Int{
        return compareValuesBy(this, other, Person::lastName, Person::firstName)
    }
}
```







