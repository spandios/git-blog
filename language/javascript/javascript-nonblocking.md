# Javascript 엔진과 NonBlocking 동작원리

### 자바스크립트 엔진

#### V8

javascript는 태생적으로 인터프리터 언어이다. 따로 컴파일이 필요없고 개발 편의성이 좋은 인터프리터 방식이 별 기능이 없던 옛날 브라우저에서는 좋은 선택이라고 생각한다. 하지만 점점 자바스크립트는 더 빠르고 고성능의 작업이 필요로 하게 된다.&#x20;

때문에 구글은 2009년 V8엔진을 출시하였고 지금의 크롬에서 복잡한 javascript를 빠르게 처리하며 node.js를 이용해 백엔드까지 개발 수 있게 되었다.

V8은 c++로 작성되었고 아래의 순서로 해석하고 실행한다.

<figure><img src="https://blog.kakaocdn.net/dn/3r1iJ/btry0bNZaFJ/NNK0pj7RncBW9uucgNfnrK/img.png" alt=""><figcaption></figcaption></figure>

1. 파서에게 소스코드 전달한다.
2. 파서는 AST(Abstract Syntax Tree)추상 구문 트리로 파싱하고 Ignition에 전달한다. 추상 구문 트리는 컴퓨터가 이해하기 쉽게 구조화한 자료구조이다.
3. Ignition은 받은 트리는 좀 더 쉽게 바이트 코드로 컴파일 한다.

v8엔진은 위의 과정대로 자바스크립트 코드 전체를 미리 바이트 코드로 변환하여 빠르게 실행할 수 있게 한다.

여기서 더 나아가 자주 사용되는 코드는 TurboFan으로 보내져 더욱 최적화된 코드로 컴파일되고 사용한다.

\


#### 엔진 구성요소

* Heap

코드 실행에 필요한 변수, 함수 등이 저장되어 있는 메모리 장소이다.

* CallStack

콜스택은 코드를 읽어내려가면서 실행할 작업을 쌓는다. 이 후 가장 최근에 쌓인 작업부터 실제로 코드를 실행하고 완료되면 스택에서 제거한다. 콜 스택은 싱글 스레드로 처리가 되므로 동시에 여러가지 작업은 하지 못할 것이지만nodejs, 브라우저만 봐더라도 그렇지 않다는 것을 알 수 있다. 그것은 엔진 뿐아니라 Runtime환경이 NonBlocking로 동작 하도록 가능하게 하기 때문이다.

<figure><img src="https://blog.kakaocdn.net/dn/cwXyCO/btry0csBNcU/hJmusaz48mJGz6vHpwszqk/img.png" alt=""><figcaption></figcaption></figure>

#### &#x20;

### NonBlocking

NonBlocking은 호출된 함수가 진행 제어권을 가지지 않기 때문에 결과를 기다리지 않고 다음 코드로 진행이 계속 되는 방식이다.

따라서 싱글 스레드이지만 여러 작업이 동시에 처리되는 것처럼 보이게 한다.&#x20;

\


\


### 동작원리

먼저 자바스크립트의 환경을 알아봐야하는데 위에 알아봤던 v8엔진만으로는 비동기 작업을 처리할 수 없기 때문이다.

아래는 각각 브라우저에서의 자바스크립트 처리 환경과 node.js의 처리 환경이다.

\


<figure><img src="https://blog.kakaocdn.net/dn/bwuhfp/btryU2dHYcE/pl9BiYDHtIxcwPGRLi834K/img.png" alt=""><figcaption><p>브라우저 환경</p></figcaption></figure>

\


<figure><img src="https://blog.kakaocdn.net/dn/bthZKT/btryV2YQeW8/qc5SK0h9uqadzNPES8MVp0/img.jpg" alt=""><figcaption><p>node.js 환경</p></figcaption></figure>

\


일단 두 환경 모두 Javascript 엔진말고도 event loop, queue를 확인 할 수 있다.

브라우저 환경을 예시로 각 요소가 어떤 책임을 맡고 행동하는지 살펴보면\~

\


**1. WebAPI**

WebAPI는 브라우저에서 setTimeout이나 fetch같은 비동기 처리를 위한 함수들을 제공한다.

비동기 처리하는 코드가 CallStack에 쌓이자마자 제거되고 이 코드는 WebAPI로 이동하여 처리하게 된다.

이 쪽에서의 비동기 함수처리는 콜스택의 메인스레드에서 진행하는 것이 아니기 때문에 별개로 처리하게 된다.

\


**2. Callback Queue(task queue)**

WebAPI에서 처리가 완료된 비동기 함수의 콜백함수는 이 큐에 들어와 대기한다.사실 큐는 마이크로태스크 큐와 콜백 큐 등등 우선순위가 다른 큐들이 존재한다.

\


**3. EventLoop**

이벤트 루프는 콜 스택과 큐를 항상 살펴본다.

만약 콜 스택에 처리할 작업이 모두 비어있다면 큐를 살펴보고 만약 큐에 처리해야할 작업이 있다면 꺼내와 콜 스택에 넣고 처리한다. 이벤트루프는 싱글스레드로 처리되는 콜 스택과 여러 비동기처리가 완료된 큐를 중재하면서 NonBlocking을 가능하도록 하고 있다.

\


정리하자면, call stack은 싱글 스레드로 처리되고 있고 비동기에 대한 처리는 콜 스택과 별개로 런타임 환경의 백그라운드로 처리한다. 처리가 완료되면 콜백함수가 콜백 큐에 넣어지게 되고 이벤트루프는 콜 스택이 비어져있을 때 큐에 있는 코드를 꺼내와 콜스택에 넣고 처리한다.

\


아래는 위에서 설명한 실제 흐름의 코드와 이미지이다.

\


```
const foo = () => console.log("First");
const bar = () => setTimeout(() => console.log("Second"), 500);
const baz = () => console.log("Third");

bar();
foo();
baz();
```

<figure><img src="https://blog.kakaocdn.net/dn/UTVqq/btryU1Tp4Yy/Csj9UXYYMKcX7mH3YP62X1/img.gif" alt=""><figcaption></figcaption></figure>

\


참고

[https://evan-moon.github.io/2019/06/28/v8-analysis/](https://evan-moon.github.io/2019/06/28/v8-analysis/)

[https://blog.leehov.in/23﻿](https://blog.leehov.in/23)
