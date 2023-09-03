# 호이스팅 그리고 var const let

## 호이스팅이란

호이스팅은 실행 컨텍스트 enviormentRecord의 정보를 수집하는데 발생하는 현상이다.

enviormentRecord는 각종 식별자를 수집하기 때문에 코드 실행 전부터 변수명이나 함수명을 알고 있는 것과 같다.

실제 자바스크립트 엔진이 이런 정보를 위로 올리는 것은 아니지만, 코드 실행 전에도 식별자를 이미 안다는 것은 코드 최상단에 이미 식별자가 있다는 것으로 생각할 수 있다. 따라서 호이스팅이란 끌어 올린다는 뜻이 이해가 된다.

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

위에서 보았듯이&#x20;



## const와 let

var의 문제를 해결하기 위해 ES6부터 등장한 const let 키워드가 등장한다. &#x20;

{% hint style="info" %}
호이스팅 말고도 var는 함수 레벨 스코프, 중복 변수 선언 같이 많은 문제점이 있다.
{% endhint %}

```javascript
function test(){
    console.log(a); // ReferenceError: Cannot access 'a' before initialization
    const a = 3;
}
```

위의 코드를 보면 const와 let은 var와 다르게 값 할당 전 접근하려하면 친절히 에러를 호출해준다.

그렇다면 const, let은 호이스팅이 일어나지 않는 것일까? 정답은 아니다.



### TDZ(Temporal Dead Zone, 일시적 사각 지대)

JavaScript 변수는 총 3단계에 걸쳐 생성된다.&#x20;

1. 선언:  변수 이름만 만든다.
2. 초기화: 메모리에 공간을 할당한다. 이 때 초기 값은 undefined이다.
3. 할당: 실제 값을 할당한다.

var는 호이스팅 시 선언과 동시에 초기화까지 진행된다. 그래서 실제 값을 할당하기 전에 접근해도 undefined를 가지고 있는 것이다.

하지만 const와 let은 선언 부분만 호이스팅되고 초기화, 할당 부분은 호이스팅에서는 진행 되지 않는다.&#x20;

이를 **TDZ**에 있다고 하며 이 구간에 있을 때 접근하면 var와 달리 오류가 발생하게 하게 하는 것으로 보완한 것이다.



### const 특징&#x20;







### let 특징



###

