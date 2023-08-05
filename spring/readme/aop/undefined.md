# 프록시

## Proxy Pattern, Decorator Pattern

AOP의 핵심은 관점(관심)이 다른 부분을 분리해 서로의 코드에 영향을 주지 않는 것이라고 생각한다.&#x20;

핵심 기능의 코드를 건들이지 않으면서 다른 부가 기능을 수행하는 것은 디자인 패턴 중 데코레이터 패턴과 같다. 실제로 스프링 AOP는 Proxy Pattern과 Decorator Pattern을 사용해 AOP의 기능을 구현한다.&#x20;

그런데 한 가지 문제가 있다. 부가 기능을 추가하려면 반드시 대상 클래스(핵심 기능)의 인터페이스를 온전히 구현하는 Proxy 클래스가 필요한 것이다. 만약 대상 클래스가 천개라면 천개의 프록시 클래스를 매번 생성해야 한다.&#x20;



## Reflection을 이용한 동적 프록시 생성

reflection 기술을 이용하면 매번 대상 클래스에 대응되는 프록시 클래스를 직접 생성하지 않고 동적으로 만들 수 있다.  기존 객체의 메서드를 동적으로 변경하는 것도 가능하다. 따라서 대상 클래스의 인터페이스를 구현하는 프록시 객체 또한 만들 수 있다.

스프링은 CGLIB 같은 오픈소스나 자바가 기본으로 제공하는 JDK 동적 프록시 기술을 사용해 동적으로 프록시를 생성한다.

{% content-ref url="../../../language/kotlin/kotlin-in-action/annotation-reflection/reflection.md" %}
[reflection.md](../../../language/kotlin/kotlin-in-action/annotation-reflection/reflection.md)
{% endcontent-ref %}



### JDK 동적 프록시&#x20;

JDK 동적 프록시는 프록시 객체를 매번 생성하지 않고도 동적으로 프록시를 만들 수 있게 해주는 기술이다. 반드시 인터페이스를 가지고 있어야한다.

```kotlin
import java.lang.reflect.InvocationHandler
import java.lang.reflect.Method
import java.lang.reflect.Proxy

interface ExampleInterface {
    fun hello()
}

// 대상 클래스 
class ExampleClass : ExampleInterface {
    override fun hello() {
        println("Hello, world!")
    }
}

class ExampleClass2 : ExampleInterface{
    override fun hello() {
        println("Goodbye, world!")
    }
}

// 프록시 기능을 수행할 Handelr
class ExampleInvocationHandler(private val target: ExampleInterface) : InvocationHandler {
    override fun invoke(proxy: Any, method: Method, args: Array<out Any>?): Any? {
        println("[Before] method call: ${method.name}")
        val result = method.invoke(target, *(args ?: arrayOf()))
        println("[After] method call: ${method.name}")
        return result
    }
}

fun main() {
    val example = ExampleClass()
    val proxy: ExampleInterface = Proxy.newProxyInstance(
        example.javaClass.classLoader, // 대상 클래스의 클래스 정보 
        arrayOf(ExampleInterface::class.java), // 인터페이스 정보
        ExampleInvocationHandler(example) // handler
    ) as ExampleInterface // 프록시 객체가 동적으로 생성됨
    
    val example2 = ExampleClass2()
    val proxy2: ExampleInterface = Proxy.newProxyInstance(
        example2.javaClass.classLoader, // 대상 클래스의 클래스 정보 
        arrayOf(ExampleInterface::class.java), // 인터페이스 정보
        ExampleInvocationHandler(example2) // handler
    ) as ExampleInterface // 프록시 객체가 동적으로 생성됨

    proxy.hello()
    proxy2.hello()
}

```

#### 실행 순서

1. 만들어진 proxy 객체의 hello를 호출한다.
2. proxy 객체는 InvocationHandler의 invok 함수가 호출된다.&#x20;
3. invok 함수 내부 로직이 수행되면서 기존 대상 클래스의 메서드를 호출한다.&#x20;



기존 프록시라면 아래와 같이 실행되지만&#x20;

<img src="../../../.gitbook/assets/file.excalidraw (23).svg" alt="" class="gitbook-drawing">

jdk 동적 프록시는 아래와 같이 실행된다.&#x20;

<img src="../../../.gitbook/assets/file.excalidraw (2).svg" alt="" class="gitbook-drawing">





### CGLIB (Code Generator Library)

바이트코드를 조작해 동적으로 클래스를 생성하는 기술을 제공하는 라이브러리

외부 라이브러리이지만 스프링 내부소스에 포함했기 때문에 따로 설치할 필요는 없다.

cglib는 dynamic proxy와 다르게 인터페이스 없이도 생성할수 있지만, 상속을 사용하기 때문에 final이면 안된다.

cglib는 jdk dynamic proxy의 invocation handler와 다르게 method interceptor라는 이름으로 부가 기능을 수행한다.





