# 실행 컨텍스트

## 실행 컨텍스트란

* 실행 컨텍스트는 **코드 실행에 필요한 환경 정보**를 모아놓은 객체이다.&#x20;
* 코드가 실행할 때 컨텍스트를 구성하고, 이를 스택 자료구조인 콜 스택에 쌓는다. 함수가 종료되면 스택에서 제거가 된다.
* 실행 컨텍스트를 구성하는 종류로는 전역, 함수, eval 등이 있다.



## 수집 정보&#x20;

실행 컨텍스트에 담기는 정보로는 다음과 같다.

<img src="../../.gitbook/assets/file.excalidraw (36).svg" alt="" class="gitbook-drawing">



### **Lexical Environment(LE)**

현 컨텍스트에 식별자들에 대한 정보(environmentRecord)와 외부 환경 정보(outerEnvironment)가 있으며 변경 사항이 실시간으로 반영 된다.

#### **enviromentRecord**

현재 컨텍스트와 관련된 코드의 **식별자** 정보들이 저장된다. 함수에 지정된 매개변수 식별자, 선언한 함수 ,변수들의 식별자 등이 식별자에 해당한다. 이런 정보를 수집하기 위해 [**호이스팅**](var-const-let.md)이 일어난다.&#x20;

#### **outerEnvironmentReference**

현재 호출된 함수가 선언될 당시의 LexicalEnviorment를 참조한다. 즉, 현 컨텍스트 바로 **전 단계인 상위** 컨텍스트의 Lexcial Environment를 바라보고 있다.&#x20;

{% hint style="info" %}
스코프 체인&#x20;

스코프란 식별자에 대한 유효 범위이며 어떤 식별자를 찾기 위해 가까운 LE의 enviormentRecord에서 찾고 만약 없으면 outerEnvironmnetReference에 접근해 상위 environmentRecord에서 다시 찾는 것을 **스코프 체인**이라 한다.

최상위 outerEnviormentReference까지 찾고 찾지 못하면 undefined를 내놓는다.



{% code lineNumbers="true" %}
```javascript
const a = 'ga';
const b = 'b';
function test(){
    const a = 'a'; 
    console.log(a);
    console.log(b);
}
test();
```
{% endcode %}

5 line-> log 결과로 'a'가 호출 되는데 가장 가까운 LE enviormentRecord에서 a를 찾을 수 있기 때문이다.&#x20;

6 line -> test 실행 컨텍스트의 내부 enviornmentRecord에서 b 식별자를 찾을 수 없기 때문에 outerEnviornmentReference에서 찾으며 b가 있기 때문에 값 'b'가 출력된다.
{% endhint %}



### This Binding

this 키워드가 참조하는 객체를 담고 있다.&#x20;

this는 함수를 호출할 때 결정되기 때문에, 해당 함수를 호출하는 구문 앞에 점 또는 대괄호가 있는지를 잘 파악해야한다.

아래 코드를 보면 메서드 호출, 일반 함수 호출, 화살표 함수에서 this가 무엇을 바라보고 있는지를 알 수 있다.

{% code lineNumbers="true" %}
```javascript
const obj1 = {
    outer: function(){
        console.log(this);  // 14라인 obj1.로 호출되기 때문에 this는 obj를 가르킨다. 
        const innerFunc = function(){
            console.log(this); // 7라인 .이 없다 따라서 전역객체를 가르킨다.
        }
        innerFunc(); 
        const innerFunc2 = () =>{
        // 화살표 함수는 thisBinding을 가지고 있지 않기 때문에 상위 스코프의 thisBinding을 가르킨다.
        // 따라서 obj를 가르킨다.
            console.log(this);  
        }
        innerFunc2();
    }
}

obj1.outer();
```
{% endcode %}

콜백 함수 내부에서의 this는 콜백 함수를 넘겨받은 함수가 정의하기 따라 다르다는 것을 인지하자.

{% hint style="info" %}
#### 명시적 this binding

this를 수동으로 바꾸고 싶다면 다음 3가지 메서드 중 하나를 이용하면 된다.&#x20;

1. **call**(this, arg, arg2): call은 함수를 즉시 실행하며 첫번째 인자로 this를  받고 나머지 인자로는 호출할 함수의 매개변수로 대응된다.
2. **apply(**this,\[arg,arg2]): call과 동일하지만, 두번째 인자를 배열로 받아 배개변수로 지정한다는 것에 차이가 있다.
3. **bind**(this, arg,arg2): call과 동일하지만, 바로 실행되지 않고 바인딩 된 새 함수를 리턴한다.
{% endhint %}



### Variable Environment

LE와 같은 정보를 가지고 있지만 LE는 최신 정보를 반영하지만 이 환경 정보는 선언 당시의 값을 계속 가지고 있는다.





