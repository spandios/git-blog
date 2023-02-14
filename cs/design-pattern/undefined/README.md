# 객체 생성 관련

## Singleton Pattern

### 사용이유와 장단점

* 사용이유
  * 무상태 객체나 설계상 유일해야할 때
* 장점
  * 하나의 인스턴스만을 사용하기 때문에, 불필요한 인스턴스 생성 비용을 아낀다.
* 단점
  * 멀티쓰레드 환경이라면 여러 인스턴스를 만들 경우도 있다.
  * private 생성자이므로 상속이 불가하며 많은 테스트 프레임워크에서 사용 불가
  * 전역변수와 같으므로 많은 곳에서 사용되면 결합도가 높아진다.

### 구현

```java
// Static Inner Class를 사용해 미리 로딩되어진 Pattern Instnace를 사용한다.
// 역직렬화 때도 대처함.
public class Pattern implements Serializeable{
	private Pattern(){} 

	private static class PatternHolder{
		private static final Pattern INSTANCE = new Pattern();
	}

	public static Pattern getInstance(){
		return PatternHolder.INSTNACE;
	}
	
  // 역직렬화 때 이 메서드를 반드시 실행함.
	protected Object readResoleve(){
		return getInstnace();
	}

}

```

###

### Enum을 통한 구현

* 왜? Reflection과 직렬, 역직렬화에 안전하다.
* 단점
  * 사용하지 않더라도 미리 클래스가 로딩됨.

```java
public enum Pattern {
	INSTANCE;
}
```

* Singleton 구현하기
  1. 단순구현 ⇒ 멀티 쓰레드일 시 문제 발생 할 수 있음.
  2. 이른 초기화 ⇒ static 변수로 미리 만들어 둠으로 써 사용, 미리 만든 것 자체가 사용 안하면 낭비일 수도 있음
  3. syncronize ⇒ 사용 되어 질 때 동기적으로 만들 수 있음. 너무 복잡함
  4. static inner class 사용하기 ⇒ 멀티쓰레드와 이른 초기화에 대응 함 Reflection 대응은 불가능.
  5. enum class 사용하기 ⇒ 멀티 쓰레드, Reflection, 직렬, 역질화 대응하지만 이른초기화.
* enum을 사용해 싱글톤 패턴을 구현하는 방버의 장점과 단점은?
  1. 장점 : 멀티쓰레드와 Reflection, 직렬화 역직렬화에 대응
  2. 단점 : 이른 초기화

### 스프링에서

1. 스프링 빈에서 싱글톤 스코프.



## Factory Method Pattern

클라이언트에서 직접 인스턴스를 만들지 않고, 어떤 인스턴스를 만들지 \*\*서브 클래스(Factory Class)\*\*가 정하는 패턴

구체적인 객체 생성 과정을 하위 또는 구체적인 클래스로 옮기는 것이 목적이다.

* 왜?
  * 어떤 로직에서 직접 클래스를 생성하고 어떤 파라미터마다 클래스에 매번 어떤 설정을 해준다면, 어떤 클래스가 추가될 때마다 매번 로직이 변경에 열려있다.
  * 클래스 생성의 역할만을 하는 팩토리 클래스를 통해 클래스 타입이 추가 돼도 로직이나 클라이언트 측은 큰 변경이 없도록 느슨하게 설계한다.
  * 핵심은 만들어낼 Product(객체)와 Creator(factory) 클래스를 추상화하고 실제 어떤 구현 객체를 사용할지는 특정 팩토리 클래스에서 특정 구현 객체를 생성하는 역할을 맡는다.
* 장점
  * 변경에 닫힌 느슨한 프로그래밍이 가능하게 된다.
* 단점
  * 코드양이 많아진다.

#### 구현

```java
// Factory Interface 

public interface ShipFactory {

    default Ship orderShip(String name, String email) {
        validate(name, email);
        prepareFor(name);
        Ship ship = createShip();
        sendEmailTo(email, ship);
        return ship;
    }

    void sendEmailTo(String email, Ship ship);

    Ship createShip();

    private void validate(String name, String email) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("배 이름을 지어주세요.");
        }
        if (email == null || email.isBlank()) {
            throw new IllegalArgumentException("연락처를 남겨주세요.");
        }
    }

    private void prepareFor(String name) {
        System.out.println(name + " 만들 준비 중");
    }

}

// WhiteShipFactory 실제  특정 구현 객체를 만들어내는 Factory
public class WhiteshipFactory extends DefaultShipFactory {

    @Override
    public Ship createShip() {
        return new Whiteship();
    }
}

// 클라이언트는 단지 Factory를 통해 얻어진 객체를 사용할 뿐이다. (IOC)

public class Client {

    public static void main(String[] args) {
        Client client = new Client();
        client.print(new WhiteshipFactory(), "whiteship", "keesun@mail.com");
        client.print(new BlackshipFactory(), "blackship", "keesun@mail.com");
    }

    private void print(ShipFactory shipFactory, String name, String email) {
        System.out.println(shipFactory.orderShip(name, email));
    }

}
```

#### 팩토리 메소드 복습

1. 확장에 열려 있고 변경에 닫혀있는 객체 지향원칙을 설명하기.

기존 코드는 수정하지 않으면서 새로운 기능을 추가할 수 있는 것.

1. 자바 8부터 추가된 default 메소드에 대해 설명하기

기존 인터페이스는 추상 메서드만 선언하였지만, default method를 통해 기본 구현체를 실제 만들 수 있다.

자바9부터는 인터페이스에 오버라이딩이 불가능하고 내부 구현 메서드에서만 사용 가능한 private method도 작성할 수 있다.

따라서 abstract class의 기능을 많이 대체하므로 이제 interface를 더 활용해서 쓰도록 하자.

#### 스프링에서

