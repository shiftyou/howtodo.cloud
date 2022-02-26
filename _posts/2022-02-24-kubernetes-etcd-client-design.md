---
layout: post
title: "etcd client 디자인"
date: 2022-02-25
categories: kubernetes
tags: openshift kubernetes etcd
image: Broadcast_Mail.png
---
# etcd v3.1
#### 작동방식
- 여러 etcd endpoint로 구성된 경우 여러 tcp 연결을 유지.
- 그 중 하나를 선택하여 클라이언트 요청을 보냄.
- 오류를 수신하면 나머지 다른 tcp 연결에서 무작위로 연결

![](/images/Pasted%20image%2020220223124835.png)

#### 문제점
- 빠른 장애조치가 가능하지만 많은 리소스가 필요
- 밸런스가 노드의 상태 또는 클러스터 구성원을 이해하지 못해서 멈출 수 있음

# etcd v3.2/v3.3
#### 작동방식
- 여러 클러스터 엔드포인트가 주어지면, 먼저 모든 엔드포인트에 연결을 시도
- 하나가 연결이 되면 밸런서는 하나의 TCP 연결만 유지하고 나머지는 close

![](/images/Pasted%20image%2020220223180554.png)

- 오류가 발생하면 Error Handler 로 전송

![](/images/Pasted%20image%2020220223180752.png)
- Error Handler는 gRPC 서버에서 오류를 가져오 고 오류코드와 메시지를 기반으로 동일한 endpoint에서 다시 시도할지 아니면 다른 주소로 전환할지를 결정

![](/images/Pasted%20image%2020220223181108.png)
![](/images/Pasted%20image%2020220223181111.png)

- watch 및 KeepAlive 와 같은 Stream API는 timeout 이 없어 HTTP/2 ping을 통해서 서버가 응답하지 않으면 다른 endpoint로 전환 
![](/images/Pasted%20image%2020220223181338.png)

#### 문제점
- 비정상적인 endpoint 목록을 관리하나 기본 5초의 타임아웃이 있음.
- 5초 전에 비정상인 endpoint가 정상으로 돌아와도 5초후에 정상으로 확임됨.

![](/images/Pasted%20image%2020220223183320.png)

# etcd v3.4

#### 동작방식
- 비정상 endpoint 목록을 유지하는 대신 클라이언트가 현재 endpoint에서 연결이 끊길 때 마다 다음 endpoint로 round robin
- 이전에는 하나의 tcp 연결만 필요하지만, 노드 수 만큼의 tcp 연결 필요. 하지만, 더 빠른 장애조치 성능과 유연한 로드밸런서 제공
- 기본 라운드로빈, 다른 방식 제공 가능
- 새로운 request 마다 라운드로빈으로 노드에 접근

![](/images/Pasted%20image%2020220223183950.png)

*Reference*
- https://etcd.io/docs/v3.5/learning/design-client/

