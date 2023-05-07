---
layout: post
title: thinking about SoC(separation of concerns) and dependency
date: 2023-03-01
category: thinking
---

# about SoC(separation of concerns)

 module의 책임과 역할이 명확히 하는 것이다. 하나의 module이 하나의 책임과 역할을 가져야한다. module이 여러개의 책임과 역할을 가진다면 system의 complexity는 높아지고 err 발생 가능성을 잠재하게 된다.

 예를 들어 A라는 BFF(Back end For Front) server가 resource server의 역할을 함께 한다고 가정해보자. 그러면 B라는 BFF server가 resource 접근을 위해서 A BFF server에 접근하게 될 수 있다. 이러한 상황에서 A BFF와 B BFF의 authentication system이 다르다면 err를 발생 할 것이다. 하지만 개발자는 해당 에러를 예측하기 힘들다. 왜냐하면 개발자가 B BFF server를 통해서 A BFF server에 접근한 이유는 authentication 혹은 resource server에 대한 orchestration(authentication과 orchestration이 BFF의 대표적인 책임과 역할이다.)가 아니라 특정 resource에 접근하기 위해서 이기 때문이다. 그러므로 해당 에러에 대해 상당히 많은 개발비용을 치러야 될 수 있다. 

# about dependency

특정 정보에 대한 dependency는 낮으면 낮을 수록 좋다. system이 확장되면 system은 여러 기능들을 필요하게 되므로 dependency가 높아지는 필연성을 가진다. dependency가 높아지면 system은 복잡해진다. 그러므로 각 모듈간의 dependency를 낮추고 가능한 한 dependency library를 적게 유지해야된다.

예를 들어 MSA환경에서 micro service가 Spring security의 BearerTokenAuthenticationFilter를 사용하여 Authentication과 Authorization을 처리하는 것이 바람직한 것일까? 아니다 이는 바람직하지 않다.

예시와 같은 디자인에서는 첫번째로는 모든 micro service는 특정 IDP에 dependency를 가지게 된다. 다시 말해 만약 IDP가 교체되어 하는 요구사항이 생기면 모든 micro service에 영향을 미칠 수 있다. 그러므로 BFF 혹인 api gateway에게 JWT verify와 parsing에 대한 역할과 책임을 부여하므로서 micro service와 IDP의 dependency을 제거해야한다.

두번째로는 Spring security에 대한 dependency이다. spring security는 여러 방법의 authentication과 authorization에 대해 지원한다. 하지만 MSA환경에서 micro service는 하나의 authorization만 채택할 가능성이 높다. 그러므로 굳지 Spring security를 사용하여 dependency를 높이는 것보다 Spring MVC의 Filter와 같이 기존의 기능들을 사용하여 authorization을 처리하는 것이 바람직한 방법이다. 
