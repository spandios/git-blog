# 호이스팅 그리고 var const let

## 호이스팅

호이스팅이란 자바스크립트 엔진이 코드를 실행하기 전에 소스 코드를 먼저 분석하고 처리하는 데 발생하는 현상이다.

실제 자바스크립트 엔진이 실제로 식별자를 위로 올리는 것은 아니지만, 코드 실행 전에 식별자를 이미 안다는 것은 마치 코드 최상단으로 식별자를 끌어올리는 것으로 생각할 수 있다.&#x20;

아래 코드를 실행하면 아직 변수를 정의하지 않았는데도 오류 없이 undefined를 출력한다. 호이스팅이 발생해 변수 a라는 식별자를 코드 실행 전에도 이미 알기 때문이다.

```javascript
function test(){
    console.log(a); // undefined;
    var a = 3;
}

// hoisting이 된다면?
function test(){
    var a; // 
    
    console.log(a);
    a=3;
}
```

함수 선언문도 호이스팅이 일어나 선언 자체를 끌어올리게 된다.

아래 코드를 실행하면 먼저 undefined를 출력하게 되고 그 후 3을 출력한다. 호이스팅이 일어나 function a를 알기 때문이다.

```javascript
function test(){
    console.log(a);
    var a = 3;
    function a(){
        console.log("a")
    }
    console.log(a);

}

// hoisting이 된다면?
function test(){
    var a;
    function a(){
    
    }
    
    console.log(a); // function a
    a=3;
    console.log(a); // 3
}
```



## var&#x20;

위의 코드에서는 호이스팅의 혼라스러운 점을 보여주기 위해 이제는 사용하지 않는 var를 사용했다. var는 아래와 같은 특징을 가진다.

### 호이스팅이 발생한다.

그래서 위의 예시처럼 아직 초기화되지 않은 변수에 접근해도 오류가 발생하지 않아 혼란이 발생한다.



### 스코프가 함수 레벨이다.

보통의 프로그래밍 언어와 다르게 스코프가 함수 레벨이기 때문에 혼란스러운 부분이 있다.

```javascript

function test(){
    var a = 1;
    if(a == 1){
        var a = 2;
    }
    console.log(a); //2
}
test(); 

```



### 변수를 중복 선언 가능하다.&#x20;

놀랍게도 같은 변수를 같은 스코프에서 중복 선언해도 문제가 없다.

```javascript

function test(){
    var a = 1;
    var a = 2;
    console.log(a); // 2
}
test();
```



## const와 let

위에서 봤듯이 var는 혼란스러운 점이 다수 있다. 당시 javascript 사용처가 그렇게 복잡하지 않은 환경이기 때문에 창시자가 이런 설계를 했던 것 같다.&#x20;

하지만 js의 사용처가 점점 더 복잡해짐에 따라 단점이 더 부각되면서 보완 필요성이 생겼다. 그래서 es6부터 새로운 키워드인 const와 let이 등장한다.



### 호이스팅 해결&#x20;

```javascript
function test(){
    console.log(a); // ReferenceError: Cannot access 'a' before initialization
    const a = 3;
}
```

위의 코드를 보면 const와 let은 var와 다르게 값 할당 전 접근하려하면 친절히 에러를 호출해준다.

그렇다면 const, let은 호이스팅이 일어나지 않는 것일까? 정답은 아니다.

#### TDZ(Temporal Dead Zone, 일시적 사각 지대)

JavaScript 변수는 총 3단계에 걸쳐 생성된다.&#x20;

1. 선언:  변수 이름만 만든다.
2. 초기화: 메모리에 공간을 할당한다. 이 때 초기 값은 undefined이다.
3. 할당: 실제 값을 할당한다.

var는 호이스팅 시 선언과 동시에 초기화까지 진행된다. 그래서 실제 값을 할당하기 전에 접근해도 undefined를 가지고 있는 것이다.

하지만 const와 let은 선언 부분만 호이스팅되고 초기화, 할당 부분은 호이스팅에서는 진행 되지 않는다.&#x20;

이를 **TDZ**에 있다고 하며 이 구간에 있을 때 접근하면 var와 달리 오류가 발생하게 하게 하는 것으로 보완한 것이다.



### 블록 레벨의 스코프

이제 보통의 언어처럼 const,let은 블록 레벨의 스코프를 가진다.&#x20;

```javascript
function test(){
    const a = 1;
    if(a == 1){
        const a = 2;
    }
    console.log(a); // 1
}
test();

```



### 변수를 중복 선언하지 못한다.

이제 보통의 언어처럼 같은 식별자를 중복 선언하지 못한다.

```javascript
function test(){
    const a = 1;
    const a = 1; // SyntaxError: Identifier 'a' has already been declared
}
test();

```



### const 특징&#x20;

const는 상수 값을 할당할 때 사용한다. 한번 값을 할당하면 변경하지 못하며, 반드시 선언과 함께 값을 할당해야한다.&#x20;

```javascript
const a = 1;
a = 2; // Uncaught TypeError: Assignment to constant variable.

const a; // Uncaught SyntaxError: Missing initializer in const declaration
```

데이터 영역이 바라보는 값은 불변이지만 객체의 프로퍼티까지 불변은 아닌 것을 인지하자.

```javascript
const test = { a: 1}
test.a = 2; // 가능 
const test2 = [];
test2.push(1);

test = {b : 1} // Uncaught TypeError: Assignment to constant variable.
```

만약 객체의 프로퍼티 값까지 불변으로 만들고 싶다면 Object.freeze()를 사용하자. (중첩 객체나 배열은 적용x)



### let 특징

let은 var처럼 변수를 저장할 때 사용한다. const와 다르게 값을 변경할 수 있으며 선언과 함께 값을 할당하지 않아도 된다. 이 때는 undefined로 초기값이 할당된다.

```javascript
const a = 1;
a = 2;

const b; 
console.log(b); // undefined
```



###

