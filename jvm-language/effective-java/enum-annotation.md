---
description: 자바에는 특수한 목적의 참조타입가 있다. 클래스의 일종인 열거 타입과 인터페이스의 일종인 애너테이션을 알아보자.
---

# 열거 타입과 애너테이션



## 34. int 상수 대신 열거 타입을 사용하라

### 정수 상수 열거

자바에서 열거 타입을 지원하기 전에는 정수 상수를 아래처럼 한 묶음으로 선언해 사용했다.

```java
public static final int APPLE_FUIT = 0;
public static final int APPLE_PIPPIN = 1;
```

이 패턴 기법에는 단점이 많다. 모두 int형 이기 때문에 타입 안전을 보장할 방법이 없고 표현력도 좋지 않다.&#x20;

평범한 상수를 나열한 것 뿐이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다. 그래서 상수의 값이 바뀌면 모두 다시 컴파일 해야한다.

정수 대신 문자열 상수를 사용하는 패턴도 있다. 상수의 의미를 출력할 수 있지만, 마찬가지로 모두 String형이기 때문에 오타가 발생해도 컴파일러는 체크하지 못하고 결국 런타임 버그가 발생하게 된다.



### Enum&#x20;

자바의 열거 패턴은 이런 단점을 보완한다.&#x20;

```java
public enum Apple { FUJI, PIPPIN}
public enum Orange {Lavel, TEMPLE}
```

enum의 아이디어는 단순하다. enum 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다. enum은 외부에서 접근할 수 없도록 생성자를 제공하지 않으므로 사실상 final이다. 따라서 클라이언트가 enum을 직접 생성하거나 확장 할 수 없기 때문에 enum은 싱글턴이다.

### 컴파일 타입 안정성

열거 타입은 컴파일러 타입 안전성을 제공한다. 만약 위의 Apple Enum을 매개변수로 받는 메서드가 있다면, 건네 받은 매개변수는 Apple의 인스턴스들 즉, FUJI와 PIPPIN임을 확신할 수 있다. 당연히 다른 타입의 값을 넣으면 컴파일 단계에서 오류가 난다.&#x20;

```java
// apple의 값은 Apple.FUJI 또는 PIPPIN이다. 
void method(Apple apple){
    
}
```

### 메서드나 필드 추가가 가능하다

enum에 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.&#x20;

단순하게 보면 상수의 모임일 뿐이지만, 실제로는 클래스이기 때문이다.&#x20;

아래는 각 행성마다 질량과 반지름이 있고, 이 두 속성을 이용해 표면중력을 계산해 제공 하는 enum이다.&#x20;

열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 필드에 저장하면 된다.&#x20;

```java
public enum Planet {
    MERCURIUS(3.303e+23, 2.4397e6),
    VENUS(4.869e+24, 6.0518e6),
    EARTH(5.976e+24, 6.37814e6),
    MARS(6.421e+23, 3.3972e6),
    JUPITER(1.9e+27, 7.1492e7),
    SATURN(5.688e+26, 6.0268e7),
    URANUS(8.686e+25, 2.5559e7),
    NEPTUNE(1.024e+26, 2.4746e7);
    
    private final double mass;
    private final double radius;
    private final double surfaceGravity;
    
    private static final double G = 6.67300E-11;
    
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }
    
    public double mass() {
        return mass;
    }
    
    public double radius() {
        return radius;
    }
    
    public double surfaceGravity() {
        return surfaceGravity;
    }
    
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}

```

enum의 values 함수는 자신 안에 정의된 상수들의 값을 배열로 담아 반환하는 정적 메서드이다.&#x20;

enum의 toString은 상수 이름을 문자열로 반환한다

```java
for (Planet p : Palnet.values()){
    System.out.print("%s에서의 무게는 %f이다. ", p, p.surfaceWieght(mass));
}
```

### 상수 별 메서드 구현&#x20;

만약 상수 값에 따라 실제 동작까지 변경되는 enum은 아래 처럼 스위치문을 통해 구현할 수 있다.

```java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE,HH;
    
    public double apply(double x, double y) {
        switch (this) {
            case PLUS -> {
                return x + y;
            }
            case MINUS -> {
                return x - y;
            }
            case TIMES -> {
                return x * y;
            }
            case DIVIDE -> {
                return x / y;
            }
        }
        throw new AssertionError("Unknown op: " + this);
    }
}

```

하지만 만약 새로운 상수 추가 후 case문에 추가를 깜빡하면 컴파일은 되지만, 새로 추가된 상수에 대한 동작은 정의되지 않기 때문에 오류가 발생한다.&#x20;

더 나은 방법은 추상 메서드를 선언하고 각 상수가 자신에 맞게 재정의 하는 것이다.&#x20;

