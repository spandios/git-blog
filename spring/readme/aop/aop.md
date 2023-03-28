# 스프링 AOP

## Aspect&#x20;

부가 기능을 모듈로 분리한 개념으로 어떤 기능과 이 기능을 어디에 적용할지 선택하는 기능을 포함하고 있다. 전에 알아본 어드바이저와 개념적으로 같다.&#x20;

Aspect는 관점이라는 뜻으로 이를 사용한 프로그래밍이 관점 지향 프로그래밍이다(Aspect Oriented Programming). AOP는 애플리케이션을 바라보는 관점을 횡단 관심사 관점으로 바라 보는 것이다.

&#x20;

## AOP 적용 방식&#x20;

aop가 실제로 적용되는 방식은 3가지가 있다.

### 1. 컴파일 타임 - 위빙&#x20;

AspectJ라는 AOP 프레임워크는 소스코드를 .class로 컴파일하는 시점에 관여해 적용 대상에 부가 기능 코드 자체를 추가한다. AspectJ를 직접 사용한다.

단점: 특별한 컴파일러가 필요하고 복잡하다.&#x20;

### 2. 클래스 로드 시점&#x20;

jvm 클래스 로더에 컴파일 된 .class를 로드한다. 이 때 .class 파일을 조작해 위빙해 적용 대상에 부가 기능 코드 자체를 추가한다. 이를 로드 타임 위빙이라 한다. AspectJ를 직접 사용한다.

단점: 자바를 실행할 때 특별한 옵션을 통해 클래스 로더 조작기를 지정해야하므로 번거롭고 운영이 어렵다.&#x20;



### 3. 런타임 시점(프록시)

이전까지 알아본 프록시 방법으로 적용하는 방식으로 프록시, Spring Container ,빈 후처리기, DI 등 여러 개념이 필요하지만 AspectJ나 다른 복잡한 옵션 등이 필요 없다.&#x20;

AspectJ는 실제 바이트 코드를 조작하기 때문에 생성자, 필드 값 접근, static 메서드 접근, 메서드 실행 등 적용 가능한 지점의 범위가 넓다. 적용 가능한 지점을 **조인 포인트**라고 한다.&#x20;

하지만 Spring AOP는 Proxy를 사용하기 때문에 Proxy의 한계가 그대로 적용된다. Proxy는 메서드 오버라이딩 개념으로 동작하기 때문에 생성자나 static 메서드, 필드 값 접근에는 조인 포인트로 할 수 없다.  Spring AOP의 조인 포인트는 메서드 실행으로 제한되며 스프링 빈에만 AOP를 적용할 수 있다.&#x20;

AspectJ를 사용하면 더 복잡하고 다양한 기능을 사용할 수 있지만 그만큼 복잡하고 학습할 것 도 많다. 실무에선 Spring AOP만으로 충분히 문제를 해결할 수 있으니 Spring AOP에 집중하도록 하자.&#x20;



{% hint style="info" %}
AOP 용어 정리&#x20;

* Join point: AOP를 적용할 수 있는 모든 지점을 뜻하는 추상적인 개념&#x20;
* Pointcut: 어드바이스가 적용될 위치를 선별하는 기능&#x20;
* Target: 포인트 컷으로 결정되는 어드바이스가 적용될  객체
* Advice: 부가 기능, around, before, after 같은 다양한 종류의 어드바이스가 있음
* Aspect: 어드바이스 + 포인트 컷, @Aspect&#x20;
* Advisor: 하나의 어드바이스와 하나의 포인트 컷으로 구성 (spring aop에서만 사용)
* Weaving: 포인트 컷으로 결정한 타겟의 조인 포인트에 어드바이스를 적용하는 것
* AOP 프록시: AOP 기능을 구현하기 위해 만든 프록시 객체, 스프링에서 AOP 프록시는 JDK Dynamic Proxy 또는 CGLIB 프록시가 있다.
{% endhint %}



## 구현 예



