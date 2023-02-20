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







###

