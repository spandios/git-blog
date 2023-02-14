---
description: DDD에 특히 필요한 것 위주로 정리
---

# 레포지터리와 모델 구현

## ORM

객체 기반의 도메인 모델과 관계형 데이터 모델 간의 매핑 처리를 도와주는 기술



## @Embeddable

Value 타입 객체는 @Embeddable 애너테이션으로  매핑한다. 사용은 @Embdded 사용하자.

기존 Value 타입 객체에 정의된 컬럼 이름을 바꿔 사용하고 싶다면 @ArrtibuteOverrieds를 사용하자.

```java
@Embeddable
public class Orderer{
    @Embedded
    @AttributeOverrieds(
    @AttributeOverride(name="id", column = @Column(name = "orderer_id")
    )
    private MemberId memberId;
    
    @Column(name = "orderer_name")
    private String name;
}


@Embeddable
public class MemberId{
    @Column(name = "member_id")
    private String id;
}
```



## AttributeConverter&#x20;

프로퍼티 값과 db 컬럼 값을 변환을 처리할 수 있는 interface이다. &#x20;

```java
/**
 * A class that implements this interface can be used to convert 
 * entity attribute state into database column representation 
 * and back again.
 * Note that the X and Y types may be the same Java type.
 *
 * @param <X>  the type of the entity attribute
 * @param <Y>  the type of the database column
 */
public interface AttributeConverter<X,Y> {

    /**
     * Converts the value stored in the entity attribute into the 
     * data representation to be stored in the database.
     *
     * @param attribute  the entity attribute value to be converted
     * @return  the converted data to be stored in the database 
     *          column
     */
    public Y convertToDatabaseColumn (X attribute);

    /**
     * Converts the data stored in the database column into the 
     * value to be stored in the entity attribute.
     * Note that it is the responsibility of the converter writer to
     * specify the correct <code>dbData</code> type for the corresponding 
     * column for use by the JDBC driver: i.e., persistence providers are 
     * not expected to do such type conversion.
     *
     * @param dbData  the data from the database column to be 
     *                converted
     * @return  the converted value to be stored in the entity 
     *          attribute
     */
    public X convertToEntityAttribute (Y dbData);

```

* 타입 파라미터 X는 객체의 타입이고 Y는 데이터베이스 컬럼의 타입이다.
* convertToDatabaseColumn 메서드는 객체 프로퍼티 값을 데이터베이스 컬럼 값으로 변환한다.
* convertToEntityAttribute 메서드는 데이터베이스 컬럼 값을 객체 프로터리 값으로 변환한다.

### 사용하기

```java
// Converter 생성 
public class MoneyConvnerter implements AttributeConvereter<Money, Interger>{
    @Override
    public Integer convertToDatabaseColumn(Money money){
        return money == null ? null : money.getValue();
    }
    
    @Override
    public Integer convertToEntityAttribute(Interger value){
        return value == null ? null : new Money(value);
    }
}

// 사용 명시
public class Order{
    @Convert(converter = MoneyConverter.class)
    private Money totalAmounts;
}
```



## 밸류 컬렉션 저장하기

밸류타입의 객체를 컬렉션으로 저장하려면 @ElementCollection, @CollectionTable을 사용하면 된다.

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-09 오후 12.16.05.png" alt=""><figcaption></figcaption></figure>

```java
@Entity
public class Order{
    @EmbeddedId
    private OrderNo no;
    
    @ElementCollection(eager = FetchType.EAGER)
    // 밸류를 저장할 테이블을 지정한다.
    @CollcetionTable(name="order_line", joinColumns = @JoinColumn(name = "order_number")
    // OrderColumn 애너테이션이 지정한 칼럼에 리스트의 인덱스 값을 저장함.
    @OrderColumn(name= "line_idx") // OrderLine Value에 따로 명시 안해도 됨. 
    private List<OrderLine> orderLines;
    
}
```



## 밸류를 이용한 ID 매핑

```java
@Embeddable
public class OrderNo implements Serializable{
    @Column(name="order_number")
    priave String number;
}
```

OrderNo같이 식별자를 나타내는 밸류 타입 객체는 코드는 더 많아지지만 식별자에 기능을 추가할 수 있다는 점이 장점이다.&#x20;

사용은 식별자 밸류 객체에 @EmbeddedId 애너테이션을 명시해서 가능하다.

```java
@Entity
public class Order{
    @EmbeddedId
    private OrderNo no;
}
```



## 밸류와 별도의 Table을 매핑하기

{% hint style="info" %}
애그리거트에서 루트 엔티티를 뺀 나머지 구성요소는 대부분 밸류이다
{% endhint %}

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-09 오후 1.16.30.png" alt=""><figcaption></figcaption></figure>

Article과 ArticleContent가 있다고 했을 때 1-1연관으로 매핑은 잘못된 매핑이다.

ArticleContent가 별도로 식별자를 가지고 있는 것이 아닌 단지 ARTICLE과 연관 시키기 위해 ARTICLE의 ID를 가져오기 때문이다.&#x20;

