---
description: >-
  quals, hashCode, toString, clone는 모두 재정의를 염두에 두고 설계된 것이다. 따라서 해당 메서드를 오버라이딩할
  때는 규약에 맞게 사용해야한다.
---

# 모든 객체의 공통 메서드

## 10. equals는 일반 규약을 지켜 재정의하자

equals 문제를 회피하는 가장 쉬운길은 아예 재정의 하지 않는 것이다.

재정의 해야할 때는 객체 식별성이아닌 논리적 동치성을 확인해야할 때이다.

이는 주로 값 클래스에서 나타난다. (**값 클래스:** Integer나 String처럼 값을 표현하는 클래스)

즉, 클래스가 아닌 **값**을 비교하고 확인할 때 사용된다.

**동치관계**

* 반사성 : x.equlas(x)는 true
* 대칭성 : x.equlas(y)면 y.equlas(x)도 true다
* 추이성 : x.equals(y)가 true이고 y.equls(z)가 true면 x.equlas(z)도 true이다.
* 일관성 : x.equlas(y)를 반복해서 사용하면 항상 true거나 false이다.
* null-아님 : x.eauls(null)은 항상 false이다.

## 11. equlas를 재정의한다면 hashCode도 재정의하자

* hashmap 같이 해시테이블을 사용한다면 equal이 true라도 hash code가 달라 예상한 값을 얻을 수 없다.
* 귀찮은 작업이 있으니, ide의 도움을 받자.

## 12. toString을 항상 재정의하자

재정의하지 않으면 단지 클래스 이름과@해시코드의 스트링을 보여준다.

클래스를 사용하기 즐겁고 디버깅하기 쉽게 하위 객체까지 toString을 재정의하도록하자.

객체에 명확하고 읽기 좋은 형태로 반환하도록 하자.

### 13. clone 재정의는 주의하자

되도록 clone을 쓰지말고 복제기능은 생정자와 팩토리를 사용하자.

## 14. Comparable을 구현할지 고려하라

순서를 고려해야하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하자.

Comparable interface의 추상 메서드인 compareTo에서 필드의 값을 비교할 때는 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자. 왜? 자바 7부터는 박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드인 compare메서드가 추가되었기 때문이다.

```java
public class Human implements Comparable<Human>{

        Integer age;

        Human(Integer age){
            this.age = age;
        }

        @Override
        public int compareTo(Human anotherHuman) {
            return age.compareTo(anotherHuman.age); // or Integer.compare(age, o.age);
        }
    }

			Human human = new Human(8);
      Human human2 = new Human(10);
      Human human3 = new Human(12);
      Human human4 = new Human(31);
      List<Human> humans = Arrays.asList(human, human2, human3, human4);
      Collections.shuffle(humans);
      List<Human> naturalOrder = humans.stream().sorted().toList();
      List<Human> reverseOrder = humans.stream().sorted(Comparator.reverseOrder()).toList();
```

1개 이상의 필드로 정렬하기

0이 아니면 순서가 정해졌으니 return 하면 되며, 가장 중요한 기준 점부터 체크해가면서 진행하면 된다.

```java
public int comareTo(PhoneNumber pn){
	int result = Short.compare(areaCode, pn.areaCode);
  if( result == 0 ){
		result = Short.compare(prefix, pn.prefix);
		if( result == 0 ){
			result = Short.compare(lineNum, pn.lineNum);
		}
	}
	return result;
}

// lambda 방식
public class Human implements Comparable<Human>{

        Integer age;

        Integer hairCnt;

        Human(Integer age, Integer hairCnt){
            this.age = age;
            this.hairCnt = hairCnt;
        }

        private static final Comparator<Human> COMPARATOR = 
				Comparator.comparingInt((Human human)->human.age)
				.thenComparingInt(human -> human.hairCnt);

        @Override
        public int compareTo(Human o) {
            return COMPARATOR.compare(this, o);
        }
    }
```
