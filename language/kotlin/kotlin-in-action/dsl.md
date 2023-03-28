---
description: Domain Specific Language 영역 특화 언어
---

# DSL

DSL를 사용하면 표현력이 더 좋고 코틀린 다운 API를 설계할 수 있다.&#x20;



## API에서 DSL로&#x20;

궁극적으로 우리의 목표는 코드의 가독성과 유지 보수성을 좋게 유지하는 것이다. 그런 목표를 달성하려면 개별 클래스에 집중하는 것만으로는 부족하다. 클래스는 대부분 다른 클래스와 상호작용한다. 그런 상호작용이 일어나는 interface를 잘 살펴봐야하고 다른말로 하면 클래스의 API에 신경써야 한다.&#x20;

라이브러리용 API를 다른 유저에게 제공하지 않는다고 해도 사실 우리가 프로그래밍하는 클래스는 다른 클래스에게 API를 제공하고 있는 것과 같다. 따라서 상호작용을 이해하기 쉽고 명확하게 표현하는 API를 작성해야 지속 가능한 유지보수를 할 수 있을 것이다. 즉, API가 깔끔해야 한다.

API가 깔끔하다는 것은 코드를 읽는 사람이 어떤 일이 발생할지 명확하고 쉽게 이해할 수 있고, 코드가 간결해야 한다.&#x20;

