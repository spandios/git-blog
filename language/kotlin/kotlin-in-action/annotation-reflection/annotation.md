# Annotation

코틀린 애노테이션도 자바 애노테이션과 개념은 같다. 메타데이터를 선언에 추가하면 애노테이션을 처리하는 도구가 컴파일 또는 런타임 시점에 적절한 처리를 해준다.&#x20;



## 애노테이션 적용&#x20;

애노테이션을 적용하려면 적용하려는 대상 앞에 애노테이션을 붙이면 된다.&#x20;

```kotlin
// ReplaceWith을 통해 새로운 API를 제시할 수 있음
@Deprecated("Use feedAll", ReplaceWith("feeds(animals)")) 
```

### 애노테이션 인자&#x20;

원자 타입의 값, 문자열, enum, 클래스 참조, 다른 애노테이션 클래스, 이런 요소들로 이뤄진 배열이 들어갈 수 있다.

애노테이션 인자를 지정하는 문법은 자바와 조금 다르다.&#x20;

* 클래스를 애노테이션 인자로 지정할 때는 ::class를 클래스 이름 뒤에 넣어야 함. `@MyAnnotation(MyClass::class) (`자바: MyClass.class)
*   다른 애노테이션을 인자로 지정할 때는 인자로 들어가는 애노테이션에 @를 붙이지 않는다.  @Deprecated의 ReplaceWith은 사실 애노테이션이다. &#x20;

    ```kotlin
    public annotation class ReplaceWith(val expression: String, vararg val imports: String)
    ```
* 배열을 인자로 지정하려면 @RequestMapping(path=arrayOf("/foo", "/bar")) 처럼 arrayOf함수를 사용한다. 자바에서 선언한 애노테이션 클래스를 사용한다면 value라는 이름의 파라미터가 필요에 따라 자동으로 가변 길이 인자로 변환된다. `@JavaAnnotationWithArrayValue("abc","foo","bar")`



애노테이션 인자는 컴파일 시점에 알 수 있어야 한다. 프로퍼티를 인자로 사용하려면 그 앞에 **const** 변경자를 붙여야 한다. 컴파일러는 const가 붙은 프로퍼티를 컴파일 시점 상수로 취급한다. const가 붙은 프로퍼티는 파일의 맨 위나 object 안에 선언되어야 하며, 원시타입이나 String으로 초기화해야 한다.&#x20;

```kotlin
const val TEST_TIMEOUT = 100L
@Test(timeout = TEST_TIMEOUT) fun testMethod(){ ... }
```



## 애노테이션 대상

코틀린 소스코드에서 한 선언을 컴파일한 결과가 **여러 자바 선언과 대응**하는 경우가 많다. 예를 들어 코틀린 프로퍼티는 기본적으로 자바 필드와 게터, 세터 선언과 대응한다. 따라서 애노테이션을 붙일 때 이런 요소 중 어떤 요소에 애노테이션을 붙일지 표시할 필요가 있다.&#x20;

**사용 지점 대상** 선언으로 애노테이션을 붙일 요소를 정할 수 있다.&#x20;

<img src="../../../../.gitbook/assets/file.excalidraw (9).svg" alt="" class="gitbook-drawing">

Junit의 예로 사용 지점 대상의 기능을 이해 해보자.&#x20;

Junit에서는 각 테스트 메서드 앞에 그 메서드를 실행하기 위한 규칙을 지정할 수 있다. 예를 들어 TemporaryFolder라는 규칙을 사용하면 메서드가 끝나면 삭제될 임시 파일과 폴더를 만들 수 있다.&#x20;

규칙을 지정하려면 공개 필드나 메서드 앞에 @Rule을 붙이면 된다. 하지만 코틀린 테스트 클래스의 folder라는 프로터이 앞에 @Rule을 적용하면 프로퍼티가 공개 필드여야 한다고 오류가 발생한다. 코틀린 필드는 기본적으로 비공개이기 때문이다. 따라서 @Rule 애노테이션을 정확한 대상에 적용하려면 다음과 같이 **@get:Rule**을 사용해야 한다.&#x20;

```kotlin
class HasTempFolder{
    @get:Rule
    val folder = TemporaryFolder() 
    
    @Test
    fun testUsingTempFolder(){
        val createFile = folder.newFile("myFile.txt")
    }
}
```

자바에 선언된 애너테이션을 사용해 프로퍼티에 애노테이션을 붙이는 경우 기본적으로 프로퍼티의 필드에 그 애노테이션이 붙는다. 하지만 코틀린으로 애노테이션을 선언하면 프로퍼티에 직접 적용할 수 있는 애노테이션을 만들 수 있다.&#x20;

**사용 지점 대상을 지정할 때 지원하는 대상 목록은 다음 과 같다.**&#x20;