1. 스프링 빈 팩토리

```
BeanFactory xmlFactory = new ClassPathXmlApplicationContext("config.xml");
String hello = xmlFactory.getBean("hello", String.class);
System.out.println(hello);

BeanFactory javaFactory = new AnnotationConfigApplicationContext(Config.class);
String hi = javaFactory.getBean("hello", String.class);
System.out.println(hi);
```

##

## Abstract Factory Pattern

서로 관련있는 여러 객체를 만들어주는 인터페이스

관련 있는 여러 객체를 구체적인 클래스에 의존하지 않고 만들수 있게 해주는 것이 목적이다.

* 왜?
  * 클라이언트 측이 추상화에 의존하지 않고, 구체화에 의존하기 때문에 다른 구체화 클래스를 사용하려면 변경해야하므로 변경에 열려있으며 확장에 닫혀 있다.
  * 따라서 여러 객체를 사용할 때 확장성있고 변경엔 닫힌 개발을 하기 위해 추상적인 팩토리 interface를 사용한다.
  * 실제 만들어 낼 객체는 추상팩토리를 구현할 클래스에서 정한다.
  * 따라서 클라이언트는 단지 추상팩토리를 구현한 클래스만 의존성을 주입받으면 기존 코드는 변경없이 사용 가능하다.
* 장점
  * 변경에 닫힌 느슨한 프로그래밍이 가능하게 된다.
* 단점
  * 코드양이 많아진다.
  * 여러 객체를 만들어주는 것 자체가 단일책임원칙에 벗어난 것일 수도 있다.

#### 구현

```java
// Abstract Factory Interface 
public interface ShipPartsFactory {

    Anchor createAnchor();

    Wheel createWheel();

}

// WhiteShipFactory 실제  특정 구현 객체를 만들어내는 Factory
public class WhitePartsProFactory implements ShipPartsFactory {
    @Override
    public Anchor createAnchor() {
        return new WhiteAnchorPro();
    }

    @Override
    public Wheel createWheel() {
        return new WhiteWheelPro();
    }
}

// 클라이언트는 단지 Factory를 통해 얻어진 객체를 사용할 뿐이다.

public class Client {

   ShipFactory shipFactory = new WhiteshipFactory(new WhiteshipPartsFactory());
        Ship ship = shipFactory.createShip();
        System.out.println(ship.getAnchor().getClass());
        System.out.println(ship.getWheel().getClass());

}
```

#### 자바 or 스프링에서

1. 스프링 빈 팩토리

```java
// 실제 팩토리의 구현체
public class ShipFactory implements FactoryBean<Ship> {

    @Override
    public Ship getObject() throws Exception {
        Ship ship = new Whiteship();
        ship.setName("whiteship");
        return ship;
    }

    @Override
    public Class<?> getObjectType() {
        return Ship.class;
    }
}
```

##

## Builder Pattern

Builder 구성으로 복잡한 객체 생성을 프로세스화 한다.

* 왜?
  * 객체 생성자에서 4개 이상의 파라미터 또는 여러 생성자를 통한 생성은 가독성이 떨어진다.
  * java bean대로 setProperty 방식은 완전성이 부족하다.
  * 따라서 builder 패턴을 통해 가독성과 완정성을 챙긴다.
* 장점
  * 가독성과 완정성을 챙기고 객체를 생성할 수 있다.
* 단점
  * 코드양이 많아진다.

#### 구현

```java
// Builder Pattern
public class DefaultTourBuilder implements TourPlanBuilder {

    private String title;

    private int nights;

    private int days;

    private LocalDate startDate;

    private String whereToStay;

    private List<DetailPlan> plans;

    @Override
    public TourPlanBuilder nightsAndDays(int nights, int days) {
        this.nights = nights;
        this.days = days;
        return this;
    }

    @Override
    public TourPlanBuilder title(String title) {
        this.title = title;
        return this;
    }

    @Override
    public TourPlanBuilder startDate(LocalDate startDate) {
        this.startDate = startDate;
        return this;
    }

    @Override
    public TourPlanBuilder whereToStay(String whereToStay) {
        this.whereToStay = whereToStay;
        return this;
    }

    @Override
    public TourPlanBuilder addPlan(int day, String plan) {
        if (this.plans == null) {
            this.plans = new ArrayList<>();
        }

        this.plans.add(new DetailPlan(day, plan));
        return this;
    }

    @Override
    public TourPlan getPlan() {
        return new TourPlan(title, nights, days, startDate, whereToStay, plans);
    }
}

// Director를 통해 미리 준비된 객체 생성도 가능
public class TourDirector {

    private TourPlanBuilder tourPlanBuilder;

    public TourDirector(TourPlanBuilder tourPlanBuilder) {
        this.tourPlanBuilder = tourPlanBuilder;
    }

    public TourPlan cancunTrip() {
        return tourPlanBuilder.title("칸쿤 여행")
                .nightsAndDays(2, 3)
                .startDate(LocalDate.of(2020, 12, 9))
                .whereToStay("리조트")
                .addPlan(0, "체크인하고 짐 풀기")
                .addPlan(0, "저녁 식사")
                .getPlan();
    }

    public TourPlan longBeachTrip() {
        return tourPlanBuilder.title("롱비치")
                .startDate(LocalDate.of(2021, 7, 15))
                .getPlan();
    }
}
```

#### 자바 or 스프링에서

1. StringBuilder and StringBuffer (String Builder는 Sync가 아님)

```java
// 실제 팩토리의 구현체
StringBuilder stringBuilder = new StringBuilder();
        String result = stringBuilder.append("whiteship").append("keesun").toString();
        System.out.println(result);

// Stream
Stream<String> names = Stream.<String>builder().add("keesun").add("whiteship").build();
```

## Prototype Pattern