```java
@Slf4j
@Aspect
@Component // 빈으로 등록해야함
public class Aspect {
    // Around 사용 동시에 point컷 설정
    @Around("execution(* hello.aop.order..*(..))") 
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature()); //join point 시그니처
        return joinPoint.proceed();
    }

    //Poiunt cut 설정
    @Pointcut("execution(* hello.aop.order..*(..))")
    private void allOrder(){} //pointcut signature

    //클래스 이름 패턴이 *Service
    @Pointcut("execution(* *..*Service.*(..))")
    private void allService(){}
    
    // 포인트 컷 사용
    @Around("allOrder()") 
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature()); //join point 시그니처
        return joinPoint.proceed();
    }

    //hello.aop.order 패키지와 하위 패키지 이면서 클래스 이름 패턴이 *Service
    @Around("allOrder() && allService()")
    public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {

        try {
            log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
            Object result = joinPoint.proceed();
            log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
            return result;
        } catch (Exception e) {
            log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
            throw e;
        } finally {
            log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
        }
    }

}

```



### AOP 순서&#x20;

어드바이스는 기본적으로 순서를 보장하지 않는다. 순서를 지정하고 싶으면 @Order 애노테이션을 적용하면 되는데 중요한 점은 class 단위로 적용해야 하는 것이다.&#x20;



```java
@Slf4j
public class AspectOrder {

    @Aspect
    @Order(2)
    public static class LogAspect {
        @Around("hello.aop.order.aop.Pointcuts.allOrder()")
        public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
            log.info("[log] {}", joinPoint.getSignature()); //join point 시그니처
            return joinPoint.proceed();
        }
    }

    @Aspect
    @Order(1)
    public static class TxAspect {
        @Around("hello.aop.order.aop.Pointcuts.orderAndService()")
        public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {

            try {
                log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
                Object result = joinPoint.proceed();
                log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
                return result;
            } catch (Exception e) {
                log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
                throw e;
            } finally {
                log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
            }
        }
    }


}

```



### 어드바이스 종류&#x20;

Around는 가장 많은 기능을 가지고 있지만 그 만큼 책임이 많아 실수할 여지가 생긴다. 무작정 Around를 사용하기보다 어드바이스 범위에 따라 아래의 적절한 어드바이스를 사용하자.

* @Around: 메서드 호출 전후에 수행되는 가장 강력한 어드바이스. 반드시 joinPoint.proceed()를 호출해 다음 어드바이스나 타겟을 호출해야 한다.
* @Before: 조인 포인트 실행 이전에 실행된다. 자동으로 다음 작업 진행 됨
* @AfterReturning: 조인 포인트가 정상 완료 후 실행&#x20;
* @After: 조인 포인트가 정상 또는 예외에 관계 없이 실행 (finally). 일반적으로 리소스를 해제하는 데 사용한다.



## 포인트 컷 지시자

포인트컷을 편리하게 표현하기 위한 특별한 표현식을 제공한다.&#x20;

포인트컷은 `execution`같은  포인트컷 지시자로 시작한다.&#x20;

* **execution** : 메소드 실행 조인 포인트를 매칭한다. 스프링 AOP에서 가장 많이 사용됨.

<details>

<summary>기타 지시자 종류</summary>

* within: 특정 타입 내의 조인 포인트를 매칭한다.&#x20;

<!---->

* args: 인자가 주어진 타입의 인스턴스인 조인 포인트&#x20;

<!---->

* this: 스프링 빈 객체를 대상으로하는 조인포인트&#x20;

<!---->

* bean: 스프링 전용 포인트 컷 지시자, 빈의 이름으로 포인트컷을 지정함.
* @args: 전달된 실제 인수의 런타임 타입이 주어진 타입의 애노테이션을 갖는 조인 포인트&#x20;
* @annotation: 메서드가 주어진 애노테이션을 가지고 있는 조인 포인트를 매칭&#x20;
* @within: 주어진 애노테이션이 있는 타입 내 조인 포인트&#x20;
* @target: 실행 객체의 클래스에 주어진 타입의 애노테이션이 있는 조인 포인트&#x20;

</details>



## Execution&#x20;

