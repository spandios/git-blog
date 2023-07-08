---
description: Java Virtual Machine
---

# JVM

## JVM 개요

JVM은 JAVA를 OS에 종속되지 않고 자바 코드를 실행할 수 있게 해준다. OS가 컴파일 된 java 코드를  바로 실행하는 것이 아니라 JVM이라는 가상 도구가 자바 코드와 OS 중간에서 중개자 역할을 해주기 때문이다.

자바 소스 코드는 JDK에 포함된 Compiler(javac)에 의해 중간상태의 Java ByteCode(.class )로 컴파일 된다. 이 자바 바이트코드는 JVM이 이해할 수 있는 코드이다.

JVM은 ByteCode를 최종적으로 OS가 이해할 수 있는 기계어로 변환하는 작업을 한다.&#x20;

<img src="../../../.gitbook/assets/file.excalidraw (12).svg" alt="" class="gitbook-drawing">



## JVM의 구조와 동작 원리

JVM은 명세일뿐 Vendor마다 그 구현이 다르다. 일반적인 JVM 아키텍처를 알아보도록하자.

<figure><img src="../../../.gitbook/assets/스크린샷 2023-03-18 오전 12.13.07.png" alt=""><figcaption><p>JVM Architecture</p></figcaption></figure>

## 1. ClassLoader Subsystem

클래스 로더는 컴파일된 class 파일(.class)을 메모리에 적재하는 작업을 주요로 한다.

&#x20;클래스 파일은 컴파일 타임이 아니라 런타임 시점에 처음으로 참조 되어질 때서야 **로드, 링크, 초기화**된다.

클래스로더의 각 요소에 대해 알아보자.

### 1.1 Loading

일반적으로 클래스 로딩 프로세스는 메인 클래스를 로드하는 것부터 시작되며 모든 후속 클래스 로딩 작업은 다음과 같은 작업을 할 때이다.&#x20;

* 바이트코드가 클래스에 정적 참조를 할 때
* 바이트코드가 새 객체를 생성할 때



클래스 로더는 4가지 원칙으로 지키며 역할을 수행한다.&#x20;

1. _**Visibility Principle**_

자식 클래스로더는 부모 클래스 로더에 의해 로딩된 클래스를 볼 수 있지만, 부모 클래스로더는 자식 클래스로더에 의해 로딩된 클래스를 찾을 수 없다.

2. _**Uniqueness Principle**_

부모 클래스로더에 로딩된 클래스는 자식 클래스로더에 다시 로딩되지 않아 고유하다.

3. _**Delegation Hierarchy Principle**_

클래스로더는 위의 두 원칙을 지키기 위해 총 3가지의 클래스로더를 계층 구조를 사용해 로딩을 한다.&#x20;

* Bootstrap ClassLoader: 핵심 JAVA 모듈을 로드하는 이 클래스로더는 가장 최상위 계층이며 다른 클래스로더와 다르게 c, c++ 같은 네이티브 언어로 되어 있다.
* Platform ClassLoader: Java SE 플랫폼 처럼 특정 플랫폼에 종속적인 클래스를 로드하는데 사용된다.
* System ClassLoader: 클래스패스, 모듈 패스에 있는 클래스를 로딩한다. 여기서 개발자가 작성한 대부분의 클래스를 로드한다.&#x20;



계층 구조로 클래스를 찾는 요청 흐름은 다음과 같다. 가장 먼저 요청이 들어오는 곳은 가장 하위계층인 System 클래스로더이다. 그런데 여기서 요청된 클래스를 찾는 것이 아니라 상위 계층인 Platform 클래스로더에게 위임한다. Platform 클래스로더는 다시 Bootstrap 클래스로더에게 위임하게 된다.&#x20;

Bootstrap 클래스로더 부터 클래스를 찾게 되고 찾으면 반환한다. 만약 없다면 다음 하위 계층인 Platform 클래스로더에서 찾는다.&#x20;

Platform 클래스로더가 찾으면 반환하고 없으면 다음 하위 계층인 System 클래스로더에서 찾게 되며 여기서까지도 못찾으면 ClassNotFoundException이 발생한다.

