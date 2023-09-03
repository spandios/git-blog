# 호이스팅 그리고 const let

## 호이스팅이란

호이스팅은 실행 컨텍스트의 enviormentRecord의 정보를 수집하는데 발생하는 현상이다.

enviormentRecord는 각종 식별자를 수집하기 때문에 코드 실행 전부터 변수명이나 함수명을 알고 있는 것과 같다.

그렇기에 실제 자바스크립트 엔진이 이런 정보를 위로 올리는 것은 아니지만 코드 최상단에 있는 것과 같다고 생각하면 호이스팅이란 개념이 이해가 간다.&#x20;

아래 코드를 실행하면 아직 변수를 정의하지 않았는데도 오류 없이 undefined를 출력한다. 호이스팅이 일어나 변수 a라는 식별자를 이미 알기 때문이다.

```javascript
function test(){
    console.log(a); // undefined;
    var a = 3;
}

// hoisting이 된다면?
function test(){
    var a;
    
    console.log(a);
    a=3;
}
```



### 함수 선언문

함수 선언문은 별도 할당 없이 함수를 정의하는 방식이다. 이 함수 선언문도 호이스팅이 일어나 선언 자체를 끌어올리게 된다.

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



이 처럼 호이스팅은 이해할 순 있지만 실제 개발하기에는 직관적으로 좋지 않다.&#x20;

## const와 let

위의 혼란을 해결하기 위해 ES6부터 등장한 const let 키워드가 등장했다.  aewffawefwe

()/eeweeawfaeefowefkoooakfpokfaewokeowkoea



ewafawefwe

### 호이스팅&#x20;

```javascript
function test(){
    console.log(a); // ReferenceError: Cannot access 'a' before initialization
    const a = 3;
}
```

const와 let은 var와 다르게 값 할당 전 접근하려하면 에러가 발생한다.

그렇다면 호이스팅이 일어나지 않는 것일까? 결론부터 말하자면 아니다.&#x20;

JavaScript 변수는 총 3단계에 걸쳐 생성된다.

###

