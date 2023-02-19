---
layout: post
title: "etcd leaner 디자인"
date: 2022-11-24
categories: kubernetes
tags: openshift kubernetes etcd
image: Broadcast_Mail.png
---
# ETCD 문제점
## 1. 새 클러스터 구성원이 리더에 과부하를 줌
- 새 etcd 맴버는 데이터 없이 시작하므로 leader의 로그를 따라잡을 때 까지 leader로 부터 많은 업데이트를 요구
- 따라서 리더의 네트워크가 과부하로 인하여 다른 맴버의 heart beat가 실패할 확률이 높아짐.
- 이와 같은 경우, follower 는 새로운 leader를 선출하기 위한 시간이 모자랄 수 있음.

![](/images/Pasted%20image%2020220223184909.png)

## 2. 네트워크 파티션

- 네트워크 파티션이 발생하면 leader가 quorum을 유지하면 해당 클러스터는 계속 유지.
- 아래 그림에서 최소 2개의 quorum을 유지하기 때문에 클러스터는 계속 동작

![](/images/Pasted%20image%2020220223185038.png)

- 리더가 클러스터의 나머지 부분에서 격리되면 기존 리더는 quorum을 만족하지 못하기 때문에 클러스터 가용성에 영향을 미치는 follower로 상태 변경

![](/images/Pasted%20image%2020220225112259.png)

3노드 클러스터에서 노드가 하나 추가되고 난 뒤에 네트워크 파티션이 일어날 경우, 새 노드가 리더와 동일한 파티션에 있는 경우, 리더는 여전히 활성 쿼럼3을 유지하기 때문에 리더 선택이 발생하지 않고 클러스터 가용성에 영향이 없습니다.

![](/images/Pasted%20image%2020220225112820.png)

클러스터가 2-2로 분할 될 경우 두 분할 모두 3의 정족수를 유지하지 않아서 새로운 leader의 선출과정이 일어나게 됩니다.

![](/images/Pasted%20image%2020220225112950.png)

3개의 노드에서 네트워크 파티션이 먼저 발생한 다음에 새로운 구성이 추가된다면, 클러스터의 노드가 4개가 되어 쿼럼이 3이 필요하지만, 활성화된 노드느 2개 뿐이니 새로운 leader가 선출 과정이 일어납니다.

![](/images/Pasted%20image%2020220225113215.png)
맴버 추가 과정은 해당 클러스터의 쿼럼이 변경될 수 있기 때문에 기존의 비정상 노드를 우선 제거하고 노드를 추가하는 것이 좋습니다.

# Learner 

etcd v3.4에서는 리더의 로그를 따라잡을 때까지 클러스터 **에 투표가 없는 구성원** 으로 참여하는 새로운 노드 상태 "learner"를 도입했습니다.

리더는 새 learner 노드를 추가하기 위해 가능한 최소한의 작업을 수행해야 합니다. 
`member add --learner`
클러스터에 투표하지 않는 구성원으로 가입하지만 여전히 리더로부터 모든 데이터를 받는 새 learner 를 추가하는 명령입니다.

![](/images/Pasted%20image%2020220225114619.png)

learner가  리더의 진행 상황을 따라잡으면 
`member promote`
API를 사용하여 learner를 투표 구성원으로 승격할 수 있으며, 이는 정족수에 포함됩니다.

![](/images/Pasted%20image%2020220225114722.png)

etcd 서버는 운영 안전성을 보장하기 위해 승격 요청을 검증합니다. 해당 로그가 리더의 로그를 따라잡은 후에야 learner는 투표 구성원으로 승격될 수 있습니다.

![](/images/Pasted%20image%2020220225114801.png)

learner는 승격될 때까지 대기 노드 역할만 합니다.  승격이 되기 전까지는 learner는 리더가 될 자격이 없습니다. 또한 learner는 클라이언트 읽기 및 쓰기를 거부합니다.

![](/images/Pasted%20image%2020220225114943.png)

etcd는 클러스터가 가질 수 있는 총 learner 수를 제한하고 로그 복제로 리더에 과부하가 걸리는 것을 방지합니다

OCP 4.9 부터 etcd v3.5가 사용되고 있습니다.

*Referece*
- https://etcd.io/docs/v3.5/learning/design-learner/