<img src="../../../.gitbook/assets/file.excalidraw (31).svg" alt="" class="gitbook-drawing">

자바 핵심 클래스가 우리가 작성한 클래스를 의존하진 않으니 로더부터 이런 계층구조를 가지는 것이 좋은 구조인 것 같다.



4. _**No Unloading Principle**_

클래스로더는 로드된 클래스를 언로드할 수 없다.



### 1.2 Linking

링크는 로드된 클래스가 초기화 전에 올바른지 검증한다.&#x20;

이 프로세스가 클래스 로드 프로세스 중 가장 오래걸리고 복잡하지만 바이트코드를 실행할 때 다시 검증할 필요가 없어 전체적으로 봤을 때 더 효율적이다.&#x20;



### 1.3 Initialization

이 단계에서 로드 된 각 클래스 또는 인터페이스의 초기화가 이루어진다.&#x20;

예를 들어, 클래스의 생성자가 호출되고 클래스에 정적 변수에 값이 할당되고 정적 블록이 실행되는 마지막 단계이다.



## 2. Runtime Data Area&#x20;

이 영역은 JVM 프로그램이 OS에서 실행될 때의 메모리 영역이다. 메모리 영역은 다시 메서드(Method) 영역, 힙(Heap) 영역, 스택(Stack) 영역, 그리고 네이티브(Native) 메소드 스택 영역으로 구분된다.



<figure><img src="../../../.gitbook/assets/스크린샷 2023-03-18 오후 5.00.24.png" alt=""><figcaption><p>Runetime Data Area</p></figcaption></figure>

### 2.1 MethodArea

JVM이 클래스를 로드할 때 해당 클래스의 정보들을 저장하는 영역이다. 즉, **클래스 수준**의 데이터를 저장한다. 이러한 정보는 실행 도중 업데이트 되지 않으며 GC에 대상도 아니고 클래스 로더로부터 언로드 되지 않는다.&#x20;

* 클래스 정보
* 필드 데이터: 필드 이름, 유형, 수정자..
* 메서드 데이터: 메스드의 이름, 반환 형식, 파라미터...
* 메서드 코드: 바이트 코드, 지역변수...

{% hint style="info" %}
JVM 메서드 영역은 다른 영역과는 달리 크기가 제한되어 있다.

클래스 정보가 많아지면 OutOfMemoryError가 발생할 수 있다.
{% endhint %}



### 2.2 Heap&#x20;

모든 객체와 해당 인스턴스 변수 및 배열의 정보를 저장하는 영역이다. 이곳도 스레드 간에 공유되는 공유 자원이다.  CG에 의해 관리되며 JVM 중 가장 큰 메모리 영역이다.&#x20;

Heap 영역은 다시 Old Generation과 Young Generation 영역으로 구분할 수 있으며 더 자세한 메모리 구조는 아래 글에서 알아본다.&#x20;

{% content-ref url="java-memory.md" %}
[java-memory.md](java-memory.md)
{% endcontent-ref %}



### 2.3 Stack&#x20;

모든 JVM 안의 스레드는 메소드 호출을 하기 위해 각각의 고유한 Stack을 가진다. 메서드 호출 때 마다 스택이 생성 되고 JVM 스택의 가장 위에 쌓인다.  이런 스택에는 지역 변수 배열, 피연산자 스택, 프레임데이터를 보유하고 있으며, 이를 스택 프레임이라고 한다. 각 메서드가 정상적으로 마무리되거나 예외가 throw되면 스택 프레임은 POP된다. 스택 영역은 공유자원이 아니기 때문에 스레드에 안전하다.&#x20;

![](<../../../.gitbook/assets/스크린샷 2023-03-18 오후 5.33.28.png>)

스택 프레임은 아래 목록의 데이터를 보유하고 있다.