Execution 문법을 통해 메서드 실행 조인 포인트를 매칭한다.&#x20;

* ?로 생략가능
* \*로 아무 값이 들어와도 됨&#x20;
* (..): 파라미터 타입과 파라미터 수가 상관 없다는 뜻

```java
execution(접근제어자? 반환타입 선언타입?메서드이름(파라미터) 예외?)
```

#### 가장 많이 생략한 포인트 컷

```java
execution(* *(..)) // 반환타입, 메서드이름, 파라미터
```



#### 가장 정확한 포인트 컷

```java
execution(public String hello.aop.member.MemberServiceImpl.hello(String))
```



#### 메서드 이름 매칭&#x20;

```java
execution(* hello(..)) // 정확한 메서드명
execution(* hel*(..)) // hel로 시작하는 메서드명
execution(* *el*(..)) // el를 포함하는 메서드명
```

#### 패키지명 매칭&#x20;

```java

execution(* hello.aop.member.MemberServiceImpl.hello(..)) // 정확한 패키지
execution(* hello.aop.member.*.*(..)) // 멤버 패키지에 해당하는 것들 
execution(* hello.aop.*.*(..)) // aop 패키지만 해당하는 것들 (member 패키지에 적용 x )
execution(* hello.aop..*.*(..)) // 점 두개는 해당 패키지와 그 하위 패키지까지 해당하는 것들도 포함한다.

```

&#x20;

#### 타입 매칭&#x20;

```java
execution(* hello.aop.member.MemberServiceImpl.*(..)) // MemberSerivceImpl 
execution(* hello.aop.member.MemberService.*(..)) // 부모타입을 선언해도 다형성이 적용되어 자식 타입도 매칭된다. 
```



#### 파라미터 매칭&#x20;

```java
execution(* *(String)) // String 파라미터
execution(* *()) // 파라미터가 없어야함 
execution(* *(*)) // 하나의 파라미터 허용하고 모든 타입 허용
execution(* *(..)) //  모든 타입, 숫자 허용 
execution(* *(String, ..)) //  첫번째는 String, 그 뒤는 모두 허용 
```



### 그 밖의 어드바이스들&#x20;

#### within&#x20;

특정 타입으로 조인포인트를 매칭한다. execution과 다르게 부모 타입을 지정하면 안되고 정확하게 타입이 맞아야한다.&#x20;

```java
within(hello.aop.member.*Service*)
within(hello.aop..*)
```

#### args&#x20;

인자로 조인포인트를 매칭.&#x20;

기본 문법은 execution과 같지만 args는 부모 타입을 허용하고 실제 넘어온 파라미터 객체 인스턴스를 보고 판단한다. 단독으로 사용되기 보다는 파라미터 바인딩에서 주로 사용됨

```java
class MemberServiceImpl{
 fun hello(s: String){} 
} 

args(String) 
args(java.io.Serializable) // // 동적으로 넘어온 실제 파라미터 객체 인스턴스를 판단하기 때문에 매칭 
args(Object) // 동적으로 넘어온 실제 파라미터 객체 인스턴스를 판단하기 때문에 매칭 

execution(* *(java.io.Serializable)) // 정적으로 판단하기 때문에 매칭 안됨
```

#### @target, @within

@target: 실행객체의 클래스에 주어진 타입의 애노테이션이 있는 조인 포인트

@within: 주어진 애노테이션이 있는 타입 내 조인 포인트

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface ClassAop {
}



// 사용 
@Around("execution(* hello.aop..*(..)) && @target(hello.aop.member.annotation.ClassAop)") //target
@Around("execution(* hello.aop..*(..)) && @within(hello.aop.member.annotation.ClassAop)") //wihin
```

#### 차이

#### @target 은 인스턴스의 모든 메서드를 조인 포인트로 적용한다. 다시말해 부모 클래스의 메서드까지 조인 포인트로 적용한다.&#x20;

#### @within 은 해당 타입 내에 있는 메서드만 조인 포인트로 적용한다. within은 오직 해당 인스턴스 메서드만 조인 포인트로 적용한다.



#### @annotation

@annotation: 메서드가 주어진 애노테이션을 가지고 있는 조인 포인트를 매칭

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MethodAop {
    String value();
}

public class MemberServiceImpl {
      @MethodAop("test value")
      public String hello(String param) {
          return "ok";
      }
}

@Around("@annotation(hello.aop.member.annotation.MethodAop)")

```



