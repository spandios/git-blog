# 실행 컨텍스트

### 실행 컨텍스트란?

실행 컨텍스트란 자바스크립트 엔진이 코드를 실행하기 위해 필요한 환경을 뜻한다.\


### 실행 컨텍스트 구성 방법

실행 컨텍스트는 아래에 방법을 통해 구성되며 코드를 실행한다.

* 전역공간
* 함수
* 블록(es6+)

실행 컨텍스트는 콜 스택안에 차례로 쌓아간다. 가장 최근 쌓인 컨텍스트부터 코드를 실행하고 완료되면 컨텍스트를 제거한다.

예를들어, 만약 아래의 코드가 실행된다면 콜 스택에 해당하는 컨텍스트를 스택형식으로 쌓고 비우면서 코드를 실행한다.\


```javascript
var a = 1;
function outer() {
  function inner() {
    console.log(a); // undefined
    var a = 3;
    console.log(a); // 3
  }

  for (let i = 0; i < 10; i++) {
    console.log(i);
  }

  inner();
  console.log(a); // 1
}

outer();
console.log(a); // 1
```

\


1. 코드 실행 : \[전역컨텍스트]
2. outer 실행: \[전역컨텍스트, outer]
3. inner 실행: \[전역컨텍스트, outer, inner]
4. inner 종료: \[전역컨텍스트, outer]
5. outer 종료: \[전역컨텍스트]
6. 코드 종료 : \[]



### 실행 컨텍스트 구성요소

아래의 구성요소 안에 코드 실행에 필요한 환경을 저장한다.\


#### Variable Environment

enviornmentRecord와 outerEnvironmentRefrerence로 구성되며 첫 실행 컨텍스트 생성 시 정보를 담고 코드가 실행되도 이 값을 변경하지 않는다. 초기값을 복원할 때 사용한다.



#### Lexical Environment

초기엔 Variable Enviornment를 그대로 복사하여 생성한다.

이후 코드가 실행 되면서 바뀌는 정보를 이 환경에 담는다. Variable Environment와 마찬가지로 enviornmentRecord와 outerEnvironmentRefrerence로 구성되어 있다.\


* EnviormentRecord

코드 실행 전 enviormentRecord에 필요한 식별자를 객체형식으로 담는다. 이 때 호이스팅이 발생할 것이다.



* OuterEnvironmentReference

호출된 함수가 선언 될 당시의 LexicalEnviorment를 참조한다.

즉 outer의 LexicalEnviorment이며 이 값을 통해 스코프 체인이 가능하다.

스코프 체인이란 내부부터 외부까지 lexical enviornment에서 스코프를 찾을 때까지 찾는 것이다.

1. 먼저 내부의 lexical enviornment를 찾는다.
2. 발견하지 못한다면 내부의 outerEnvironmentReference를 통해 outer의 LexicalEnviorment를 찾는다.
3. 전역 컨텍스트까지 LexicalEnviorment를 찾고 없다면 undefined를 반환한다.

OuterEnvironmentReference는 클로저도 가능하게 끔 한다.



### 클로저란?

내부함수가 외부함수의 변수를 사용하며, 외부에 전달되어 사용한다. 따라서 외부함수의 실행 컨텍스트가 종료되어도 내부함수가 사용하는 외부함수의 변수는 GC를 통해 삭제되지 않는다.\


#### ThisBinding

thisBinding에는 this로 지정된 객체를 저장한다. 따로 지정되지 않으면 전역 객체가 저장한다.

\
\


참고

[https://stackoverflow.com/questions/31219420/are-variables-declared-with-let-or-const-hoisted](https://stackoverflow.com/questions/31219420/are-variables-declared-with-let-or-const-hoisted)