```java
public enum Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public abstract double apply(double x, double y);
}

```

### 상수별 메서드 구현에서 같은 동작을 공유하고 싶다면&#x20;

급여명세서에서 쓸 요일을 표현하는 enum이 있다고 하자. 이 enum은 직원의 기본 임금과 그날 일한 시간이 주어지면 일당을 계산해주는 코드이다. 주중에 오버타임이 발생하면 잔업수당, 주말에는 무조건 잔업수당이 주어진다. 먼저 swtich로 구현한 코드를 보자&#x20;

```java
public enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        int overtimePay;
        switch (this) {
            case SATURDAY:
            case SUNDAY:
                overtimePay = basePay / 2;
                break;
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
}

```

간결하지만 휴가와 같은 새로운 값을 enum에 추가면 반드시 case문을 까먹지 말아야 한다.&#x20;

그럼 이제 상수별 메서드 구현으로 급여를 더 안전히 계산해보자.

```java
public enum PayrollDay {
    MONDAY{
        @Override
        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            int overtimePay;
            overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            return basePay + overtimePay;
        }
    }, TUESDAY{
        @Override
        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            int overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            return basePay + overtimePay;
        }
    }
    // ~ 
    }, SATURDAY{
        @Override
        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            int overtimePay = basePay / 2;
            return basePay + overtimePay;
        }
    }, SUNDAY{
        @Override
        int pay(int minutesWorked, int payRate) {
            return 0;
        }
    };

    private static final int MINS_PER_SHIFT = 8 * 60;

    abstract int pay(int minutesWorked, int payRate);
}

```

코드를 재사용하기가 힘들기 때문에 수목금 코드는 생략했는데도 코드가 장황해져 가독성이 크게 떨어진다.&#x20;

이럴 땐 전략 열거 타입 패턴을 사용하자.

```java
enum PayrollDay {
    MONDAY(PayType.WEEKDAY), TUESDAY(PayType.WEEKDAY), WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY), FRIDAY(PayType.WEEKDAY), SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);
    
    private final PayType payType;
    
    PayrollDay(PayType payType) {
        this.payType = payType;
    }
    
    double pay(double hoursWorked, double payRate) {
        return payType.pay(hoursWorked, payRate);
    }
    
    private enum PayType {
        WEEKDAY {
            @Override
            double overtimePay(double hours, double payRate) {
                return hours <= HOURS_PER_SHIFT ? 0 : (hours - HOURS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            @Override
            double overtimePay(double hours, double payRate) {
                return hours * payRate / 2;
            }
        };
        
        private static final int HOURS_PER_SHIFT = 8;
        
        abstract double overtimePay(double hours, double payRate);
        
        double pay(double hoursWorked, double payRate) {
            double basePay = hoursWorked * payRate;
            return basePay + overtimePay(hoursWorked, payRate);
        }
    }
}

PayrollDay.FRIDAY.pay(1, 1);
```

잔업수당 계산을  private enum(PayType)으로 생성해 주중과 주말이라는 전략을 설정했고,  PayrollDay enum은 잔업수당 계산 전략을 택하는 방식이다.이 방식은 switch보다는 복잡해졌지만 더 유연하고 안전하다.



### 사용이유&#x20;

필요한 원소를 컴파일 타임에 알 수 있는 상수 집합이라면 항상 열거 타입 enum을 사용하자. 메뉴 아이템, 연산 코드 등 허용하는 값 모두를 컴파일 타임에 미리 알고 있을 때도 쓸 수 있다.&#x20;

####

#### 팁&#x20;

* enum이 그 패키지에서만 유요하다면 접근 제한자 범위를 낮춰 구현하도록 하자.&#x20;
* 널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만들자.



## 35. ordinal 메서드 대신 필드를 사용하자&#x20;

enum은 해당 상수가 몇 번째 위치인지를 반환하는 ordinal를 제공하지만, 이 메서드는 유지보수가 힘든 메서드이다.&#x20;

1. 이미 사용 중인 정수와 값이 같은 상수는 추가할 수 없다.
2. 중간에 값을 비워둘 수 없다.
3. 타입 안전하지 않다.

orinal은 사용하지 말고 필드에 저장하도록 하자&#x20;

```java
public enum Ensemble{
    SOLO(1);
    private final int numberOfMusicians;
}
```



## 36. 비트 필드 대신 EnumSet을 사용하라&#x20;

EnumSet 클래스가 비트 필드 수준의 명료함과 성능을 제공하고 열거 타입의 장점을 선사한다.

Enumset의 내부는 비트 벡터로 구현되어 있다. 대부분의 경우에 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다. 또 다른 난해한 작업은 EnumSet이 처리해준다.&#x20;