#### @bean&#x20;

@bean : 스프링 전용 포인트컷 지시자로 빈의 이름을 지정해 조인 포인트를 매칭

```java
@Around("bean(orderService) || bean(*Repository)")
```



### 매개변수 전달&#x20;

`this, target, args, @target, @within, @annotation, @args 어드바이스들은 매개변수를 전달받을 수 있다.`&#x20;

```java
@Aspect
static class ParameterAspect {
    // pointcut 
    @Pointcut("execution(* hello.aop.member..*.*(..))")
    private void allMember() {}
        
    // getArgs 메서드 사용
    @Around("allMember()")
    public Object logArgs1(ProceedingJoinPoint joinPoint) throws Throwable
        Object arg1 = joinPoint.getArgs()[0]; 
        log.info("[logArgs1]{}, arg={}", joinPoint.getSignature(), arg1);
        return joinPoint.proceed();
    }
    
    // args사용
    @Around("allMember() && args(arg,..)")
    public Object logArgs2(ProceedingJoinPoint joinPoint, Object arg) throws Throwable {
        log.info("[logArgs2]{}, arg={}", joinPoint.getSignature(), arg);
        return joinPoint.proceed();
    }
    
    // 실제 원본객체를 받음
     @Before("allMember() && this(obj)")
    public void thisArgs(JoinPoint joinPoint, MemberService obj) {
        log.info("[this]{}, obj={}", joinPoint.getSignature(), obj.getClass());
    }
    
    // 프록시 객체를 받음
    @Before("allMember() && target(obj)")
    public void targetArgs(JoinPoint joinPoint, MemberService obj) {
        log.info("[target]{}, obj={}", joinPoint.getSignature(), obj.getClass());
    }
    
    // 프록시 객체를 받음
    @Before("allMember() && @target(annotation)")
    public void targetArgs(JoinPoint joinPoint, ClassAop annotation) {
        log.info("[target]{}, obj={}", joinPoint.getSignature(), annotation);
    }
    
    
    // 애노테이션을 받음
    @Before("allMember() && @target(annotation)")
    public void targetArgs(JoinPoint joinPoint, ClassAop annotation) {
        log.info("[target]{}, obj={}", joinPoint.getSignature(), annotation);
    }
    
    // 애노테이션을 받음
    @Before("allMember() && @within(annotation)")
    public void targetArgs(JoinPoint joinPoint, ClassAop annotation) {
        log.info("[target]{}, obj={}", joinPoint.getSignature(), annotation);
    }
    
    // 애노테이션을 받고, 애노테이션의 값도 꺼낼 수 있음
    @Before("allMember() && @annotation(annotation)")
    public void targetArgs(JoinPoint joinPoint, MethodApp annotation) {
        log.info("[target]{}, annotationValue={}", joinPoint.getSignature(), annotation.value());
    }
    
}
```





## AOP 실무 주의사항&#x20;

### 1. 프록시와 내부 호출 문제&#x20;

어떤 클래스에서 external 메서드는 internal 내부 메서드를 호출하고 있다. 이 때 각 메서드에 AOP를 적용하고 external을 호출해보자.&#x20;

external은 기대했던 대로 aop가 적용되지만, 이 후 internal메서드는 aop가 적용되지 않은채 호출이 된다.&#x20;

<img src="../../../.gitbook/assets/file.excalidraw (5).svg" alt="" class="gitbook-drawing">

이유는 실제 internal 내부 메서드는 proxy에서 호출디는게 아닌 target 원본 객체에서 호출되기 때문이다. 위의 그림을 보면 external은 AOP Proxy를 거치기 때문에 어드바이스가 호출되고 있다.&#x20;

