# 여러가지 코루틴

코틀린은 코루틴을 언어로 지원하는 형태가 아니라, 코루틴을 구현할 수 있는 기본 도구를 언어로 지원한다.&#x20;

코틀린 1.3부터는 기본적으로 코루틴 지원 기본 기능들을 사용할 수 있다. 하지만 기본 기능을 활용한 다양한 코루틴들은 [kotlinx.corutines](https://github.com/Kotlin/kotlinx.coroutines) 패키지에 있다. 따럿 의존관계를 통해앞에 패키지를 설치하도록 하자.&#x20;



```gradle
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.4")
}
```

이제 kotlinx-coroutines.core 모듈에 있는 몇가지 코루틴들을 알아보자. 정확히 말하면 코루틴을 만들어주는 코루틴 빌더라고 부른다. 코루틴 빌더에 원하는 동작을 람다로 넘겨서 코루틴을 만들어 실행하는 방식이다.&#x20;

## launch

* 코루틴을 job으로 반환하며, 만들어진 코루틴을 즉시 실행한다.
* 원한다면 job의 cancel메서드를 호출해 중단할 수 있다.

```kotlin
fun now() = ZonedDateTime.now().toLocalTime().truncatedTo(ChronoUnit.MILLIS)

fun log(msg:String) = println("${now()}:${Thread.currentThread()}: $msg")

fun launchInGlobalScope(){
    GlobalScope.launch { // launch의 람다 파라미터가 CorutineScrope 확장 함수 타입임
        log("coroutine started.")
    }
}

fun main() {
    log("main() stated.")
    launchInGlobalScope()
    log("launchInGlobalScope() executed")
    Thread.sleep(5000L)
    log("main() terminated.")
}

// 결과 
22:17:19.688:Thread[main,5,main]: main() stated.
22:17:19.726:Thread[main,5,main]: launchInGlobalScope() executed
// 아래 코루틴은 다른 스레드에서 실행됨
22:17:19.727:Thread[DefaultDispatcher-worker-1,5,main]: coroutine started. 
22:17:24.731:Thread[main,5,main]: main() terminated.
```

위의 코드 launch에서 유의할 점은 메인 함수와 GloablScope.lanuch가 만들어낸 코루틴이 서로 다른 스레드에서 동작하는 점이다. GlobalScope는 메인 스레드가 실행 중인 동안만 코루틴의 동작을 보장해준다. 만약 메인 스레드가 종료되면 코루틴은 동작하지 않을 것이다.

이를 방지하려면 비동기적으로 launch를 실행하거나, launch가 모두 다 실행될 때까지 기다려야 한다.&#x20;

runBlocking이라는 함수는 코루틴의 실행이 끝날 때까지 현재 스레드를 블록시킨다. runBlocking을 사용하면 스레드가 모두 main 스레드에서 동작될 것이다. 따라서 main 스레드를 계속 동작시키기 위해 작성했던 Thread.sleep을 제거해도 된다.

```kotlin
fun runBlockingexample(){
    runBlocking{
        this@CoroutineScope.launch { // 수신 객체 지정 람다
            log("coroutine started.")
        }    
    }
}
```

## 코루틴 협력 : yield

이제 진짜 다른 비동기 도구와  장점인 협력을 살펴보자. 다음 코루틴들은 서로 yield와 delay 함수를 통해 작업을 양보한다.&#x20;

```kotlin
runBlocking {
        launch {
            log("1")
            yield()
            log("3")
            yield()
            log("5")
        }
        log("after first launch")
        launch {
            log("2")
            delay(1000L)
            log("4")
            delay(1000L)
            log("6")
        }
        log("after second launch")
    }

23:41:41.266:Thread[main,5,main]: after first launch
23:41:41.275:Thread[main,5,main]: after second launch
23:41:41.275:Thread[main,5,main]: 1
23:41:41.276:Thread[main,5,main]: 2
23:41:41.278:Thread[main,5,main]: 3
23:41:41.278:Thread[main,5,main]: 5
23:41:42.283:Thread[main,5,main]: 4
23:41:43.289:Thread[main,5,main]: 6
```

`delay`는 지정한 시간 동안 다른 코루틴에게 실행을 양보한다. log("2") 다음에 delay가 호출되기 때문에 log("3")을 호출하고 yield를 호출하지만 아직 delay 시간이 남아 있어 log("5")를 호출하는 것을 볼 수 있다.



## async

* launch와 같음&#x20;
* 유일한 차이는 launch는 job을 반환하지만 async는 Deffered를 반환
* 심지어 Deffered는 job을 상속하기 때문에 launch 대신 async를 사용해도 문제가 없음
* Deffered
  * job과 차이점 : 타입 파라미터가 있는 제네릭 타입이며 안에는 await함수가 정의되어 있음
  * Deffered의 타입 파라미터는 코루틴이 반환하는 타입
  * Job은 Deffered\<Unit>이라고 생각 가능
* 따라서 async는 코드 블록을 비동기로 실행하거나 await을 사용해 결과값이 나올때까지 기다렸다가 값을 얻어낼 수 있다.&#x20;

1부터 3까지 수를 더하는 과정을 async await을 사용해 처리해보자.&#x20;

```kotlin
fun sumAll(){
    runBlocking{
        val d1 = async {delay(1000L); 1}
        log("after async(d1)")
        val d2 = async {delay(1000L); 2}
        log("after async(d2)")
        val d3 = async {delay(1000L); 3}
        log("after async(d3)")
        
        log("1+2+3 = ${d1.await() + d2.await() + d3.await()}")
    }
}

d1, d2, d3가 순서대로 log찍힘
```



<img src="../../../../.gitbook/assets/file.excalidraw (25).svg" alt="" class="gitbook-drawing">

## 코루틴 컨텍스트와 디스패처&#x20;

* launch, async 등은 모두 CoroutineScope의 확장함수
* 사실 CoroutineScope는 CoroutineContext필드 하나만 가지고 있다. CoroutineContext 필드를 launch 등의 확장 함수 내부에서 사용하기 위한 매개체 역할만을 담당한다.&#x20;
* CoroutineContext는 코루틴이 실행 중인 여러 작업과 디스패처를 저장하는 일종의 맵이다.
* 다음에 실행할 작업을 선정하고 어떻게 스레드에 배정할지에 대한 방법을 결정한다.

```kotlin
launch(context : CoroutineContext){ //여기서 컨텍스트를 정할 수 있다.

}

runBlocking{
    launch(this.coroutineContext){}
    launch(Dispachers.Default){}
    launch(newSingleThreadContext("Thread"){}
}
```

## 코루틴 빌더와 일시 중단함수

launch, async, runBlocking을 제외한 2가지 코루틴 빌더가 더 있다.&#x20;

* produce: 정해진 채널로 데이터를 스트림으로 보내는 코루틴을 만든다.&#x20;
* actor: 정해진 채널로 메시지를 받아 처리하는 액터를 코루틴으로 만든다.&#x20;

delay와 yield들은 일시 중단 함수라고 부른다. 이 두가지 외에도 일시 중단 함수들이 더 있다.

* withContext: 다른 컨텍스트로 코루틴을 전환한다.
* withTimeout: 코루틴이 정해진 시간 안에 실행되지 않으면 예외를 발생시킴
* withTimeoutOrNull: 정해진 시간안에 실행되지 않으면 null을 리턴&#x20;
* awaitAll: 모든 작업의 성공을 기다린다.&#x20;
* joinAll: 모든 작업이 끝날 때까지 현재 작업을 일시 중단시킨다.&#x20;

## suspend 키워드

* 만약 코루틴이 아니거나 일시 중단 함수가 아닌 함수에서 delay나 yield같이 일시 중단 함수를 호출하면 에러가 발생한다.&#x20;
* fun 앞에 suspend 키워드를 선언하면 일시 중단 함수를 만들 수 있다.&#x20;