* property: 프로퍼티 전체, 자바 에서 선언된 애노테이션에서는 이 사용 지점 대상 사용 할 수 없음
* filed: 프로퍼티에 생성되는 뒷받침하는 필드&#x20;
* get: 게터&#x20;
* set: 세터&#x20;
* receiver: 확장 함수나 프로퍼티의 수신 객체 파라미터
* param: 생성자 파라미터
* setparam: 세터 파라미터
* delegate: 위임 프로퍼티의 위임 인스턴스를 담아둔 필드
* file: 파일 안에 선언된 최상위 함수와 프로퍼티를 담아두는 클래스&#x20;

코틀린은 애노테이션 인자로 클래스나 함수 선언이나 타입 외에 임의의 식을 허용한다. &#x20;

```kotlin
@Suppress("UNCHECKED_CAST")
val strings = list as List<String> // 임의의 식에 애노테이션을 적용함 
```



{% hint style="info" %}
자바 API를 애노테이션으로 제어하기&#x20;

코틀린은 코틀린으로 서언한 내용을 자바 바이트코드로 컴파일하는 방법과 코틀린 선언을 자바에 노출하는 방법을 제어하기 위한 애노테이션을 제공한다.&#x20;

* @JvmName: 코틀린 선언이 만들어내는 자바 필드나 메서드 이름을 변경한다.
* @JvmStatic: 메서드, 객체 선언, 동반 객체에 적용하면 그 요소가 자바 정적 메서드로 노출된다.
* @JvmOverloads: 디폴트 파라미터 값이 있는 함수에 대해 컴파일러가 자동으로 오버로딩한 함수를 생성한다.
* @JvmField: 프로퍼티에 사용하면 게터나 세터가 없는 공개된 자바 필드로 프로퍼티를 노출시킨다.
{% endhint %}



## 애노테이션을 활용한 JSON 직렬화 제어&#x20;

* 직렬화: 객체를 텍스트나 이진 형식으로 반환
* 역직렬화: 텍스트나 이진 형식을 원래의 객체로 변환&#x20;

이제부터 제이키드라는 코틀린으로 작성된 JSON 직렬화 라이브러리를 만들어보자. 먼저 Person 클래스를 예시로 직렬화 하고 역직렬화를 살펴보자. 역직렬화 할 때 타입 인자로 클래스를 명시했다.

```kotlin
data class Person(val name:String, val age: Int)

// 직렬화
val person = Person("Alice", 29)
serialize(person) => {"name":"Alice","age":29} // json 형식으로 직렬화 되었다

// 역직렬화
val json = """{"name":"Alice","age":29}"""
val person = deserialize<Person>(json) // 역직렬화
```

애노테이션을 방식으로 객체를 직렬화하거나 역직렬화하는 방법을 제어할 수 있다. 객체를 JSON으로 직렬화할 때 제이키드 라이브러리는 기본적으로 모든 프로퍼티를 직렬화하며 프로피티 이름을 키로 사용한다. 애노테이션은 이런 동작을 변경할 수 있다. @JsonExclude와 @JsonName의 애노테이션을 다룰 것이다.&#x20;

* @JsonExclude: 직렬화나 역직렬화 시 이 프로퍼티를 무시할 수 있음&#x20;
* @JsonName: 프로퍼티를 표현하는 키/값 쌍 중에 키를 기본 프로터티 이름 대신 애노테이션이 지정한 이름으로 쓰게 할 수 있음



## 애노테이션 선언&#x20;

이제 JsonExclude 애노테이션을 예로 선언하는 방법을 살펴보자.&#x20;

`annotation class JsonExclude(val name:String)`

&#x20;애노테이션 클래스는 오직 선언이나 식과 관련 있는 메타데이터의 구조를 정의하기 때문에 본문에는 아무 코드도 들어올 수 없다. 파라미터를 정의하려면 주 생성자에 선언하면 되고 val만 사용할 수 있다.&#x20;

아래는 자바 애노테이션의 예이다.&#x20;

```kotlin
public @interface JsonName{
    String value();
}
```

코틀린 애노테이션은 name이라는 프로퍼티를 사용했고 자바는 value 메서드를 썼다.&#x20;

자바에서는 value를 제외한 모든 어트리뷰트에는 이름을 명시해야한다. 반면, 코틀린은 이름 붙인 인자 구문을 사용할 수도 있고 안 할 수도 있다.



## 메타 애노테이션: 애노테이션을 처리하는 방법 제어&#x20;

자바처럼 코틀린 애노테이션 클래스에도 애노테이션을 붙일 수 있다. 이런 애노테이션을 **메타애노테이션**이라고 부른다.

표준 라이브러리에 이미 몇가지 메타 애노테이션이 있으며, 그 애노테이션들은 컴파일러가 애노테이션을 처리하는 방법을 제어한다. 그 중 가장 자주 쓰이는 메타애노테이션은 @Target일 것이다.&#x20;

