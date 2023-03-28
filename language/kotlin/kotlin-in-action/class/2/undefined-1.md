# 프로퍼티

## 용어정리

### 필드: 멤버변수

```java
public class Java{
    private int level; // filed
}
```

### 프로퍼티: 멤버변수와 접근자를 한데 묶어 프로퍼티라 한다.

```java
public class Java{ 
    // 프로퍼티
    private int level;         
    
    public int getLevel(){
        return this.level;
    }
    public void setLevel(int level){
        this.level = level;
    }
}
```

### 코틀린은 자동으로 접근자 메서드를 생성해주기 때문에 필드 대신 프로퍼티로 부른다.&#x20;

```kotlin
// Kotlin
public class Kotlin(var level : Int)


// Java Decompile
public final class Kotlin {
   private int level;

   public final int getLevel() {
      return this.level;
   }

   public final void setLevel(int var1) {
      this.level = var1;
   }

   public Kotlin(int level) {
      this.level = level;
   }
}

```



## 뒷받침 필드 (Backing Field)

뒷받침 필드는 코틀린이 실제로 데이터를 담기 위한 자바의 필드이다.&#x20;

```java
public final class Kotlin {
   private int level; // 뒷받침 필드 

   public final int getLevel() {
      return this.level;
   }

   public final void setLevel(int var1) {
      this.level = var1;
   }

   public Kotlin(int level) {
      this.level = level;
   }
}

```

### 뒷받침 필드 생략

코틀린이 반드시 뒷받침 필드를 사용하는 것은 아니다. 필요가 없으면 알아서 생략해준다.&#x20;

아래 코드는 `nickname`을 커스텀 getter를 통해 구현하고 있다. nickname 값은 email을 토대로 계산하기 때문에 따로 nickname 값을 저장할 필요가 없으므로, 뒷받침 필드가 필요 없다.&#x20;

```kotlin
// Kotlin
class SubscribingUser(val email: String) : User{
    override val nickname
    get() = email.substringBefore('@')
}

// Java nickname 뒷받침 필드가 생략된 것을 볼 수 있다. 
public final class SubscribingUser {
   private final String email;


   @NotNull
   public String getNickname() {
      return StringsKt.substringBefore$default(this.email, '@', (String)null, 2, (Object)null);
   }

   @NotNull
   public final String getEmail() {
      return this.email;
   }
}

```



### field 키워드

field 키워드는 커스텀 접근자에서 직접 뒷받침 필드에 접근할 수 있도록 도와준다.

```kotlin
class Kotlin(var level:Int){
    var name:String = ""
        set(value){
            field = if(value.isNotBlank()) value else "No Name"
        }
        get() = if(field == "No Name") "무명" else field
}

```

만약 필드 없이 프로퍼티에 바로 접근하면 어떻게 될까?&#x20;

<pre class="language-kotlin"><code class="lang-kotlin"><strong>   // Kotlin
</strong><strong>   class Kotlin(var level:Int){
</strong>    var name:String = ""
        set(value){
            name = if(value.isNotBlank()) value else "No Name"
        }
        get() = if(field == "No Name") "무명" else name
   }   
</code></pre>

디컴파일된 자바 코드를 보면 getName은 문제 없어보이지만 setName은 자신을 재귀호출로 불러오고 있다. 프로퍼티에 직접 값을 넣는게 아니라 setter 함수를 사용해 값을 넣어주기 때문이다.&#x20;

```java
   // Java
   public final void setName(@NotNull String value) {
      this.setName(!StringsKt.isBlank(value) ? value : "No Name");
   }
   
   public final String getName() {
      return Intrinsics.areEqual(this.name, "No Name") ? "무명" : this.name;
   }
```

[https://phantasmicmeans.tistory.com/entry/8-Kotlin-Backing-Field](https://phantasmicmeans.tistory.com/entry/8-Kotlin-Backing-Field)





## 인터페이스에 선언된 프로퍼티 구현&#x20;

* 코틀린은 인터페이스에 프로퍼티를 선언할 수 있다.&#x20;

```kotlin
interface User{
    val nickname: String
}
```

* 구현 클래스들은 반드시 해당 프로퍼티를 제공해야한다는 것을 뜻한다. 물론 뒷받침하는 필드나 게터 등의 실제 정보는 가지고 있지 않다.&#x20;

### 구현해보기&#x20;

* 주 생성자

주 생성자의 간결한 구문을 통해 구현할 수 있다. User의 추상프로퍼티 nickname을 구현하고 있으므로 override 키워드를 반드시 표시해야한다.

```kotlin
class PrivateUser(override val nickname:String) : User
```

* 커스텀 게터

```kotlin
class SubscribingUser(val email: String) : User{
    override val nickname
    get() = email.substringBefore('@')
}
```

* 프로퍼티 초기화

```kotlin
class FacebookUser(val accountId:Int): User{
    override val nickname = getFacebookName(accountId)
}
```



### 게터와 세터가 있는 프로퍼티

* 인터페이스에 게터와 세터가 있는 프로퍼티를 선언할 수도 있다.&#x20;
* 물론 그 게터와 세터는 뒷받침 필드를 참조할 수 없다.  인터페이스에는 상태를 저장할 수 없기 때문에 뒷받침 필드 자체가 없기 때문이다.&#x20;

```kotlin
interface User{
    val email:String // 하위 클래스는 email은 반드시 오버라이드해야 한다. 
    val nickanme:String // 반면 nickname은 오버라이드 하지 않아도 된다. 
    get() = email.substringBefore('@')
}
```

### 접근자의 가시성 변경

원하다면 get,set 앞에 가시성 변경자를 추가할 수 있다.

```kotlin
class LengthCounter{
    var counter: Int = 0
    private set // 외부에서 이 프로퍼티의 값을 바꿀 수 없도록 한다.
    
    fun addWorld(word:String){
        counter += world.length 
    }
}
```
