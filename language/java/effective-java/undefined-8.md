# 동시성

## 78.공유 중인 가변 데이터는 동기화해 사용하라&#x20;

* 여러 스레드가 가변 데이터를 공유한다면 읽고 쓰는 동작은 반드시 동기화해야한다.
* synchornized 키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다.&#x20;
* 위의 동기화 기능에 한가지 더 중요한 기능은 스레드 사이의 안정적인 통신을 위해 정상적으로 저장한 값을 온전히 읽어온다는 것을 보장한다.&#x20;
* Thread.stop 메서드는 안전하지 않으므로 사용 X



### synchorinized를 통한 thread 멈추는 방법

```java
## 잘못된 코드
public class StopThread{
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException{
        Thread backgroundThread = new Thread(new Runnable(){
            @Override
            public void run(){
                int i = 0;
                while(!stopRequested){
                    i++;
                }
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true; // 1
    }
}
```

위의 코드는 boolean 값을 동기화하지 않기 때문에 1에서 메인 스레드가 수정한 값을 백그라운드 스레드가 언제 보게될지는 보증하지 않는다. 때문에 1초 후에 종료되지 않고 무한정으로 함수가 동작할 수 있다.

다음과 같이 동기화를 이용하면 정상적으로 종료된다.&#x20;

```java
public class StopThread{
    private static boolean stopRequested;

    private static synchronized void requestStop(){
        stopRequested = true;
    }

    private static synchronized boolean stopRequested(){
        return stopRequested;
    }

    public static void main(String[] args) throws InterruptedException{
        Thread backgroundThread = new Thread(new Runnable(){
            @Override
            public void run(){
                int i = 0;
                while(!stopRequested()){
                    i++;
                }
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

특히 쓰기 메서드와 읽기 메서드를 모두 동기화한점을 주목하자.&#x20;

### Volatile

* **volatile** 필드를 사용하면 synchorinized 메서드들을 생략해도 된다.&#x20;
* volatile은 동기화 중 베타적 실행(원자성)은 필요 없고, 통신(스레드 간 안정적으로 값을 조회) 쪽만 필요하다면 사용해야한다.

```java
public class StopThread{
    private static volatile boolean stopRequested;

    public static void main(String[] args) throws InterruptedException{
        Thread backgroundThread = new Thread(new Runnable(){
            @Override
            public void run(){
                int i = 0;
                while(!stopRequested){
                    i++;
                }
            }
        });
         ~~ 
    }
}

```

* 주의할 점&#x20;

아래 코드를 보면 증가 연산자를 사용하고 있기 때문에 nextSerialNumber 필드에 두 번 접근한다.&#x20;

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber(){
    return nextSerialNumber++;
}
```

만약 두 번째 스레드가 비집고 들어와 값을 읽으면 첫 번째 스레드와 똑같은 값을 돌려받을 것이다.&#x20;

따라서 메서드에 synchroized를 붙이고 volatile은 제거한다. 이제 동시에 호출해도 스레드간 서로 간섭하지 않고 이전 호출하여 변경된 값을 안정적으로 조회할 수 있다.&#x20;



* atomic 패키지에 있는 클래스들을 사용하면 락 없이도 스레드에 안전하게 개발할 수 있다. 원자성뿐아니라 통신쪽도 지원한다.&#x20;

```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber(){
    return nextSeialNum.getAndIncrement();
}
```

* 사실 이런 문제를 피할수 있는 가장 좋은 방법은 가변 데이터를 공유하지 않는 것이다.
* 가변 데이터는 단일 스레드에서만 사용하자.&#x20;





## 79. 과도한 동기화는 피하라&#x20;

* 과도한 동기화는 성능을 떨어트리고, 교착상태에 빠뜨리며, 예측할 수 없는 동작을 낳기도 한다.&#x20;
* 동기화 영역 안에서 외계인 메서드를 호출하는 등의 과도한 작업은 하지말자.&#x20;
* 동기화 영역에선 딱 동기화가 필요한 작업만 최소한으로 수행해야한다.&#x20;



## 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라&#x20;

### 실행자&#x20;

* java.util.concurrent 패키지의 실행자 프레임워크&#x20;

```java
ExecutorService exec = Executors.newSingleThreadExectuor();
exec.execute(runnalble); //task 실행
exec.shutdown(); // 종료 
```