```java
public class Text{
    public enum Style {BOLD, ITALIC, UNDERLINE}
        
    public void applyStyles(Set<Style styles){} // EnumSet을 넘기는게 가장 좋음

}
text.applyStyles(EnumSet.of(Style.BOLD, STYLE.ITALIC));
```



## 37. ordinal 인데싱 대신 EnumMap을 사용하라&#x20;

배열이나 리스트에서 원소를 꺼낼 때 ordinal를 통해 인덱싱하지 말자. 35에서 봤듯이 유지보수에도 좋지 않고 타입 안전하지 않다. 이럴 땐 EnumMap을 사용하자. EnumMap은 enum타입을 키로 갖는 Map이다. 성능도 우수하고 리스트의 인덱싱 과정에서 오류가 날 가능성도 원천봉쇄된다.&#x20;

```java
Map<EnumClass, Set<Clazz>> enumMap = new EnumMap<>(EnumClass.class);
enumMap.put(EnumClass.Example, clazz1);
enumMap.put(EnumClass.Example, clazz2);

```



## 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

enum을 확장해서 사용하는 건 대부분 좋지 않다. 확장한 타입의 원소는 기반 타입의 원소로 취급되지만, 그 반대는 성립하지 않는다. 기반 타입과 확장 타입 요소를 선회하는 방법도 마땅치가 않으며, 확장성을 높이려면 설계와 구현이 복잡해 진다. \
그럼에도 필요하다면 interface를 이용하자.&#x20;

```java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation{
    PLUS("+"){
        double apply(double x, double y){
            return x + y 
        }
    }
}

public enum ExtendedOperation implements Operation{
    EXP("^"){
        double apply(double x, double y){
            return Math.pow(x,y); 
        }
    }
    
}
```





## 39. 명명 패턴보다 애너테이션을 사용하라&#x20;

### 명명 패턴&#x20;

명명 패턴은 변수나 함수의 이름을 정해진 규칙대로 작성하는 패턴을 뜻한다. 전통적으로 도구나 프레임워크가 특별히 다뤄야하는 곳에는 명명 패턴을 사용했다. 예를들어, JUnit은 버전 3까지 테스트 메서드 일므을 test로 시작하게끔 했다고 한다.&#x20;



### 명명 패턴의 단점&#x20;

1. 오타가 나면 안된다.
2. 이름만 체크하기 때문에 올바른 프로그램 요소인지는 알 수 없다.
3. 프로그램 요소를 매개변수로 전달한 방법이 마땅히 없다.&#x20;



### 애너테이션&#x20;

애너테이션은 위의 문제를 해결해준다. @Test 애너테이션을 직접 작성하면서 이해해보자.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}

public class Sample {
    @Test public static void m1(){
        
    }
    
    public static void m2(){
        
    }
}


```

* 메타 애너테이션: 애너테이션에 설정하는 애너테이션
* @Retention: 애너테이션이 유지되는 범위를 설정한다.
* @Target: 애너테이션이 적용될 대상을 설정한다.
* 마커 애너테이션: 매개변수 없이 단순히 대상에 마킹하는 애너테이션



**애너테이션은 그 대상에 직접적인 영향을 주지는 않는다. 다만, 이 애너테이션에 관심 있는 기능에게 정보를 줘 추가적인 처리를 하도록 도와준다.** &#x20;

다음의 RunTest가 그런 도구의 예이다. RunTest는 Sample Class의 메서드 중 `isAnnotationPresent`메서드를 통해 Test 애너테이션이 있는 대상에만 관심을 가지고 있다.&#x20;

```java
public class RunTests {
    public static void main(String[] args) throws Exception{
        int tests = 0;
        int passed = 0;
        Class testClass = Class.forName("Sample");
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) { // Test 애너테이션이 있ㄴ느지 
                tests++;
                try {
                    m.invoke(null); 
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    // 메서드이 예외를 던지면 여기서 
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " failed: " + exc);
                } catch (Exception exc) {
                    // 애너테이션을 잘못 사용했을 때
                    System.out.println("Invalid @Test: " + m);
                }
            }
        }
        
    }
}

```



이제 특정 예외를 던져야 성공하는 테스트를 지원해보자. 그러려면 새로운 애너테이션 타입이 필요하다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value(); // 이제 매개변수를 받음
}
```

이 애너테이션은 모든 예외 타입을 수용한다는 매개변수 타입을 가지고 있다. 다음은 실제 사용 예시이다.

```java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class) // 클래스 리터럴이 매개변수의 값으로 사용 됨
    public static void m1(){
        int i = 0;
        i = i / i;
    }
    
    @ExceptionTest(ArithmeticException.class) // 클래스 리터럴이 매개변수의 값으로 사용 됨
    public static void m2(){
        int[] a = new int[0];
        int i = a[1];
    }
    
    @ExceptionTest(ArithmeticException.class) // 클래스 리터럴이 매개변수의 값으로 사용 됨
    public static void m3(){
        
    }
}

```

