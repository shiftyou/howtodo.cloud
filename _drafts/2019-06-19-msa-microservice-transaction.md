---
layout: post
title:  마이크로서비스에서의 트랜잭션
categories: microservice
tags: msa microservice transaction 트랜잭션 마이크로서비스 saga
---


마이크로서비스는 각 서비스가 자신만의 데이터를 가지고 있다. 게다가 데이터는 API로만 관리될 수 있고 숨겨져 있다.
기존 애플리케이션은 데이터를 사용하고 데이터는 트랜잭션을 가지게 된다.  
마이크로서비스로 구성되는 애플리케이션은 어떻게 트랜잭션을 관리하게 될까?


가장 일반적으로 두개 이상의 애플리케이션이 각각의 데이터베이스를 가지고 있다면, 이는 하나의 트랜잭션으로 묶는 Two-Phase Commit으로 관리할 수 있다.
이는 Transaction Manager가 존재하고 이 Manager가 두개 이상의 데이터베이스의 트랜잭션을 보장하는 것이다.  

하지만 마이크로서비스에서는 이러한 Transaction Manager가 존재할 수가 없다. 각 서비스가 개별적으로 동작하고 서로간에 API 호출로 불려지기 때문이다. 이로인해 마이크로서비스에서는 NoSQL을 많이 사용하고 있다.

# SAGA 패턴

분산 트랜잭션 중에 가장 널리 알려진 패턴이 SAGA 패턴이다.  
사바(SAGA)는 각 트랜잭션이 단일 서비스 내의 데이터를 갱신하는 일련의 로컬 트랜잭션이다. 첫번째 트랜잭션이 완료되고 난 후 두번째 트랜잭션은 이전의 작업 완료에 의해 트리거 되는 방식이다.

![](https://microservices.io/i/data/saga.jpg)

SAGA 패턴에는 두가지 방식이 있다.
- Event/Chreography : 각 로컬트랜잭션이 이벤트를 발생시고 다른 서비스가 트리거링 하는 방식.
- Command/Orchestration : 오케스트레이터가 어떤 트랜잭션을 수행할 건지 알려주는 방식

## Choreography-based SAGA

![](https://microservices.io/i/data/Saga_Choreography_Flow.001.jpeg)

1. Order Service가 Order를 생성시키고 pending 상태로 놔둔다. 그리고 Order Created 이벤트를 생성한다.
1. Customer Service가 Order를 위한 OrderCreated 이벤트를 받고 Credit을 생성한 후 Credit Reserved 이벤트 혹은 CreditLimitExceeded 이벤트를 생성한다.
1. Order Service는 위의 이벤트를 다시 받고 현재 pending 상태를 approved  나 cancelled  상태로 바꾼다. 


각 서비스마다 자신의 트랜잭션을 관리하며 현재 상태를 바꾼 후, 이벤트를 발생시키고, 그 이벤트를 다른 서비스에 전달하는 방식으로 트랜잭션이 처리되는 형식이다. 결과값 또한 잘 진행되지 않았을 시에는 보상 에벤트를 발생시킴으로써 트랜잭션이 관리된다.

다른 예로 살펴보면, 

![](https://blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-09-at-6.13.39-PM-768x817.png)

모든 서비스는 이벤트가 생성되고, 그 이벤트를 받아서 다른 서비스가 수행되는 형태로 이루어져 있다.

만약 호출되는 서비스의 마지막 서비스가 실패했을 경우는 보상 이벤트를 발생한다.

![](https://blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-09-at-6.36.17-PM-768x526.png)

Stock Service가 OUT_OF_STOCK_EVENT를 발생시키면 Order Service 및 Payment Service 모두가 Event를 받아서 환불하고 주문상태를 실패로 설정한다.

각 트랜잭션마다 공통된 공유 ID를 정의하는 것이 중요하므로 Event를 던질 때 마다 모든 Listener 가 참조하는 트랜잭션을 알 수 있다.

- 장점
    - 이해하기 쉽고 간단하다.
    - 구축하기 쉽다.

- 단점
    - 어떤 서비스가 어떤 이벤트를 수신하는지 추측하기 어렵다.
    - 트랜잭션이 많은 서비스를 거쳐야 할 때 상태를 인지하기 어렵다.
    - 모든 서비스는 호출되는 각 서비스의 이벤트를 리슨해야 한다.


## Orchestration-based SAGA

![](https://microservices.io/i/data/Saga_Orchestration_Flow.001.jpeg)

1. Order Service가 Order를 생성하고 pending 상태로 기록하고 CreateOrderSaga 를 생성한다.
1. CreateOrderSaga는 RecserveCredit 명령을 Customer Service 에게 내린다.
1. Customer Service는 credit을 생성하여 응답한다.
1. CreateOrderSaga 는 응답을 받고 ApproveOrder 나 RejectOrder를 Order Service에게 호출한다.
1. Order Service는 현재 pending 상태를 approved 나 canceled 상태로 변경한다.

CreateOrderSaga 라는 서비스가 전체 트랜잭션을 관리하기 위해서 다른 서비스들을 호출하는 방법으로 구성되어 있다.

다른 예를 살펴보면,

![](https://blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-11-at-7.40.54-PM-768x470.png)

모든 흐름은 Order Saga Orchestrator 에서 다른 서비스를 호출하여 트랜잭션을 관리하고 있다.

롤백을 위한 형태는 다음과 같다.

![](https://blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-11-at-7.41.06-PM-768x489.png)

만약 Stock Service 에서 out of stock 으로 실패의 응답을 주었을 때, OrderSagaOrchestrator는 이전에 수행한 Payment Service 에게 환불하는 명령만 보내고 현재 주문상태를 실패로 설정한다.

각 명령에 해당하는 상태 시스템을 가지고 있고, 이 상태 시스템을 통해서 전체 트랜잭션의 상태를 파악하기 쉽기 때문에 테스트 하기도 쉽다.

- 장점
    - 서비스간의 종속성이 없고 Orchestrator가 호출하기 때문에 분산트랜잭션의 중앙 집중화가 된다.
    - 서비스의 복잡성이 줄어든다.
    - 구현 및 테스트가 쉽다.
    - 롤백을 쉽게 관리할 수 있다.

- 단점
    - 모든 트랜잭션을 Orchestrator가 관리하기 때문에 로직이 복잡해 질 수 있다.
    - Orchestrator라는 추가 서비스가 들어가고 이를 관리해애하여 인프라의 복잡성이 증가된다.

---

Reference : 
- https://blog.couchbase.com/saga-pattern-implement-business-transactions-using-microservices-part/
- https://microservices.io/patterns/data/saga.html


