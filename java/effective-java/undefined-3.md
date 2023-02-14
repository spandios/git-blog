# 제네릭

## 개요

제네릭은 자바5부터 사용할 수 있는 문법으로 지원하기 전까지는 객체를 꺼낼 때마다 형변환을 해야 했다.

그래서 형변환 오류가 종종 나곤 했지만, 제네렉의 등장 이후로 넣을 수 있는 타입을 컴파일러에게 알려줬다. 그래서 컴파일러는 알아서 형변환 코드를 추가할 수 있게 되고, 엉뚱한 타입의 객체를 넣으려는 시도를 원천 차단 가능해졌다.

### 용어정리

* 제네릭 클래스: 클래스와 인터페이스 선언에 타입 매개변수가 쓰임
* 제네릭 타입: 제네릭 클래스와 인터페이스 ex) List\<E>
* unbounded wildcard : 제네릭은 사용하고 싶지만 그 타입이 정확히 무엇인지는 신경 쓰고 싶지 않을 때 ?를 쓰자. ex) List\<?>
* upper bounded wildcard : List\<E extends Number>
* lower bounded wildcard : List\<E super Number>
* 제네릭 메서드: static \<E> List\<E> asList(E\[] a)
* recursive type bound : \<T extends Comparable\<T>>
* 타입 토큰 : ex) String.class



## 26. 로 타입은 절대 사용하지 말자.

* raw type이란 타입 매개변수가 없는 제네릭을 뜻한다.
* 로 타입은 제네릭이 주는 안정성과 표현력을 잃는다.
* 단지 호환성 때문에 존재
* List\<Object>는 모든 타입을 허용한다는 의사이지만, 로 타입인 List는 제네릭에 완전히 발을 뺀 셈이다.
* ? → wildcard 제네릭은 로 타입과 다르게 어떤 원소도(컬렉션에서) 넣을 수 없다. 따라서 타입 불변성을 지킬 수 있으며 꺼낼 수 있는 타입도 전혀 알 수 없게 해준다.



## 27. 비검사 경고를 제거하라

* 제네릭은 컴파일러가 안전하지 않은 코드에 대해 경고를 보내준다.
* 할 수 있는 한 모든 경고를 제거하자. ClassCastException을 일으킬 수 있는 잠재적 가능성을 줄여준다.
* 경고를 제거할 수는 없지만 타입 안전하다고 확신한다면 @SuppressWarnings(”uncheck”) 애너테이션을 달아 경고를 숨기자. 이 애너테이션은 가능한 좁은 범위로 사용하자.그 후 경고를 숨기기로 한 근거를 주석에 남기자.



## 28. 배열 보다는 리스트를 사용하자

제네릭은 타입 정보가 런타임에는 소거가 된다. 취지 자체가 컴파일 단계에서 컴파일러에게 타입을 알려주고 형변환과 타입 체크를 하기 위해서기 때문이다. 반면, 배열은 런타임시에도 타입을 인지하고 확인하다. 때문에 서로의 호환이 좋지 않다. 그래서 배열 보다는 리스트를 사용하는게 런타임 시 ClassCastException을 만날일이 없으니 좋다.



## 29. 이왕이면 제네릭 타입으로 만들라

클라에서 직접 형변환하는 것보다 제네릭 타입이 더 안전하고 편함

### 제네릭 배열 생성을 금지하는 제약 우회하기

`elements = (E[]) new Object[10];`



## 30. 이왕이면 제네릭 메서드로 만들자

```kotlin
public static <E> Set<E> union(Set<E> s1, Set<E> s2){
	Set<E> result = new Hashset<>(s1);
	result.addAll(s2);
	return result;
}

// 재귀적 타입 한정, 모든 타입 E는 자기 자신과 비교할 수 있음(Comparable을 구현함)
public static <E extends Comparable<E>> E max(Collection<E> c);

```



## 31. 한정적 와일드카드를 사용해 API 유연성을 높이자

