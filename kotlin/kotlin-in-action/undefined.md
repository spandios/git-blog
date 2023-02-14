# 코틀린 기초

## 식이 본문인 함수

함수 본문이 한 줄이라면 중괄호와 return을 없앤 형식으로 처리할 수 있다.

이 때는 리턴 타입을 생략 가능하다. 자바와 다르게 함수가 클래스 안에 없어도 된다.

```kotlin
fun max(a:Int, b:Int) = if ( a > b )a else b 
```

## 변수

* 타입추론이 가능하기 때문에 초기화 시 타입 생략 가능
* 초기화하지않는다면 타입 명시 필요
* 변경 가능 변수 vs 변경 불가능 변수
  * val ⇒ 변경 불가능 (value)
  * var ⇒ 변경 가능 (variable)
  * 변경 불가능한 참조 + 변경 불가능한 객체를 side effect없는 함수와 조합하면 ⇒ 함수형 코드

```kotlin
val a = 32
var b : String

```

## 문자열 템플릿

```kotlin
val name = "허주영"

1. "Hello $name"
2. "Hello ${if(name == "허주영") "알파카" else "닝겐"}"

```

## Class

* 기본 가시성은 public
* Value Object: 코드 없이 값만 있는 클래스

```kotlin
class Person(val name: String)
```

* 프로퍼티

코틀린은 val, var에 따라 자동으로 getter, setter같이 접근자 메서드를 만들어준다.

하지만 게터 세터를 호출하는 대신 프로퍼티를 직접 사용해도 된다.

```kotlin
class Person(val name:String, var isMarried:Boolean) 
// name은 변경 불가능하므로 name을 비공개 필드, 공개 getter만 제공
// isMarried는 변경 가능하므로 비공개 필드, 공개 getter, 공개 setter도 제공한다. 

val person = Person("Heo", false)
person.isMarried = false // 프로퍼티 직접 사용 
println("$person.name, $person.isMarried") //직접 사용 

```

* 커스텀 접근자

```kotlin
class Rectangle(val height:Int, val widht:Int){
	val isSquare:Boolean 
	get(){
		return height == width
	}
	// or 
	get = height == width
}
```

## 패키지와 Import

* 코틀린은 패키지 구조와 디렉터리 구조가 맞아 떨어질 필요가 없다. 그러나 자바의 패키지 방식을 따르는게 좋다.
* start import시 하위 전체 클래스 뿐아니라 최상위까지도 가져온다.

## Enum

* enum은 enum뒤에 class를 붙이자
* enum은 class 앞에 존재해야지만 키워드다.

```kotlin
enum class Color(val r:Int, val g:Int, val b: Int){
	RED(255,0,0), ORANGE(255,165,0); //메서드를 정의하는 경우 ;를 넣어야한다.

	fun rgb() = (r * 256 + g) +256 + b
}

print(Color.RED.rgb())

```

## When

* switch처럼 분기문법이지만 더 강력하다
* break를 사용하지 않는다.
* 한 when 분기에 여러 값을 대입할 수 있다. Color.Red, Color.Green

```kotlin
fun getMeomonic(color:Color){
	when(color){
			Color.Red -> "Richard" 
			Color.ORANGE, Color.Green -> "Of" // 

	}
}
```

* java처럼 상수만 사용할 수 있는게 아닌 임의의 객체도 사용 가능하다.

```kotlin
fun mix(c1:Color, c2:Color) = 
when (setOf(c1,c2)){
	setOf(RED,YELLOW) -> ORANGE,
	setOf(YELLOW,BLUE) -> GREEN
}
```

* when은 인자가 없어도 되지만 가독성이 떨어진다. 각 조건은 boolean이다.

```kotlin
mixOptimized(c1:Color,c2:Color){
	when{
			(c1 == RED && c2 == YEELOW) -> GREEN
				(c1 == BLUE && c2 == YEELOW) -> RED		
	}
}

```

## 스마트 캐스트

* is로 타입을 체크할 때, 만약 타입이 맞다면 자동으로 컴파일러가 그 타입으로 캐스팅해주는 것

```kotlin
if ( e is Int){
	
}
```

## Iterator

* kotlin은 for loop가 없다 for (int i=0;\~\~)
* 대신 범위를 사용한다

```kotlin
for ( i in 1 .. 100) { //1부터 100까지 

}
```

* 거꾸로 + 증가 값

```kotlin
for ( i in 100 downTo 1 step2)
```

* map 구조분해 할당

```kotlin
for ( (index, element) in list.withIndex()){ // 인덱스와 함께 컬렉션을 이터레이션한다.

}
```

## In

in연산자를 이용해 어떤 값이 범위에 속하는지 검사할 수 있다.

!in은 값이 범위에 속하지 않은 것을 뜻한다.

```kotlin
"c" in "a" .. "z" == true
"Kotlin" in setOf("java", "Scala") == false
```

## Try Catch

* try도 식이다