* 지역 변수 배열: 메서드에서 사용될 지역 변수를 저장한다. 클래스 인스턴스의 참조, 매개변수, 메서드의 지역변수를 순서대로 배열에 저장한다.&#x20;
* 피연산자 스택: 피연산자 스택은 산술 및 논리 연산, 비교 및 ​​메서드 호출에서 얻은 값과 같은 계산의 중간 결과를 유지한다.
* 프레임 데이터: 실행 중인 메서드에 대한 메타데이터를 포함한다. 메서드 호출 및 반환을 관리하고 실행 중 프로그램 상태를 추적하는 역할을 한다.



### **2.4 PC Register**&#x20;

각 스레드마다 현재 실행중인 명령어의 주소(메소드 영역의 메모리 주소)를 보유한다. Thread가 어떤 부분을 어떤 명령을 실행해야할지 알수있어진다.&#x20;



### 5.5 Native Method Stack <a href="#c9bb" id="c9bb"></a>

JNI를 통해 호출된 네이티브 메서드 정보(주로 C/C++로 작성됨)를 저장하기 위해 별도의 스택이다.&#x20;



## 3. 실행 엔진&#x20;

실행 엔진은 위의 런타임 데이터 영역에 할당된 데이터를 읽어 바이트 코드의 명령을 한 줄씩 실행한다.

<figure><img src="../../../.gitbook/assets/스크린샷 2023-03-18 오후 5.54.31.png" alt=""><figcaption></figcaption></figure>

### 3.1 Interpreter

자바 바이트코드를 바로 해석하고 명령들을 하나씩 실행한다. 하지만 인터프리터 단점 그대로 똑같은 메서드를 매번 일일히 해석해야 하기 때문에 Compiler 방식보다 느리다는 단점이 있다.



### 3.2 Just-In-Time Compiler&#x20;

JIT 컴파일러는 전체 바이트 코드를 기계어로 변환한뒤 반복되는 코드를 찾고,해당 코드를 기계어로 변환하여 캐싱해둔다. 변환된 기계어는 인터프리터가 해석하지 않아도 되기 때문에 효율적으로 실행할 수 있다.

JIT는 계속해서 프로파일링해서 자주 사용되는 메서드 등 최적화가 필요한 부분을 찾아낸다. 따라서 기계어로 컴파일된 메서드가 시간이 지남에 따라 자주 사용되지 않는다면 캐시에서 해당 기계어를 삭제하고 인터프리터로 실행할 것이다.&#x20;

따라서 JAVA는 인터프리터 언어이자 컴파일러 언어이다.



### 3.3 Garbage Collector

사용하지 않는 객체를 개발자 대신 알아서 찾아주고 메모리에서 해제한다. 사용하지 않은 객체를 찾기 위해 여러 알고리즘을 사용해 힙을 주기적으로 검사한다. GC에 대해 더 자세한 사항은 아래 글에서 알아본다.

{% content-ref url="gabagecollection.md" %}
[gabagecollection.md](gabagecollection.md)
{% endcontent-ref %}



## 4. Java Native Interface (JNI) <a href="#a056" id="a056"></a>

JVM이 C / C ++ 라이브러리를 호출하기 위한 인터페이스



## 5. Native Method Libraries

실행 엔진에 필요한 C/C ++ 네이티브 라이브러리





### 참고

{% embed url="https://medium.com/platform-engineer/understanding-jvm-architecture-22c0ddf09722" %}

{% embed url="https://doozi0316.tistory.com/entry/1%EC%A3%BC%EC%B0%A8-JVM%EC%9D%80-%EB%AC%B4%EC%97%87%EC%9D%B4%EB%A9%B0-%EC%9E%90%EB%B0%94-%EC%BD%94%EB%93%9C%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%8B%A4%ED%96%89%ED%95%98%EB%8A%94-%EA%B2%83%EC%9D%B8%EA%B0%80" %}

{% embed url="https://homoefficio.github.io/2018/10/13/Java-%ED%81%B4%EB%9E%98%EC%8A%A4%EB%A1%9C%EB%8D%94-%ED%9B%91%EC%96%B4%EB%B3%B4%EA%B8%B0/" %}