@Target 메타애노테이션은 애노테이션을 적용할 수 있는 요소의 유형을 지정한다. 지정하지 않으면 모든 선언에 적용할 수 있게 된다. 애노테이션이 붙을 수 있는 대상이 정의된 AnnotationTarget란 enum이 있다. 이 enum안에는 클래스, 파일, 프로퍼티, 프로퍼티 접근자, 타입, 식 등에 대한 이넘 정의가 들어가 있다.&#x20;

<details>

<summary>enum class AnnotationTarget </summary>

```
public enum class AnnotationTarget {
    /** Class, interface or object, annotation class is also included */
    CLASS,
    /** Annotation class only */
    ANNOTATION_CLASS,
    /** Generic type parameter */
    TYPE_PARAMETER,
    /** Property */
    PROPERTY,
    /** Field, including property's backing field */
    FIELD,
    /** Local variable */
    LOCAL_VARIABLE,
    /** Value parameter of a function or a constructor */
    VALUE_PARAMETER,
    /** Constructor only (primary or secondary) */
    CONSTRUCTOR,
    /** Function (constructors are not included) */
    FUNCTION,
    /** Property getter only */
    PROPERTY_GETTER,
    /** Property setter only */
    PROPERTY_SETTER,
    /** Type usage */
    TYPE,
    /** Any expression */
    EXPRESSION,
    /** File */
    FILE,
    /** Type alias */
    @SinceKotlin("1.1")
    TYPEALIAS
}
```

</details>

만약 메타애노테이션을 직접 만들어야 한다면 @Target에 ANNOTATION\_CLASS로 지정하면 된다.&#x20;

{% hint style="info" %}
@Retention&#x20;

자바 @Retention은 정의 중인 애노테이션 클래스를 유지하는 정책을 아래에 선택해 정할 수 있다.&#x20;

* 클래스를 소스 수준에서만 유지&#x20;
* .class 파일에 저장&#x20;
* 실행 시점에 리플렉션을 사용해 접근 가능&#x20;

자바는 기본적으로 .class파일에 저장만하고 런타임에는 유지하지 않는다.&#x20;

하지만 대부분의 애노테이션은 런타임에 유지해야하기 때문에 코틀린은 기본 정책을 런타임으로 지정한다.&#x20;
{% endhint %}



## 애노테이션 파라미터로 클래스 사용&#x20;

정적인 데이터를 대상으로 하는 위의 방식과는 다르게 어떤 클래스를 선언 메타데이터로 참조할 수 있는 기능이 필요한다면 어떻게 해야할까? 이때는 클래스 참조를 파라미터로 하는 애노테이션 클래스를 선언하면 된다.&#x20;

제이키드 라이브러리에 있는 @DeserializeInterface는 인터페이스 타입인 프로퍼티에 대한 역직렬화를 제어할 때 쓰는 애노테이션이다. 인터페이스의 인스턴스를 직접 만들 수는 없기 때문에, 어떤 클래스르 사용해 인터페이스를 구현할지 지정할 수 있어야 한다. 한번 애노테이션을 정의 해보자.

```kotlin
annotation class DeserializeInterface(val targetClass: KClass<out Any>)
```

KClass는 자바 java.lang.Class 타입과 같은 역활하는 타입이다. 코틀린 클래스에 대한 참조를 저장할 때 사용된다.&#x20;

KClass 타입 파라미터에 out을 안쓴다면 오직 Any만 들어갈 수 밖에 없다. out 키워드를 통해 공변성을 가지게 끔 해야한다. &#x20;

사용의 예는 다음과 같다. `@DesrializeInterface(CompanyImpl::class) val company: Company`

## 애노테이션 파라미터로 제네릭 클래스 받기&#x20;

기본적으로 제이키드는 원시 타입이 아닌 프로퍼티를 중첩된 객체로 직렬화한다. 이런 기본 동작을 변경하고 싶으면 값을 직렬화하는 로직을 직접 제공하면 된다. @CustomSerializer 애노테이션은 커스텀 직렬화 클래스에 대한 참조를 인자로 받는다. 이 직렬화 클래스는 반드시 ValueSerializer 인터페이스를 구현해야만 한다.

```kotlin
interface ValueSerializer<T>{
    fun toJsonValue(value : T) : Any?
    fun fromJsonValue(jsonValue: Any?): T
}
```

@CustomSerializer 애노테이션을 구현해보자. ValueSerializer 인터페이스를 보면 제네릭이라 타입 파라미터가 있다. 따라서 항상 타입인자를 제공해야하는데 이 애노테이션이 어떤 타입에 대해 쓰일지는 전혀 알지 못하기 때문에 스타 프로젝션을 사용할 수 있다.&#x20;

```kotlin
annotation class CustomSerializer {
    val serializerClass: KClass<out ValueSerializer<*>>
}
```

패턴을 기억하자. 클래스를 인자로 받아야 하면 애노테이션 파라미터 타입에 KClass\<out 허용할 클래스 이름>을 쓴다.&#x20;

제네릭 클랙스를 인자로 받아야한다면 KClass\<out 허용할 클래스 이름<\*>>으로 쓴다.&#x20;