매개변수화 타입은 불공변이다. 즉, 서로 다른 타입 Type1과 Type2가 있을 때 List\<Type1>은 List\<Type2>의 하위 타입도 상위 타입도 아니다.

하지만 때론 불공변 방식보다 유연할 때가 필요하다. 이 때 한정적 와일드 카드를 사용하자.

extneds: 하위 타입, super : 상위 타입

Iterable\<? extends E> : E를 포함한 E의 하위 타입

Iterable\<? s uperE> : E를 포함한 E의 상위 타입



### 공식

* 매개변수화 타입 T가 생산자면 \<? extends T>

```kotlin
//src를 통해 사용할 E 인스턴스를 사용 중 
public void pushAll(Iterable<? extends E> src){ 
	for (E e : src) 
			push(e);
}
```

* 소비자면 \<? super T>

```kotlin
// dst가 이 E 인스턴스를 소비한다.
public void popAll(Collection<? super E> dist){
	while(!isEmpty()) dst.add(pop())
}
```

```kotlin
public class Chooser<T>{
	private final List<T> choiceList;
	public Choose(Collection<? extends T> choices){
		choiceList = choices;
	}
}
// Interger는 Number의 하위타입이다.
List<Integer> intergers = listOf(1,2);
Choose<Number> chooser = new Chooser(integers); // 만약 한정적 와일드카드가 아니였다면 못 넣음. 
```

### 매개변수 vs 인수

```kotlin
fun add(value : Int){ // value => 매개변수 
}
add(10) // 실제 값 인수 

class Set<T> => T는 타입 매개변수
Set<Interger> => Interger는 타입 인수 
```

### Comparable, Comparator

이 인터페이스는 언제나 소비하기 때문에 일반적으로 Comparable\<? super E>를 사용하는게 더 낫다.

왜냐하면 이 한정적 와일드 카드를 사용하지 않는다면 만약 상위 타입이 comparable을 구현했다면 상위타입과 하위타입 둘 중 어떤 인스턴스와 Comparable를 할 지 모르기 때문이다.

1. 제네릭과 가변인수를 함께 쓸 때는 신중하자

가변인수와 제네릭은 궁합이 좋지 않다. 간변인수 기능은 배열을 노출해 추상화가 완벽하지 않고, 배열과 제네릭 타입의 규칙이 서로 다르기 때문이다.





## 33. 타입 안전 이종 컨테이너를 고려하자

* 일반적인 제네릭 형태는 한 컨테이너가 다룰 수 있는 타입 매개변수가 정해져 있다.
* 하지만 각 타입의 class 객체를 매개변수화한 키 역할로 사용하면 제약 없는 타입 안전 이종 컨테이너를 사용 가능하다.
* class의 클래스가 제네릭. class 리터럴 타입은 Class\<T>다. ex) String.class 타입은 Class\<String>이다.
* 한편, 컴파일타임 타입 정보와 런타임 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴을 **타입 토큰**이라 한다. ex) String.class

```kotlin
//api
public class Favorites{
	public <T> void putFavorite(Class<T> type, T instance);
	public <T> T getFavorite(Class<T> type);
}

// 클라이언트 
public static void main(String[] args){
	Favorites f = new Favorites();
	f.putFavorite(String.class, "Java");
	f.putFavroite(Class.class, Favorite.class);
	String favroiteString = f.getFavorite(String.class);
	Class<?> favoriteClass =f.getFavorite(Class.class);
}

// 구현 
public class Favorites{
 // 와일드카드로 어떤 클래스의 타입이 와도 된다. 
	private Map<Class<?>, Object> favorites = new HashMap<>();

	public <T> void putFavorite(Class<T> type, T instance){
		favorites.put(Objects.requireNonNull(type), instance);
	}
	
	public <T> T getFavorite(Class<T> type){
		return type.cast(favorites.get(type)); // 형변환 연산자의 동적 버전
	}
}

```