| 일반 구문                                       | 간결한 구문                                                             | 코틀린 언어 특성        |
| ------------------------------------------- | ------------------------------------------------------------------ | ---------------- |
| StringUtil.capitalize(s)                    | s.capitalize()                                                     | 확장 함수            |
| 1.to("one"                                  | 1 to "one"                                                         | 중위 호출            |
| set.add(2)                                  | set += 2                                                           | 연산자 오버로딩         |
| map.get("key")                              | map\["key"]                                                        | get 메서드에 대한 관례   |
| file.use({f -> f.read()})                   | file.use {it.read()}                                               | 람다를 괄호 밖으로 빼는 관례 |
| <p>sb.append("yes") <br>sb.append("no")</p> | <p>with (sb) {</p><p>   append("yes")<br>   append("yes")<br>}</p> | 수신 객체 지정 람다      |

위의 표를 보면 코틀린이 어떻게 깔끔한 API를 만들 수 있게 도와주는지 보여준다.&#x20;

코틀린 DSL은 간결한 구문을 제공하는 기능과 그런 구문을 확장해서 여러 메서드 호출을 조합해 구조를 만들어낸다.&#x20;

그 결과 DSL은 메서드 호출만을 제공하는 API에 비해 더 표현력이 풍부해지고 사용하기 편하다.&#x20;

코틀린 DSL도 온전히 컴파일 시점에 타입이 정해진다. 따라서 정적 타입 지정 언어의 장점을 누릴 수 있다.



## DSL 개념&#x20;

범용 프로그래밍 언어와 반대로 특정 영역에 초점을 맞춘 언어가 있다. 가장 익숙한 DSL은 SQL과 정규식이다. 이런 SSL은 스스로 제공하는 기능을 제한해 특정 영역에 효율적으로 목표를 달성한다.

범용 프로그래밍언어은 명력적이고 DSL은 선언적이다. 명령적 언어는 어떤 기능을 완수하기 위해 필요한 각 단계를 순서대로 정확히 기술하는 반면, 선언적 언어는 원하는 결과를 기술만 하고 그 안의 과정은 엔진에 맡긴다.&#x20;

DSL의 단점은 DSL을 범용 언어로 만든 애플리케이션과 조합하기 어렵다는 점이다. DSL은 자체 문법이 있기 때문에 다른 언어의 프로그램안에 포함시킬 수 없다. 이런 문제를 해결하면서 DSL의 다른 이점을 살리는 방법으로 내부 DSL이라는 개념이 유명해지 고 있다.&#x20;



## 내부 DSL

독립적인 문법 구조와 달리 내부 DSL은 범용 언어로 작성된 프로그램 일부며, 범용 언어와 동일한 문법을 사용한다.&#x20;

따라서 내부 DSL은 다른 언어가 아니라 DSL의 핵심 장점을 유지하면서 주 언어를 특별한 방법으로 사용하는 것이다.&#x20;

외부 DSL는 SQL를 사용하고 내부 DSL는 데이터베이스 프레임워크 exposed를 사용해보자. 목표는 가장 많은 고객이 살고 있는 나라를 알아내는 것이다.

```sql
SELECT Country.name, COUNT(Customer.id)
FROM Country
JOIN Customer ON Country.id = Customer.country_id
GROUP BY Country.name
ORDER BY COUNT(Customer.id) DESC 
LIMIT 1
```

```kotlin
// exposed
(Country join Customer)
.slice(Contry.name, Count(Customer.id))
.selectAll()
.groupBy(Country.name)
.orderBy(Count(Customer.id),isAsc = false) 
```

두 번째 코드는 코틀린 언어를 사용하지만 내부적으로는 SQL 쿼리가 동작한다. 코드는 어떤 로직을 달성하기 위해 만들지만, 범용 언어를 사용하낟.&#x20;



## DSL의 구조&#x20;

기본 API는 함수 호출과 다른 호출 사이에 아무 맥락이 존재하지 않는다. 반대로 DSL의 메서드 호출은 람다를 중첩시키거나 메서드 호출을 연쇄하는 방식으로 구조를 만든다.&#x20;

DSL은 여러 함수 호출을 조합해서 연산을 만들며, 타입 검사기는 여러 함수 호출이 바르게 조합됐는지 검사한다.&#x20;

DSL 구조의 장점은 같은 문맥을 함수 호출 시마다 반복하지 않고도 재사용할 수 있다는 점이다.&#x20;

코틀린 DSL gradle 의존관계 예를 보자.&#x20;

```kotlin
dependencies{ // 람다 중첩을 통해 구조를 만든다.
    compile("~")
    compile("~")
}

// 반면 일반 API는 코드 중복이 있다.
project.dependencies.add("compile", "~~")
project.dependencies.add("compile", "~~")
```

메서드 호출 연쇄는 DSL 구조를 만드는 또 다른 방법으로 가독성이 더 좋다. 아래는 코틀린 테스트 예이다.

```kotlin
str should startWith("kot") // 메서드 호출을 연쇄시켜 구조를 만듬

// 반면 일반 API는 가독성이 비교적 떨어진다
assertTrue(str.startsWith("kot"))
```



## 수신객체 지정 DSL 사용&#x20;

수신 객체 지정 람다는 구조화된 API를 만들 때 도움이 되는 강력한 기능이다.&#x20;

### 수신 객체 지정 람다와 확장 함수 타입

전에 보았던 buildString을 통해 코틀린이 수신 객체 지정 람다를 어떻게 구현하는지 보자.&#x20;

```kotlin
fun buildString(builderAction: (StringBuilder) -> Unit){
    val sb = StringBuilder()
    builderAction(sb)
    return sb.toString()
}

// 실제 사용 
val s = buildString{
    it.append("Hello")
    it.append("World")    
}
```

그런데 기존에 알던 buildString과는 다르게 it을 계속 붙여야 한다. 이럴 땐 수신 객체 지정 람다로 바꿔야 한다.&#x20;

앞의 buildString을 수신 객체 지정 람다를 사용해 다시 정의해보자.

```kotlin
fun buildString(builderAction: StringBuilder.() -> Unit){ // 수신 객체가 있는 함수타입의 파라미터선언 
    val sb = StringBuilder()
    sb.builderAction() // 인스턴스를 람다의 수신 객체로 넘겨줌
    return sb.toString()
}

val s = buildString{
    this.append("Hello")
    append("Wrold")
}
```

파라미터 타입을 선언할 때 일반 함수 타입이아니라 확장 함수 타입을 사용했다. 확장 함수 타입 선언은 람다의 파라미터 목록에 있던 수신 객체 타입을 파라미터 목록을 여는 괄호 앞으로 빼 놓으면서 중간에 마침표를 붙인 형태다.

StringBuilder.() -> Unit으로 바꿧다. 앞에 오는 타입을 수신 객체라고 부르며, 람다에 전달되는 그런 타입의 객체를 수신 객체라고 부른다.&#x20;

왜 확장 함수 타입일까? 우리는 전에 확장 함수를 배웠다. 확장 함수는 그 확장 클래스 내부에 정의하지 않았지만 마치 내부에서 정의한 것을 호출하듯이 사용할 수 있엇다. 또, 수신 객체를 지정해야 했고 함수 본문 안에서는 그 수신 객체를 특별한 수식자 없이도 사용 가능했다. 똑같이 buildString 메서드는 StringBuilder에 정의된 메서드가 아니지만 StringBuilder 인스턴스인 sb는 확장 함수 호출과 같은 동작을 한다.&#x20;

표준 라이브러리의 실제 buildString은 apply 함수를 이용해 한줄로 구현한다.

```kotlin
fun buildString(builderAction: StringBuilder.() -> Unit): String. = 
StringBuilder().apply(builderAction).toString()
```

apply 함수는 자신의 수신객체를 람다나 함수의 묵시적 수신 객체로 사용한다. (T.() -> Unit)

```kotlin
inline fun <T> T.apply(block: T.() -> Unit) : T{
    block()
    return this 
}

// StringBuilder에 적용하면 
builderAction: StringBuilder.() -> Unit {
    스트링 빌더 인스턴스(sb).builderAction()
}
```

with 함수는 다음과 같다. 람다를 호출해 얻은 결과를 리턴하고 있다.&#x20;

```kotlin
inline fun <T,R> with(receiver:T, block: T.() -> R) : R = receiver.block()
```

기본적으로 apply와 with 모두 자신이 제공받은 수신 객체로 확장 함수 타입의 람다를 호출한다.&#x20;

결과를 받아서 슬 필요가 없다면 이 두함수를 서로 바꿔 쓸 수 있다.&#x20;

```kotlin
val map = mutableMapOf(1 to "one")
map.apply { this[2] = "two"}
with(map) {this[3] = "three"}
```



## 수신 객체 지정람다를 HTML 빌더 안에서 사용&#x20;

코틀린 HTML builder는 안전하면서도 편하다. 다음 코드를 보자.&#x20;

```kotlin
fun createSimpleTable() = createHTML().
table{
    tr{
        td {+"Cell"}
    }
}
```

이 코드는 모두 코틀린 코드이고 특별한 언어가 아니다. 모두 평범한 함수다.&#x20;

여기서 관심을 가질 만한 것은 각 수신 객체 지정 람다가 이름 결정 규칙을 바꾼다는 점이다. table 함수에 넘겨진 람다에서는 tr 함수를 사용해 \<tr> html 태그를 만든다. 하지만 그 람다 밖에서는 tr이라는 이름의 함수를 찾을 수 없다.&#x20;

각 블록의 이름 결정 규칙은 각 람다의 수신 객체에 의해 결정 된다. td 함수는 tr 안에서만 호출 가능하다.

다음은 HTML Builder를 위한 태그 클래스 정의다.

```kotlin
open class Tag

class Table: Tag{
    fun tr(init: TR.() -> Unit){
        ~
    }
}
class TR: Tag{
    fun td(init: TD.() -> Unit){
        ~
    }
}
class TD: Tag
```

각 클래스에는 내부에 들어갈 수 있는 태그를 생성하는 메서드가 들어가 있다. 여기서 어떤 일이 벌어지는지 더 분명히 보기 위해 모든 수신 객체를 명시하면서 다시 써보자.

```kotlin
fun createSimpleTable() = createHTML().
table{
    (this@table).tr{ //Table
        (this@TR).td { // TR
        +"Cell"
        }
    }
}
```

만약 수신 객체 지정람다가 아닌 일반 람다를 사용하면 일일히 it을 붙이거나 적절한 파라미터를 정의해 사용해야 할 것이다.&#x20;

수신 객체 지정 람다가 다른 수신 객체 지정 람다 안에 들어가면 내부 람다에서 외부 람다에 정의된 수신 객체를 사용할 수 있다. 예를 들어 td 함수의 인자인 람다 안에서는 세가지 수신 객체(table, tr, td)를 사용할 수 있다.&#x20;

이제 html이 어떻게 생성 되는건지 살펴보자.&#x20;

```kotlin
fun table(init: TABLE.() -> Unit) = TABLE().apply(init)
```

table 함수는 TABLE 태그의 새 인스턴스를 만들고 그 인스턴스를 초기화하고 반환한다. 위에서 봤던 createTable에서 table 함수에 전달된 람다에는 tr 함수 호출이 들어있다. 이 table 호출에서 모든 부분을 명시하면 `table(init = {this.tr{this.td{~}})`이다. &#x20;

아래는 HTML 빌더의 전체 구현 코드이다.

```kotlin
open class Tag(val name:String){
    private val children = mutableListOf<Tag>()
    protected fun <T:Tag> doInit(child:T, init:T.()->Unit):T{
        child.init()
        children.add(child)
        return child
    }
    override fun toString() = "<$name>${children.joinToString("")}</$name>"

}


class TABLE:Tag("table"){
    fun tr(init:TR.()->Unit) = doInit(TR(), init)
}

fun table(init:TABLE.()->Unit) = TABLE().apply(init)


class TR:Tag("tr"){
    fun td(init:TD.()->Unit) = doInit(TD(), init)
}

class TD:Tag("td")

fun createTable() = table{
    tr {
        td {
        }
    }
}

=> <table><tr><td></td></tr></table>
```



## 코틀린 빌더

내부 DSL를 사용하면 일반 함수처럼 반복되는 내부 DSL 코드를 함수로 묶어 재사용할 수 있다.&#x20;

부트스트랩의 드롭다운 목록을 추가하는 예제를 코틀린 DSL로 구현해보자.&#x20;

```kotlin
fun buildDropdown() = createHTML().div(classes = "dropdown") {
    button(classes= "btn dropdown-toggle"){
        +"Dropdown"
        span(classes = "caret")
    }
    ul{
        li{a("#") {+"Action"}}
        li{a("#") {+"Another action"}
    }
}
```

div, button은 모두 일반 함수이기 때문에 반복되는 로직을 별도의 함수로 분리하면 코드를 더 읽기 쉽게 할 수 있다. 다음은 함수를 통해 재사용 한 코드이다.

```kotlin
fun dropdownExample() = createHTML().dropdown {
  dropdownButton { +"Dropdown" }
  dropdownMenu {
    item("#", "Action")
    item("#", "Another action")
  }
}
```

item함수를 예를 들어보자. item 함수 첫 번째 파라미터로 href, 두 번째 파라미터로 name을 받고 있다.  함수 내부 코드는 `li {a(href){+name}}`이라는 원소를 새로 추가한다.  이렇게 재사용 가능한 함수로 빼내어 더 가독성 좋고 세부사항을 감추어 유지보수를 쉽게 해줄 수 있다.&#x20;



## invoke 관례를 사용한 더 유연한 블록 중첩&#x20;

invoke 관례를 사용하면 객체를 함수처럼 호출 할 수 있다. 이미 함수 타입의 객체(Function1 등)를 함수처럼 호출하는 경우를 살펴봤다. 마찬가지로 invoke 관례를 사용하면 함수처럼 호출할 수 있는 객체를 만드는 클래스를 정의할 수 있다.&#x20;



## invoke 관례: 함수처럼 호출할 수 있는 객체&#x20;

관례는 특별한 이름이 붙은 함수를 일반 메서드 호출 구문으로 호출하지 않고 더 간단한 다른 구문으로 호출할 수 있게 해주는 기능이다. invoke는 get과 달리 괄호를 사용한다.&#x20;

```kotlin
class Greeter(val greeting: String) {
  operator fun invoke(name: String) {
    println("$greeting, $name!")
  }
}

val bavarianGreeter = Greeter("Servus")
bavarianGreeter("Dmitry")
// Servus, Dmitry!
```

이 코드는 Greeter 안에 invoke 연산자 메서드를 정의한다. 따라서 객체를 함수처럼 호출할 수 있다.&#x20;

invoke 함수는 신기한게 없다. 그냥 어느 연산자 메서드처럼 동작한다. 원하는 대로 파라미터 개수나 타입을 지정할 수 있고 오버로딩도 가능하다.&#x20;



### invoke 관례와 함수형 타입&#x20;

일반적인 람다 호출 방식이 실제로는 invoke 관례를 적용한 것에 지나지 않는다. 인라인 람다를 제외한 모든 람다는 함수형 인터페이스(Function1 등)를 구현하는 클래스로 컴파일 된다. 각 인터페이스 안에는 인자의 개수만큼 파라미터를 받는 invoke 메서드가 있다.&#x20;

이제 복잡한 조건을 사용해 이슈 목록을 걸러내는 클래스를 만들어보자.

```kotlin
data class Issue(val id: String, val project: String, val type: String, val priority: String, val description: String)

class ImportantIssuesPredicate(val project: String): (Issue) -> Boolean { // 부모 타입이 함수 타입이다. 
  override fun invoke(issue: Issue): Boolean { // 함수형 인터페이스를 구현하는 클래스에서 invoke를 실행되는데 이것을 재정의 해 실제 함수 동작을 수정한다.
    return issue.project == project && issue.isImportant()
  }
  
  private fun Issue.isImportant(): Boolean {
    return type == "Bug" &&
            (priority == "Major" || priority == "Critical")
  }
}

val i1 = Issue("IDEA-154446", "IDEA", "Bug", "Major", "Save settings failed")
val i2 = Issue("KT-12183", "Kotlin", "Feature", "Normal", "Intention: convert serveral ~~")

val predicate = ImportantIssuesPredicate("IDEA")
for (issue in listOf(i1, i2).filter(predicate)) { // 술어를 filter에 넘김
  println(issue.id)
}
==> IDEA-154446
```

이 코드에서는 filter 술어가 너무 복잡하기 때문에 한 람다로 표현하기 어렵다. 람다는 함수 타입 인터페이스를 구현하는 클래스로 변환되고,  invoke함수가 실행되기 때문에 invoke메서드를 오버라이드 하는 방식대로 하면, 복잡한 술어와 실제로 사용하는 곳에서 환경을 분리해 관심사 분리가 가능하다.



### DSL의 invoke 관례&#x20;

아래는 다시 그레이들 DSL 예제이다. invoke 메서드를 봐보자.

```kotlin
class DependencyHandler {
  // 일반적인 명령형 API 정의
  fun compile(coordinate: String) {
    println("Added dependency on $coordinate")
  }
  // invoke를 정의해 DSL 스타일 API를 제공한다
  operator fun invoke(body: DependencyHandler.() -> Unit) {
    body()
  }
}
val dependencies = DependencyHandler()
dependencies.compile("org.jetbrains.kotlin:kotlin-stdlib:1.0.0")
// 결과: Added dependecy on org.jetbrains.kotlin:kotlin-stdlib:1.0.0
dependencies {
  compile("org.jetbrains.kotlin:kotlin-stdlib:1.0.0")
}
// 결과: Added dependecy on org.jetbrains.kotlin:kotlin-stdlib:1.0.0

```

invoke 메서드는 수신 객체 지정람다를 파라미터로 받는데, 람다의 수신 객체는 다시 DependencyHandler이다. 따라서 람다 안에서 compile을 바로 호출할 수 있다.&#x20;



## 실전 코틀린 DSL&#x20;

### 중위 호출 연쇄: 테스트 프레임워크의 should

코틀린테스트 DSL에서 중위 호출을 어떻게 사용하는지 살펴보자.&#x20;

```kotlin
object start

infix fun String.should(x: start): StartWrapper = StartWrapper(this)

class StartWrapper(val value: String) {
  infix fun with(prefix: String) = if (!value.startsWith(prefix)) {
    throw AssertionError("String $value does not start with $prefix")
  } else {
    Unit
  }
}

```

위의 코드로 인해 `"kotlin" should start with "kot"` 이런 식으로 assert가 가능하다.&#x20;

전혀 코틀린 코드가 아닌 것 같은데 중위 호출을 일반 메서드로 생각해보면 다음과 같다. `"kotlin".should(start).with("kot")`

사실 should는 start라는 타입을 파라미터 타입으로 할 필요가 없다. object는 싱글턴으로 인스턴스가 하나 밖에 없기 때문에 굳이 그 객체를 인자로 넘기지 않아도 바로 접근할 수 있기 때문이다. 하지만 이는 should뒤에 start라는 명명을 하기 위해서이기 때문에 타당하다. 이제 should를 호출한 결과로 StartWrapper 인스턴스를 반환하며 StartWrapper 인스턴스는 검사할 문자열을 파라미터로 받는 with함수를 중위 호출 할 수 있다.&#x20;



### 원시타입에 대한 확장 함수 정의 : 날짜 처리&#x20;

```kotlin
val yeseterday = 1.days.ago
val tomorrow = 1.days.fromNow
```

java8 java.time API와 코틀린을 사용하면 위의 API를 구현하는데 단지 몇 줄의 코드면 충분하다.

```kotlin
import java.time.Period
import java.time.LocalDatet

val Int.days: Period // Int 타입의 확장 프로퍼티 
	get() = Period.ofDays(this) 

val Period.ago: LocalDate // Period 타입의 확장 프로퍼티
	get() = LocalDate.now() - this //LocalDate.minus

val Period.fromNow: LocalDate
	get() = LocalDate.now() + this // LocalDate.plus

println(1.days.ago)
== > 2023-02-28

println(1.days.fromNow)
==> 2023-03-02
```



### 멤버 확장 함수: SQL을 위한 내부 SQL&#x20;

클래스 안에서 확장 함수와 확장 프로퍼티는 클래스의 멤버인 동시에 이 클래스를 확장하는 다른 타입의 멤버이기도 한다. 이런 함수나 프로퍼티를 멤버 확장이라 한다.&#x20;

exposed 프레임워크 코드를 예시로 보자.

```kotlin
object Country : Table(){
    val id = integer("id").autoincrement().primaryKey()
}

class Table{
    fun integer(name:String): Column<Int>
    fun <T> Column<T>.primaryKey(): Column<T>
    fun Column<Int>.autoIncrement(): Column<T>
}

```

integer 메서드가 칼럼을 새로 만든다. 그 뒤 autoincrement와 primarykey 메서드를 통해 각 컬럼 속성을 지정하고 있다. 이 때 멤버 확장을 통해 메서드를 연쇄 호출할 수가 있다.

이 두 함수는 Table의 클래스 멤버이다. 따라서 밖에서는 이 함수를 호출 할 수 없다. 메서드가 적용되는 범위를 제한하고 있는 것이다.  확장 함수가 수신 객체 타입을 제한하고 있는 autoIncrement 메서드를 주목할 만하다.&#x20;