하지만 internal 호출은 타겟의 원본 메서드에서부터 시작되기 때문에 원본 internal 메서드만 호출되기 때문에 어드바이스가 호출되지 않는 것이다. this가 AOP Proxy가 아니라 Target인 것이다.

#### 해결법&#x20;

1. 구조를 분리하기(권장)

```java
@Component
@RequiredArgsConstructor
public class CallService {
  private InternalService internalService; // Service를 따로 분리해서 호출함.
  
  public void external() {
     internalService.internal();
 }
}
```

2. ObjectProvier나 ApplicationContext를 사용해서 지연 조회해 자기 자신을 주입받기&#x20;

```java
@Component
@RequiredArgsConstructor
public class CallService {
  private final ObjectProvider<CallServiceV> callServiceProvider;
  public void external() {
      applicationContext.getBean(CallService.class);
      CallService callService = callServiceProvider.getObject(); 
      callService.internal(); //외부 메서드 호출
 }
 
 public void internal() {
      log.info("call internal");
 }
}
```

### 2. JDK 동적 프록시 문제&#x20;

### 2.1 타입 캐스팅&#x20;

JDK 동적 프록시는 인터페이스만을 구현하면서 프록시를 생성하기 때문에 구체 클래스는 전혀 알지 못한다. 따라서 구체 클래스로 타입 캐스팅이 되지 않는다.  &#x20;

반면 CGLIB는 구체 클래스를 상속받는 형식으로 프록시를 생성하기 때문에 인터페이스와 구체 클래스 둘다 타입 캐스팅이 가능하다.

### 2.2 의존관계 주입&#x20;

JDK 동적 프록시는 구체클래스를 의존 관계 주입하지 못한다. 앞서 살펴봤듯이 JDK 동적프록시는 인터페이스를 기반으로 만들어진다. 따라서 구체 클래스를 알지못하기 때문에 의존관계 또한 주입하지 못한다.&#x20;

반면 CGLIB는 구체타입을 의존관계 주입을 할 수 있다. &#x20;

물론 의존관계 주입은 Interface 기반으로 의존관계하는게 맞지만, 테스트나 여러가지 이유로 구체 클래스를 직접 의존관계 주입 받아야하는 경우가 있다. 이 때는 CGLIB를 사용하도록 한다.&#x20;



## 3. CGLIB 문제&#x20;

* 대상 클래스에 기본 생성자가 필수

CGLIB는 구체 클래스를 상속 받는 형식으로 프록시를 생성한다. 자바 언어에서 상속을 받으면 자식 클래스의 생성자를 호출할 때 부모 클래스의 생성자도 호출해야한다. &#x20;

CGLIB 프록시는 대상 클래스를 상속받고 생성자에서 대상 클래스의 기본 생성자를 호출하기 때문에 반드시 대상클래스에는 기본 생성자가 있어야 한다.



* 생성자 2번 호출&#x20;

한번은 실제 타겟의 객체를 생성할 때 또 한번은 프록시 객체를 생성할 때 부모 클래스(타겟)의 생성자를 호출한다.&#x20;

* final 키워드 클래스, 메서드 사용 불가&#x20;

final class는 상속이 불가능하고, final method는 오버라이딩이 불가능하다. 따라서 final 키워드는 사용하지 못한다.&#x20;



## 스프링의 해결책&#x20;

스프링의 기술 선택 변화&#x20;

* 스프링 3.2부터 CGLIB를 스프링 내부에 함께 패키징 해 별도 라이브러리 추가가 불필요해졌다.
* 스프링 4.0부터 CGLIB 기본 생성자가 필수인점과 생성자가 2번 호출되는 문제를 해결했다. (objenesis라이브러리 사용)
* 스프링 부트 2.0부터 CGLIB를 기본으로 사용한다.&#x20;

따라서 인터페이스라도 JDK 동적프록시가 아니라 CGLIB를 사용한다. 스프링이 기본생성자와 생성자 2번 호출 문제를 해결했기 때문이다. 보통 AOP 적용 대상은 final로 선언하지 않기 때문에 크게 문제가 되지 않을 것이다.&#x20;

















