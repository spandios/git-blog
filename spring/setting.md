---
description: 근데 Kotlin을 곁들인
---

# Spring Boot 프로젝트 설정

[https://start.spring.io](https://start.spring.io) 은 기본 스프링 셋팅과 따로 선택한 기술을 포함한 스프링 프로젝트를 손쉽게 생성해준다.

이번 글은 start.spring.io가 제공해준 프로젝트에 작성된 gradle 환경설정과 spring boot의 핵심 의존성을 알아보려 한다.&#x20;

## 전체 gradle&#x20;

```gradle
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

group = "com.demo"
version = "0.0.1-SNAPSHOT"
java.sourceCompatibility = JavaVersion.VERSION_17

repositories {
    mavenCentral()
}

plugins {
    id("org.springframework.boot") version "3.0.2"
    id("io.spring.dependency-management") version "1.1.0"
    kotlin("jvm") version "1.7.22"
    kotlin("plugin.spring") version "1.7.22"
    kotlin("plugin.jpa") version "1.7.22"
}

allOpen {
    annotation("jakarta.persistence.Entity")
    annotation("jakarta.persistence.MappedSuperclass")
    annotation("jakarta.persistence.Embeddable")
}


dependencies {
    // Spring
    implementation("org.springframework.boot:spring-boot-starter")
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-aop")
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("org.springframework.boot:spring-boot-starter-actuator")
    annotationProcessor("org.springframework.boot:spring-boot-configuration-processor")

    // Spring Data
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    runtimeOnly("mysql:mysql-connector-java")
    implementation("com.h2database:h2")

    // Kotlin
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
    implementation("org.jetbrains.kotlin:kotlin-reflect")

    // Test
    testImplementation("org.springframework.boot:spring-boot-starter-test")

    // Util
    implementation("org.modelmapper:modelmapper:3.1.0")
}

tasks.withType<KotlinCompile> {
    kotlinOptions {
        freeCompilerArgs = listOf("-Xjsr305=strict")
        jvmTarget = "17"
    }
}

tasks.withType<Test> {
    useJUnitPlatform()
}

```



## Plugins

```gradle
plugins {
    id("org.springframework.boot") version "3.0.2"
    id("io.spring.dependency-management") version "1.1.0" 
    kotlin("jvm") version "1.7.22"
    kotlin("plugin.spring") version "1.7.22"
    kotlin("plugin.jpa") version "1.7.22"
}
```

### org.springframework.boot

* Spring Boot 애플리케이션을 빌드할 수 있도록 지원하는 플러그인
* &#x20;애플리케이션을 패키징하고 실행 가능한 JAR 파일을 생성하는 데 사용된다

### io.spring.dependency-management

* 의존성 관리를 담당하는 플러그인으로 사용하는 스프링 버전과 알맞고 검증된 의존성들을 알고있다.
* 따라서 의존성에 버전을 따로 명시하지 않아도 자동으로 알맞은 버전으로 의존하도록 해준다.&#x20;

```gradle
implementation("org.springframework.boot:spring-boot-starter-web") //버전을 명시하지 않음 
```

* kotlin("jvm")
  * 코틀린 코드가 JVM 바이트코드로 컴파일하도록 하는 플러그인

### kotlin("plugin.spring")

Spring은 AOP 기술을 사용할 때 기본적으로 CGLIB(Code Generation Library)를 이용해 Proxy를 만든다. CGLIB는 대상 클래스를 상속해 Proxy 클래스를 만드는 방식이기 때문에 반드시 상속이 가능해야한다.&#x20;

하지만 코틀린은 클래스가 기본적으로 final이기 때문에 필요한 클래스에 모두 일일히 open 붙이는 것은 끔직한 일이다.&#x20;

이 플러그인은 아래의 애너테이션이 명시된 클래스를 자동으로 open 변경자를 추가해준다.

* [`@Component`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Component.html)
* [`@Async`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/annotation/Async.html)
* [`@Transactional`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html)
* [`@Cacheable`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/annotation/Cacheable.html)
* [`@SpringBootTest`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/context/SpringBootTest.html)

{% embed url="https://kotlinlang.org/docs/all-open-plugin.html" %}

### kotlin("plugin.jpa")

jpa는 엔티티를 영속하거나 db로부터 받은 정보로 엔티티를 생성할 때 기본 생성자(no-arguement)를 이용한다.&#x20;

코틀린에서는 주 생성자의 모든 파라미터에 기본 파라미터 값을 넣어주거나 아무것도 받지 않는 construcotr를 생성해 기본 생성자를 생성할 수 있다. 이 때 주 생성자를 많이 사용하는데 기본 생성자를 만들기 위해 모든 엔티티의 프로퍼티의 기본 파라미터 값을 넣어준다는 것은 끔직한 일이다.&#x20;

이 플러그인은 `@Entity`, `@MappedSuperclass,` `@Embeddable`클래스에 자동으로 기본 생성자를 추가해준다. (실제로 코드상에서는 사용할 순 없음)

{% embed url="https://spring.io/guides/tutorials/spring-boot-kotlin/" %}

### entity에 open 적용하기&#x20;

위의 kotlin("plugin.spring") 플러그인들이 필요한 곳에 자동으로 open으로 변경해준 것처럼, entity 관련 class들에서도 open으로 변경할 필요가 있다. 왜냐하면 hibernate는 lazy-loading과 다른 최적화 작업을 위해 상속을 이용한 proxy를 생성하기 때문이다. 따라서 아래와 같이 allOpen 대상에 entity 관련 클래스들을 추가해준다.&#x20;

```gradle
allOpen {
    // javax to jakarta 
    annotation("jakarta.persistence.Entity")
    annotation("jakarta.persistence.MappedSuperclass")
    annotation("jakarta.persistence.Embeddable")
}
```



## Spring boot  의존성

```gradle
dependencies {
    // Spring
    implementation("org.springframework.boot:spring-boot-starter")
    implementation("org.springframework.boot:spring-boot-starter-web")
    annotationProcessor("org.springframework.boot:spring-boot-configuration-processor")

    // Spring Data
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    runtimeOnly("mysql:mysql-connector-java")
    implementation("com.h2database:h2")

    // Kotlin
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
    implementation("org.jetbrains.kotlin:kotlin-reflect")

    // Test
    testImplementation("org.springframework.boot:spring-boot-starter-test")

    // Util
    implementation("org.modelmapper:modelmapper:3.1.0")
}
```

위의 의존성 중 가장 필수적인 의존성만 간략히 알아보자.



### spring-boot-starter

가장 중요한 스프링 프레임워크뿐 아니라 테스팅과 로깅, auto-configuration 등 부트 코어 의존성을 제공한다.&#x20;



### spring-boot-starter-data-jpa

jpa를 더 편하게 사용할 수 있게 도와주는 data jpa뿐 아니라 Hibernate , jdbc 드라이버를 제공해준다.



### spring-boot-starter-web

mvc , restful, embedded tomcat 기능을 제공한다.&#x20;



### spring-boot-starter-test

* `@SpringBootTest` 주석은 전체 Spring 애플리케이션 컨텍스트를 로드하고 통합 테스트 스타일에서 Spring Boot 애플리케이션을 테스트하는 방법을 제공한다.
* `@MockBean` 주석: 테스트를 위한 모의 개체 생성을 허용한다.
* `TestRestTemplate` 클래스는 RESTful 웹을 테스트하는 편리한 방법을 제공해준다.
* `@DataJpaTest` 주석 - JPA 테스트를 위한 관련 구성 요소만으로 Spring 애플리케이션 컨텍스트를 구성해준다.



## Kotlin Null Check&#x20;

코틀린에서는 기본 null 가능성에 대한 기능과 더불어 `@Nullable`과 `@NotNull 애노테이션을` 제공하며, 이러한 애노테이션은 JSR-305에서 정의한 애노테이션과 호환된다.&#x20;

코틀린은 자바 API를 플랫폼 타입으로 받아들이지만, jsr305에서 정의한 널 가능성에 관련 애노테이션이 명시되어 있다면 이것을 토대로 널 가능성을 체크할 수 있다. Spring API는 jsr305 애너테이션을 이용해 널 가능성을 지원하고 있다. 따라서 아래의 옵션을 활성시켜 자바로 작성된 Spring API도 코틀른의 널 가능성 기능을 온전히 사용할 수 있다.

```gradle
tasks.withType<KotlinCompile> {
    kotlinOptions {
        freeCompilerArgs = listOf("-Xjsr305=strict")
        jvmTarget = "17"
    }
}
```





## 기타&#x20;

* &#x20;com.fasterxml.jackson.module:jackson-module-kotlin: 코틀린 객체를 직렬화하고 역직렬화할 필요한 기본 생성자를 자동으로 만들어준다.
* org.jetbrains.kotlin:kotlin-reflect: reflection 라이브러리&#x20;





## 참고

{% embed url="https://spring.io/guides/tutorials/spring-boot-kotlin/" %}

