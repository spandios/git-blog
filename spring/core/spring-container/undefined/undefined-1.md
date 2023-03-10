# 빈 스코프

## 빈 스코프란?

스프링 빈이 유지 되는 범위를 뜻한다.



## 사용법

```java
@Scope("") <- 지정할 스코프 종류를 문자열로 입력한다. 
@Component
public class ScopeBean{}
```



## 스코프 범위

### 1. Singleton

* 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프.
* 가장 많이 사용되며 미설정 시 기본으로 적용된다.

### 2. Prototype

* 컨테이너에 요청이 들어올 때마다 새로 빈을 생성해서 반환한다.
* 빈을 생성하고, 의존관계 주입, 초기화까지만 처리하지 그 이후는 관리하지 않는다. 따라서 @PostDestory 메서드가 호출되지 않는다.

{% hint style="info" %}
**싱글톤 빈이 프로토타입 스코프 빈을 의존할 때의 문제점과 해결법**

**문제점**&#x20;

* 요청이 싱글톤 빈에게 가고 싱글톤 빈은 프로토타입 스코프의 빈을 사용하게 된다.
* 싱글톤 빈 입장에서는 사용할 빈이 프로토타입인지는 관심이 없다.&#x20;
* 자신의 역할 즉, 딱 처음 초기화하고 이것을 유지하는 것에 관심이 있다.
* 따라서 싱글톤 빈은 자신의 맡은 역할대로 이미 생성된 프로토타입 빈을 계속 사용한다.&#x20;
* 이것은 프로토타입 스코프 빈이 요청을 받을 때 마다 새로 빈을 생성하고 반환을 기대하는 것과 다른 결과다.
* 이 때의 프로토타입 스코프 빈은 요청을 받는 게 아니라 사용 되어지는 것이기 때문이다.



**해결법**

* Dependency Lookup(DL) 의존관계 탐색을 통해 직접 요청하고 사용한다.
* ObjectProvider를 사용해 직접 bean을 요청하고 사용한다.
{% endhint %}

### 3. Request

* 웹 관련 스코프로 웹 요청이 들어오고 나갈때 까지 유지되는 스코프이다.



## Proxy&#x20;

어떤 빈의 진짜 조회를 지연 시키고 싶다면 proxy를 사용해보자.

* CGLIB를 통해 클래스를 상속 받은 가짜 프록시 객체를 만들어 주입한다.
* 실제로 요청이 들어오면 그 때서야 진짜 빈을 조회하고 반환한다.



