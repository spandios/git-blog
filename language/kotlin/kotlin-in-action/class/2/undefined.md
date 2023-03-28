# 생성자

* 코틀린은 주 생성자와 부 생성자를 구분한다.&#x20;
* 초기화 블록을 통해 초기화 로직을 추가할 수 있다.



## 주 생성자

* class 이름 뒤에 괄호로 둘러싸인 코드를 주 생성자라고 부른다.&#x20;
* 주 생성자는 생성자 파라미터를 지정하고 명시한대로 프로퍼티를 정의하는 목적으로 쓰인다.&#x20;

`class User(val nickname : String)`

* 만약 이런 클래스 선언을 풀어서 명시한다면 다음과 같다.

```kotlin
class User constructor(_nickname:String){
    val nickname:String
    init{
        nickname = _nickname
    }
}
```

* constructor는 생성자로 주 생성자나 부 생성자 정의를 시작할 때 사용된다.&#x20;
* init은 초기화 블록의 시작을 시작한다. 초기화 블록은 클래스가 인스턴스화될 때 실행되는 코드가 들어간다. 주 생성자와 함께 사용하게 되는데, 주 생성자는 어떤 로직을 수행할 수는 없기 때문이다. 필요하다면 여러 초기화 코드를 선언할 수도 있다.
* 만약 주 생성자에 별다른 애노테이션이나 가시성 변경자가 없다면 constructor 키워드를 생략해도 된다.&#x20;

```kotlin
class User(_nickname:String){
    val nickname = _nickname
}
```

* 지금은 클래스 본문안에 val 키워드를 통해 초기화하고 있지만 처음 본 형식처럼 주 생성자에 val 키워드를 추가해 프로퍼티 정의 뿐만아니라 초기화까지 간략히 쓸 수 있다.&#x20;

```kotlin
class User(val nickname:String = "허주영") // 디폴트 값을 넣을 수 있다.
```

* 대부분의 생성자는 단순하다. 파라미터가 없는 클래스도 많고, 단지 생성자 코드 안에서 인자로 받은 값을 프로퍼티에 설정만 하는 것도 많다. 따라서 코틀린은 간결하게 사용할 수 있는 주 생성자 문법을 제공한다.

{% hint style="info" %}
모든 생성자 파라미터에 디폴트 값을 지정하면? 기본 생성자 자동 생성&#x20;

컴파일러가 자동으로 파라미터가 없는 생성자를 작성해주며, 그 생성자는 디폴트 값을 사용해 클래스를 초기화 하게 된다. DI 프레임워크 등의 자바 라이브러리 중에서는 반드시 파라미터가 없는 기본 생성자가 있어야하는 경우가 있는데, 코틀린의 이런 기능 때문에 통합이 쉽다.
{% endhint %}

### 확장할 때 상위 클래스의 주 생성자 호출&#x20;

클래스가 확장하는 상위 클래스가 있다면 반드시 상위 클래스의 주생성자를 호출해야 한다.

```kotlin
open class User(val username:String)
class TwitterUser(nickname): User(nickname)
```



### 주 생성자에 가시성 식별자 부여&#x20;

만약 클래스 외부에서 인스턴스화 하지 않게 막으려면 constructor 키워드에 private 가시성 변경자를 선언하면 된다.

```kotlin
class User private constructor(){}
```



## 부 생성자

* 코틀린에서 부 생성자를 사용할 일은 자바보다 훨씬 적다.&#x20;
* 자바에서 오버로드한 생성자가 필요한 상황은 대부분 코틀린의 디폴트 파라미터 값과 이름 붙인 인자 문법을 사용해 해결 할 수 있기 때문이다.&#x20;
* 부 생성자는 본문안에 construcotr 키워드를 통해 선언하면 된다.

```kotlin
open class View{
    constructor(ctx:Context){
        
    }
    constructor(ctx:Context, attr:AttributeSet){
        
    }
}
```

* 위의 View 클래스를 확장하면서 똑같이 부 생성자를 정의할 수 있다. 이 때 super 키워드를 통해 대응하는 상위 클래스 생성자를 호출한다. 이는 상위 클래스의 부 생성자에게 객체 생성을 위임하는 것이다.

```kotlin
class MyButton: View{
    constructor(ctx:Context) : super(ctx){}
    constructor(ctx:Context, attr:AttributeSet): super(ctx,attr){}
}
```

* 또는 this를 통해 자신이 정의한 다른 생성자에게 객체 생성을 위임할 수 있다.

```kotlin
class MyButton: View{
    constructor(ctx:Context) : this(ctx,MY_STYLE){}
    constructor(ctx:Context, attr:AttributeSet) : super(ctx, attr{}
}
```



