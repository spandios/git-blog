---
description: aspect oriented programming
---

# AOP

## AOP

AOP(aspect oriented programming)는 객체지향을 보완하기 위해 탄생된 프로그래밍 방식이다.&#x20;

부가기능들은 대체로 여러 곳에서 적용되는 기능들이다(횡단 관심사). 그런데 OOP로 각 기능들을 모듈화 하여 개발하는 것은 무리가 아니지만, 부가 기능을 따로 모듈화는 작업은 많은 어려움이 있다. 아무리 결합도를 낮추더라도 부가 기능 존재 자체가 독립적이지 않고 핵심 기능에 의존하고 있기 때문이다.&#x20;

이 때 AOP는 부가기능을 핵심기능에서 분리해 모듈화 할 수 있게 해준다. 그로 인해 관점이 다른 두 기능을 분리 할 수 있어, 원래라면 핵심 기능에 침투할 수 밖에 없는 부가기능 코드를 핵심 기능 코드에서 깔끔히 제거할 수 있다.&#x20;

이렇게 AOP를 사용하면 다른 관점의 코드를 서로 분리할 수 있기 때문에 기존 핵심기능을 원래대로 객체지향적으로 설계할 수 있다. 부가기능도 한 핵심기능에 의존하지 않기 때문에 여러 곳에서 얼마든지 재사용이 가능하고 기능 수정이 주는 영향이 최소화 된다.

스프링은 어떻게 이렇게 강력한 AOP 기술을 제공할 수 있었는지 내부 기술과 원리를 알아보도록 하자.