이제 이 애너테이션을 다루도록 테스트 도구 RunTests를 수정해보자.&#x20;

```java
 if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " failed: " + exc);
                    Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
                    if (excType.isInstance(exc)) {
                        passed++;
                    } else {
                        System.out.println("Expected: " + excType.getName() + ", got: " + exc);
                    }
                } catch (Exception exc) {
                    System.out.println("Invalid @Test: " + m);
                }
            }
```

`m.getAnnotation(ExceptionTest.class).value();`  애너테이션의 매개변수인 ArithmeticException를 가져 온 뒤 실제 발생한 오류와 같은지 체크하도록 수정되었다.



이제 예외를 여러 개 명시하고 그중 하나가 발생하면 성공하게 만들어보자. 여러 매개변수를 받으려면 배열로 수정하면 된다.&#x20;

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable>[] value(); // 이제 매개변수를 여러개 받음
}
```

배열 매개변수를 받는 애너테이션용 문법은 유연하기 때문에 기존 @ExceptionTest들을 수정 없이 사용 가능하다.&#x20;

원소가 여럿인 배열로 지정할 때는 중괄호로 감싸고 쉼표로 구분해주면 된다.

```java
    @ExceptionTest({IndexOutOfBoundsException.class, NullPointerException.class}) // 클래스 리터럴이 매개변수의 값으로 사용 됨
    public static void m4(){
        
    }
```



자바8부터 @repeatable 메타애너테이션을 통해 여러 개의 값을 받는 애너테이션을 선언할 수도 있다.



## 40. @Override 애너테이션을 일관되게 사용하라

상위 클래스의 메서드를 재정의 했다면 @Override 애너테이션을 명시해 실수로 오버로드하는 일이 발생하지 않도록 하자. 구체 클래스가 상위 클래스의 추상 메서드를 재정의한 경우에는 optional이지만 그냥 다는게 좋을 것 같다.



## 41. 정의하려는 것이 타입이면 마커 인터페이스를 사용하라&#x20;

아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가지고 있다는 것을 알려주는 인터페이스를 **마커 인터페이스**라고 한다. Serializable 인터페이스가 좋은 예이다. 이 인터페이스를 구현한 클래스의 인스턴스는 ObjectOutputStream을 통해 쓸 수 있다고, 즉 직렬화가 가능하다는 것을 알려준다.

### 마커 인터페이스가 마커 애너테이션 보다 좋은 점&#x20;

1. 마커 인터페이스는 이를 구현한 클래스의 인스턴스를 구분하는 타입으로 쓸 수 있다.&#x20;

마커 인터페이스도 어엿한 타입이기 때문에, 마커 애너테이션을 사용하면 런타임에야 발견될 오류를 마커 인터페이스라면 컴파일 단계에서 발견할 수 있다.&#x20;

예를 들어 `ObjectOutputStream.writeObject` 메서드는 당연히 인수로 받은 객체가 Serializable을 구현했다고 가정한다. 하지만 실제 인수 타입은 Object이다. 따라서 Serializable 인터페이스를 구현하지 않은 객체를 넘겨도 런타임에서야 문제를 확인할 수 있다.&#x20;



2. 적용 대상을 더 정밀하게 지정할 수 있다.

특정 인터페이스를 구현한 클래스에만 적용하고 싶을 때처럼 마커 인터페이스는 대상을 더 정밀하게 지정할 수 있다.

Set이 일종의 마커 인터페이스라고 볼 수 있다. Set은 Collection의 하위 타입에만 적용 할 수 있으며, Collection이 정의한 메서드 외에는 새로 추가한 것이 업다.

```java
public interface Set<E> extends Collection<E> {
}
```

이런 마커 인터페이스는 객체의 특정 부분을 불변식으로 규정하거나, 그 타입의 인스턴스는 다른 클래스의 특정 메서드가 처리할 수 있다는 사실을 명시하는 용도로 사용할 수 있을 것이다.&#x20;



### 마커 애너테이션이 더 나은 점&#x20;

거대한 애너테이션 시스템의 지원을 받는다. 따라서 애너테이션을 적극 활용하는 프레임워크에서는 마커 애너테이션을 쓰는 쪽이 일관성을 지키는데 더 좋다.&#x20;



### 어떨 때 써야할까?

마킹 된 객체를 매개변수로 받는 메서드를 작성할 일이 있다면 마커 인터페이스를 사용하자. 컴파일 타임에 오류를 발견할 수 있을 것이다. 그렇지 않거나 프레임워크에서 사용한다면 마커 애너테이션이 더 좋다.&#x20;









###

