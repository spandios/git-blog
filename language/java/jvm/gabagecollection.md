# Garbage Collection

## GC란?

* 가비지 컬렉션(GC)은 개발자 대신 알아서 힙 메모리 안에 할당된 객체 중 더 이상 사용되지 않는 객체를 주기적으로 탐색하고 제거하는 프로세스이다.&#x20;
* 객체는 대부분 새로 생성되고 얼마 안가 필요 없어지는 특성이 있다. 따라서 객체 생존 기간을 기준으로 Heap 영역을 나누고(Young & Old Generation) 이것과 대응하는 GC(Minor GC & Major GC)가 작동하는 방식이다.
* c/c++과 달리 GC가 알아서 메모리를 관리해주기 때문에 편하지만 GC가 수행되는 정확한 시점을 알 수 없고, GC가 동작하는 동안에는 다른 Thread는 멈추는(Stop The World) 오버헤드가 있는 단점이 있다.
* 각 객체가 heap에서 어떻게 이동되고 GC가 발동되는지는 아래 글에서 이미 알아보았다.&#x20;

{% content-ref url="java-memory.md" %}
[java-memory.md](java-memory.md)
{% endcontent-ref %}



## GC의 동작 방식

GC를 어떻게 구현하는지에 따라 세부적인 동작은 다르겠지만 다음의 두가지 방식을 따른다.&#x20;

### Stop The World <a href="#d22d" id="d22d"></a>

GC 수행 전 프로그램 메모리를 일관되게 보기위해 자바 어플리케이션을 잠시 멈춰야한다.&#x20;

### Mark and Sweep Model <a href="#d22d" id="d22d"></a>

* Mark:  GC Root로부터 시작해 모든 객체의 참조를 파악하고 완료하면 마킹한다.
* Sweep: 마킹 되지 않은 객체 즉, 더 이상 참조되지 않은 객체를 메모리에서 제거한다.&#x20;





## GarbageCollectors

* Java는 Serial, Parallel, G1(Garbage First) 및 ZGC(Z Garbage Collector)와 같은 다양한 알고리즘을 제공한다.&#x20;



### Serial GC

mark-sweep-compact 알고리즘을 사용하는 단일 스레드 GC 알고리즘이다. Compact는 각 객체가 연속적으로 쌓일 수 있도록 힙의 앞부분부터 채워지도록 하는 작업이다.

CPU 코어가 1개일 때 사용하기 위해 개발됐고 한 개의 스레드만 사용하기 때문에 멀티 코어 환경에서 이 알고리즘을 선택하는 일은 없도록 한다.



### Parallel GC&#x20;

Serial GC와 거의 같지만 차이점은 이름 처럼 여러 스레드를 이용해 병렬로 GC를 처리한다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-03-18 오후 11.54.38.png" alt=""><figcaption></figcaption></figure>





### G1(Garbage-First) GC

* G1은 기존 Young의 영역에서 Old로 이동하는 단계를 수행하지 않는다. 대신 힙을 여러 영역으로 나누고 가비지가 많은 쪽을 가중치로 두어 우선적으로 GC를 수행한다.
* 소형 머신에서 대용량의 메모리를 갖춘 대형 멀티프로세서 머신으로 확장할 수 있도록 설계되었다.
* 현재 기본 Garbage Collector이다.



### Z 가비지 컬렉터(ZGC)

* 애플리케이션 스레드 실행을 중단하지 않고 모든 고비용 작업을 동시에 수행한다.
* ZGC는 수 밀리초의 최대 일시 중지 시간을 제공하지만 처리량이 약간 감소한다.
* 짧은 지연 시간이 필요한 애플리케이션을 위한 알고리즘







## 참고&#x20;

{% embed url="https://docs.oracle.com/en/java/javase/19/gctuning/available-collectors.html#GUID-F215A508-9E58-40B4-90A5-74E29BF3BD3C" %}

{% embed url="https://d2.naver.com/helloworld/1329" %}

{% embed url="https://medium.com/platform-engineer/understanding-java-garbage-collection-54fc9230659a" %}

{% embed url="https://mangkyu.tistory.com/118" %}
