# 실행 컨텍스트

## 실행 컨텍스트란

* 실행 컨텍스트는 **코드 실행에 필요한 환경 정보**를 모아놓은 객체이다.&#x20;
* 코드가 실행할 때 필요한 환경 정보들을 모아 컨텍스트를 구성하고, 이를 스택 자료구조인 콜 스택에 쌓는다. 함수가 종료되면 스택에서 제거가 된다.
* 실행 컨텍스트를 구성하는 종류로는 전역, 함수, eval이 있다.



## 수집 정보&#x20;

실행 컨텍스트에 담기는 정보로는 다음과 같다.

<img src="../../.gitbook/assets/file.excalidraw (36).svg" alt="" class="gitbook-drawing">



### **Lexical Environment**

현 컨텍스트에 식별자들에 대한 정보(environmentRecord)와 외부 환경 정보(outerEnvironment)가 있으며 변경 사항이 실시간으로 반영 된다.

#### **enviromentRecord**

현재 컨텍스트와 관련된 코드의 **식별자** 정보들이 저장된다. 함수에 지정된 매개변수 식별자, 선언한 함수 ,변수들의 식별자 등이 식별자에 해당한다. 이런 정보를 수집하기 위해 [**호이스팅**](const-let.md)이 일어난다.&#x20;

#### **outerEnvironmnetReference**

현재 호출된 함수가 선언될 당시의 LexicalEnviorment를 참조한다. 즉, 현 컨텍스트 바로 **전 단계인 상위** 컨텍스트의 Lexcial Environment를 바라보고 있다.&#x20;



### This Binding

this 키워드가 참조하는 객체를 담고 있다. 자바스크립트의 this는 굉장히 햇갈리기 때문에 정리할 필요가 있다.

#### ㄷㄹㄷㅈㅁ랟ㅈ라





### Variable Environment

선L언 당시의 값을 계속 가지고 있는다.

eL

***

## 스코프 체인&#x20;

어떤 식별자를 찾기 위해 가까운 enviormentRecord에서 찾고 없으면 outerEnvironmnetReference를 통해 바깥 environmentRecord에서 찾는 것을 **스코프 체인**이라 한다.







***