* 작은 프로그램이나 가벼운 서버라면 Exectuors.newCachedThreadPool을 사용하자.
* 실행자 프레임워크는 작업 단위와 실행 메커니즘이 분리된다. 반면 Thread를 직접 다루면 Thread가 작업 단위와 수행 매커니즘 역할을 모두 수행한다.&#x20;
* 태스크는 작업 단위를 나타내는 핵심 수상 개념이며 태스크는 Runnable과 Callable가 있다. Callable은 값을 반환하고 임의의 예외를 던질수 있다.&#x20;
* 자바7부터 실행자는 fork-join 태스크를 지원한다. fork-join task는 일을 먼저 끝난 스레드가 다른 스레드의 남은 태스크를 처리해 효율성이 높은 태스크이다. 직접 사용하는것은 힘들지만 이것을 이용한 것이 병렬 stream이기 때문에 잘 활용해보자&#x20;



## 81. wait와 notify보다는 동시성 유틸리티를 애용하라&#x20;

* wait과 notify보다는 고수준 동시성 유틸리티를 이용하자
* java.util.concurrent는 실행자 프레임워크, 동시성 컬렉션, 동기화 장치를 지원한다.&#x20;
*   동시성 컬렉션

    이 컬렉션은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 컬렉션이다. 높은 동시성을 도달하기 위해 동기화를 각 내부 로직에서 구현한다.&#x20;
*   동기화 장치: 스레드가 다른 세리드를 기다릴 수 있게하여, 서로 작업을 조율할 수 있게 해준다.&#x20;

    ContDownLatch: 일회성 장벽으로, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날때까지 기다리게 한다.&#x20;



## 82.  스레드 안정성 수준을 문서화하라&#x20;

멀티스레드 환경에서도 API를 안전하게 사용하게 하려면 클래스가 지원하는 스레드 안정성 수준을 정확히 명시해야 한다.&#x20;

### 스레드 목록 (안정성이 높은 순)

* 불변: 이 인스턴스는 상수와 같아서. 외부 동기화가 필요가 없다. ex) String, Long, BigInteger&#x20;
* 무조건적 스레드 안전: 이 클래스의 인스턴스는 수정될 수 있으나, 내부에서 충실히 동기화하여 별도의 외부 동기화 없이 사용해도 안전하다. ex) AtomicLong, ConccurentHashMap&#x20;
* 조건부 스레드 안전: 무조건적 스레드 안전과 같으나, 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다. ex) Collections.synchorized 래퍼 메서드가 반환한 컬렉션들이 여기에 속함
* 스레드 안전하지 않음: 이 인스턴스들은 수덩죌 수 있으며, 동시에 사용하려면 클라이언트가 외부 동기화 메커니즘으로 감싸야 한다.&#x20;
* 스레드 적대적: 이 클래스는 외부 동기화를 사용하더라도 멀티스레드 환경에 안전하지 않다.&#x20;



## &#x20;83.  지연 초기화는 신중히 사용하라&#x20;

* 지연 초기화는 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법이다.
* 정적 필드와 인스턴스 필드에 모두 사용할 수 있으며, 클래스와 인스턴스 초기화 때 발생하는 위험한 순환 문제를 해결하는 효과도 있다.&#x20;
* 다른 모든 최적화와 마찬가지로 최선은 필요 할때까지 일단 하지말라이다.&#x20;
* 지연 초기화는 초기화 할 때의 비용은 줄어들지만 대신 지연 초기화하는 필드에 접근하는 비용은 커진다.&#x20;
* 지연 초기화하는 필드를 둘 이상의 스레드가 공유한다면 어떤 형태로든 동기화를 해야 한다.&#x20;
* 대부분의 상황에서는 일반적인 초기화가 지연 초기화보다 낫다.&#x20;
* 지연 초기화가 멀티스레드로 인해 버그를 일으킬 것 같다면 아래처럼 synchorinzed를 단 접근자를 이용하자&#x20;

```java
private FiledType field;

private synchronized FieldType getField(){
    if( field == null){
        field = computeFieldValue();
    }
    return field;
}

```

* 지연 초기화 홀더 클래스 관용구: 성능 때문에 정적 필드를 지연 초기화해야 할 때 사용한다. 클래스는 클래스가 처음 쓰일 때 비로소 초기화 된다.

```java
private static class FielddHolder{
    static final FieldType field = computeFieldValue();
}

private static FieldType getField() {return FieldHolder.field};
```



## 84. 프로그램 동작을 스레드 스케줄러에 기대하지 말라&#x20;

* 스레드 스케줄러 정책에 따라 프로그램이 좌지우지되지 말아야 한다. 다른 플랫폼으로 이식이 힘들다.
* 스레드의 평균적인 수를 프로세서 수 보다 지나치게 많아지게 하지말자. 스케줄러가 따로 고민할 것이 없어지기 때문이다.&#x20;