따라서 아래와 같이 Value타입으로 매핑한다.

추가적으로 밸류를 매핑 한 테이블을 지정하기 위해 @SecondaryTable과 @ArrtibuteOverride를 사용한다.

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-09 오후 1.22.33.png" alt=""><figcaption></figcaption></figure>

```java
@SecondayTable(name="article_content",
pkJoinColumns = @PrimaryKeyJoinColumn(name= "id")
)
public class Article{

    @Embedded
    private Artcile
}
```

### @SecondaryTable

* name: 테이블명을 지정한다.
* pkJoinColimns: 밸류 테이블에서 엔티티 테이블로 조인할 때 사용할 컬럼을 지정한다.

###

### ArrtibuteOverride

AttibuteOverride를 사용해서 밸류 데이터가 저장된 테이블 이름을 지정한다.

```java
@SecondayTable(name="article_content",
pkJoinColumns = @PrimaryKeyJoinColumn(name= "id")
)
public class Article{
    @AttributeOrverrides({
        @AttributeOverride(name="content", column = @Column(table="article_content", name="content"),
        @AttributeOverride(name="contentType", column = @Column(table="article_content", name="content_type")
    })
    @Embedded
    private ArticleContent content;
}
```



## 밸류 컬렉션을 @Entity로 사용하기

구현 기술의 한계나 팀 표준 때문에 @Entity로 사용할 때도 있다.

예를들어, JPA는 @Embeddable타입의 클래스 상속 매핑을 지원하지 않는다.&#x20;

상속 구조를 갖는 밸류타입을 사용하려면 @Entity를 이용해 상속 매핑으로 처리해야 한다.



아래 예시는 추상 클래스 Image를 두고 타입에 따라 다른 Image 객체를 상속받아 사용할 수 있게 한다.

<figure><img src="../../../.gitbook/assets/스크린샷 2023-02-09 오후 3.24.39.png" alt=""><figcaption></figcaption></figure>

### Abstract Image

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@Discriminator(name="image_type")
public abstract class Image{
     ~~    
}
```

* Inheritance&#x20;

다른 하위 클래스를 위한 상속용 클래스인 것을 명시한다.&#x20;

* SINGLE\_TABLE & Discriminator&#x20;

상속 관계에 있는 모든 테이블을 합쳐서 하나의 테이블로 만든다. 때문에 다른 클래스와 구분하기 위해서는 Discriminator 애너테이션을 사용해야 한다.



### 상속받는 Image클래스들

```java
@Entity
@DiscriminatorValue("InternalImage")
public class InternalImage extends Image{

}

@Entity
@DiscriminatorValue("EternalImage")
public class ExternalImage extends Image{

}
```



### 컬렉션 매핑하기

image는 밸류이므로 독자적인 사이클을 갖지 않고 product에 완전히 의존한다.&#x20;

따라서 Product를 저장할 때 함께 저장되고 Product를 삭제할 때 함께 삭제되도록 cascade 속성을 설정하고 orpanRemoval를 true로 지정한다.

```java
public class Product{
    @OneToMany(cascade = {CascadeType.PERSIST, CascadeType.REMOVE}, orpanRemoval = true ) 
    @JoinColumn(name = "product_id")
    private List<image> images = new ArrayList();
}
```



### Clear함수 고려할 점&#x20;

#### Entity에서 Clear함수는 성능이 좋지 않다.

각 컬렉션 요소마다 각각의 delete 쿼리를 날리기 때문에 성능이 좋지 않다.&#x20;

#### Embeddable에서는 한 번의 쿼리로 삭제한다.

entity보다 성능은 좋지만 타입을 값으로 가지고 있기 때문에 구분을 하기 위해 if else 처리가 필요하다.





## 애그리거트 로딩 전략&#x20;

JPA 매핑을 설정할 때 항상 기억해야하는 점은 애그리거트에 속한 객체가 모두 모여야 완전한 하나가 된다는 것이다.

가장 편하게 애그리거트를 완전한 상태가 되려면 조회 방식을 즉시 로딩으로 설정하면 된다. 조회 즉시 연관된 엔티티나 밸류 객체를 조회하기 때문이다.

하지만 즉시 로딩이 항상 좋은 것은 아니다. 최적화 되지 않은 쿼리를 실행해 성능 이슈 또는 불필요한 리소스 비용이 발생할 수 있기 때문이다.&#x20;

애거리거트는 개념적으로는 하나여야 한다. 하지만 그렇다고 루트 엔티티를 로딩 하는 시점에 애그리거트에 속한 모든 객체를 로딩해야 한다는 것은 아니다. 성능을 고려해서 로직 중에 지금 로딩이 필요 없는 객체는 로딩하지 않다가, 필요할 때  로딩하는 지연 로딩도 고려해서 상황에 맞는 로딩 전략을 선택하는게 중요하다.





#### 참고

{% embed url="https://browndwarf.tistory.com/54" %}

