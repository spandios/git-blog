---
description: "\b4가지 영역으로 구분된 Memory 관리"
---

# Java Memory

## 1. Heap Memory

* 모든 객체와 해당 인스턴스 변수 및 배열의 정보를 저장하는 영역이다.&#x20;
* HeapMemory는 Young Generation과 Old Generation으로 나눌 수 있다.
* Heap 메모리의 크기는 Applicaiton이 동작하면서 증가되거나 감소될 수 있다.

<figure><img src="../../../.gitbook/assets/스크린샷 2023-03-18 오후 7.46.34.png" alt=""><figcaption><p>Heap Memory</p></figcaption></figure>

### Young Generation&#x20;

* 비교적 새로 생성된 객체가 할당되는 영역으로 다시 Eden과 두개의 SurviorSpaces(S0,S1)로 나눌 수 있다.&#x20;
* 새로운 객체가 생성되면 Eden으로 할당된다. 만약 이 영역에 객체가 가득차면 Minor GC가 발동된다. GC는 사용되지 않은 객체는 메모리에서 제거하며 Eden 영역의 모든 객체들을 Survior Spaces 중 하나로 이동시킨다.&#x20;
* Survivor Space도 가득차게 되면 GC는 살아남은 객체를 다른 스페이스로 모두 옮기고 이 공간을 비운다. 때문에 SurvivorSpace 중 둘 중 하나는 반드시 비워져있다.



### Old Generation

* 여러번의 GC 이후에도 살아남은 객체는 이 영역으로 이동된다. 이 영역이 가득차게 되면 Major GC가 발동된다. Major GC는 Minor GC보다 더 오래걸린다.&#x20;



## 2. Non-Heap Memory

* 이 영역은 Metaspace(자바 8 이전은 Permanent Generation, 이제 메모리가 부족하면 자동으로 증가함)을 포함하며 주로 클래스들의 메타데이터를 저장한다.
* &#x20;MethodArea도 이 곳에 포함되는데 런타임 상수 풀, 필드 및 메서드 데이터, 메서드 및 생성자 코드, 내부 스트링과 같은 클래스별 구조를 저장하며 GC에 관리되지 않고 영구적으로 저장되는 곳이다.



## 3. Cache Memory

* JIT 컴파일러에서 생성된 컴파일된 기계어, JVM 내부 구조 등을 저장한다. 이 영역은 가득차게 되면 영역 메모리를 초기화시킨다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-03-18 오후 9.05.12.png" alt=""><figcaption></figcaption></figure>



## 4. Stack Memory

* 각 스레드가 수행하려는 기능을 위해 메서드 특정 값과 힙의 다른 객체에 대한 참조를 포함한다.&#x20;
* 힙과는 다른 곳에서 분리되어 수행되며 힙 메모리에 비해 크기가 작으며 임시 데이터를 저장하는데 사용된다.

<figure><img src="../../../.gitbook/assets/스크린샷 2023-03-18 오후 9.08.44.png" alt=""><figcaption></figcaption></figure>



{% hint style="info" %}
JAVA 메모리 관련 설정&#x20;

* \-XmsSetting — initial Heap size
* \-XmxSetting — maximum Heap size
* \-XX:NewSizeSetting — new generation heap size
* \-XX:MaxNewSizeSetting — maximum New generation heap size
* \-XX:MaxPermGenSetting — maximum size of Permanent generation
* \-XX:SurvivorRatioSetting — new heap size ratios (e.g. if Young Gen size is 10m and memory switch is –XX:SurvivorRatio=2, then 5m will be reserved for Eden space and 2.5m each for both Survivor spaces, default value = 8)
* \-XX:NewRatio — providing ratio of Old/New Gen sizes (default value = 2)
{% endhint %}

### 참고&#x20;

{% embed url="https://medium.com/platform-engineer/understanding-java-garbage-collection-54fc9230659a" %}

