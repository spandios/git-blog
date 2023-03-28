---
description: object 키워드는 다양한 상황에서 사용하지만 모든 경우 클래스를 정의와 동시에 인스턴스를 생성한다는 공통점이 있다.
---

# object 키워드

## 객체 선언 - 싱글턴 쉽게 만들기

코틀린은 객체 선언 기능을 통해 싱글턴 패턴을 언어에서 기본적으로 지원한다. 객체 선언은 클래스 선언과 그 클래스에 속한 단일 인스턴스의 선언을 합친 것이다. Payroll `&& Payroll()`

```kotlin
object Payroll{
    val allEmpolyees = arrayListOf<Person>()
    fun calculateSalary(){
        ~~
    }
}
// 사용
Payroll.calculateSalary() // 생성자를 통해 인스턴스를 생성하지 않았다!!
```

* object는 클래스를 정의하고 그 클래스의 인스턴스를 만들어서 변수에 저장하는 작업을 단 한 문장으로 처리한다. 싱글턴 객체는 생성자 호출 없이 즉시 만들어지기 때문에 생성자가 필요가 없다.
* 객체 선언도 클래스나 인터페이스를 상속할 수 있다. 특히 특정 인터페이스를 구현해야 하는데, 구현 내부에 상태가 필요하지 않을 경우 유용하다.&#x20;

```kotlin
// 이제 필요한 곳에서 싱글턴 객체를 마음껏 사용할 수 있다. 
object CaseInsensitiveFileComparator : Comparator<File> {
    override fun compare(file1: File, file2: File): Int {
        return file1.path.compareTo(file2.path, ignoreCase = true)
    }
} 
```

### 클래스 안에서의 객체 선언&#x20;

클래스 안에서도 선언할 수 있다. 물론 싱글턴이다.

```kotlin
data class Person(val name:String){
    object NameComparator : Comparator<Person> {
        override fun compare(p1: Person, p2: Person): Int {
            return p1.name.compareTo(p2.name)
        }
    }
}

val persons = listOf(Person("Bob"), Person("Alice"))
println(persons.sortedWith(Person.NameComparator))
```



## 동반 객체 - 팩토리 메서드와 정적 멤버가 들어갈 장소&#x20;

* 코틀린 안에는 자바처럼 static 키워드를 사용하지 않고 정적인 멤버가 없다. 보통 최상위 함수(정적 메서드)와 객체 선언을 활용하면 되기 때문이다.
* 대부분의 경우 최상위 함수를 활용하는 걸 더 권장하지만 최상위 함수는 private으로 표시된 클래스 멤버에는 접근할 수 없다.&#x20;
* 클래스 안에 정의된 객체에 companion 키워드를 붙이면 그 클래스의 동반 객체로 만들 수 있다. 동반 객체는 private 클래스 멤버에도 접근 가능하다.

```kotlin
class A {
    companion object {
        fun bar() {
            println("Companion object called")
        }
    }
}

A.bar()
```

* 동반 객체는 바깥 클래스의 private 멤버에 접근할 수 있다.&#x20;

```kotlin
class A(private val b : String) {
    companion object {
        fun bar() {
            println("Companion object called")
            val a = A("b")
            printlnt(a.b) // "b"
        }
    }
}
```

* 아래 예시는 동반 객체를 이용해 팩토리 메서드를 이용한 것이다. private 생성자에도 접근할 수 있는 것을 볼 수 있다.

```kotlin
class User private constructor(val nickname: String) {
    companion object {
        fun newSubscribingUser(email: String) =
                User(email.substringBefore('@'))

        fun newFacebookUser(accountId: Int) =
                User(getFacebookName(accountId))
    }
}
```



### 동반객체에 이름 붙이기&#x20;

```kotlin
class A{
     companion object B{ // 동반객체에 B라는 임을 붙여 사용한다.
        fun bar() {
            println("Companion object called")
            println()
            Hello.foo()
            Hello.a
            val a = A("E")
            a.name
        }
    }
}

A.B.bar()
```



### 인터페이스 구현&#x20;

```kotlin
class Person(val name:String){
    companion object : JsonFactory<Person> { // 인터페이스를 구현한다.
        override fun fromJson(jsonText: String): Person {
                // ~~
        }
    }
}

fun loadFromJson<T>(factory: JsonFactory<T>): T {
    // ~~ 
}

loadFromJson(Person) // JsonFactory를 구현한 객체를 넘길 때 Person 클래스이름을 넘김
```



### 동반객체 확장

* 기존 클래스에 대해 호출할 수 있는 새로운 함수 정의를 하고 싶은데 관심사 분리가 필요하다면 확장함수를 이용하면 좋다.&#x20;
* 동반객체도 확장함수를 통해 확장할 수 있다.&#x20;
* companion object가 미리 정의되어 있어야 확장할 수 있다.

```kotlin
class Person(val name:String){
   companion object{
       
   }
}

fun Person.Companion.fromJson(jsonText: String): Person {
    return Person(jsonText)
}
```



### 무명 객체 식

* 무명 객체는 자바의 **무명 내부 클래스**를 대신한다. 구문은 객체 선언과 같지만 객체 이름이 없다.&#x20;
* 자바와 달리 **여러** 인터페이스를 구현하거나 클래스를 확장할 수 있으며, **final이 아닌 변수에도 접근이 가능하고 값도 변경 가능**하다.&#x20;

```kotlin
fun countClicks(window: Window){
    var clickCount = 0 
    var enterCount = 0
    window.addMouseListener(object : MouseAdapter(){
        override fun mouseClicked(e: MouseEvent) {
            clickCount++
        }

        override fun mouseEntered(e: MouseEvent) {
            enterCount++
        }
    })
}
```

{% hint style="info" %}
객체 식은 무명 객체 안에서 여러 메서드를 오버라이드해야 하는 경우에 유용하다.&#x20;

추상 메서드가 하나뿐인 인터페이스 SAM은 SAM 변환 지원을 활용하는게 더 좋다.
{% endhint %}

