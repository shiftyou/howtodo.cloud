---
layout: post
title: "etcd raft 알고리즘"
date: 2022-02-23
categories: kubernetes
tags: openshift kubernetes etcd
image: Broadcast_Mail.png
---
# ETCD
etcd는 key-value 스토어입니다.
kubernetes에서 컨트롤 플레인에 위치하여 모든 데이터가 etcd에 저장되고 있습니다. 데이터는 노드가 몇개인지, 어떤 pod가 어떤 노드에서 동작하는지, 노드의 상태는 어떤지, 어떤 서비스가 디플로이 되어있는지 등의 모든 데이터가 etcd에 저장됩니다.

kubernetes를 업그레이드 하면 이 etcd도 업그레이드 되며 무중단 업그레이드를 위하여 etcd 클러스터에서 노드들을 하나씩 롤링 업그레이드가 됩니다. 업그레이드 시 etcd 클러스터가 무너진다면 kubernetes가 오류가 생겨 모든 리소스의 정보가 사라지게 될 수도 있습니다.

따라서 etcd는 고가용성을 위해 여러대로(홀수) 운영했을 때 합의를 하는 알고리즘으로 RAFT 알고리즘을 사용합니다. 참고로 합의 알고리즘은 다수의 참여자들이 통일된 의사결정을 하기위해 사용하는 알고리즘입니다.

![](/images/Pasted%20image%2020220224162819.png)

합의(Consensus) 알고리즘은 일반적으로 복제된 상태 머신(Replicated State Machine)의 컨텍스트에서 발생합니다. 특정 Server의 State를 다른 Server의 State에게 복제하는 방식의 기법을 **복제된 상태 머신(Replicated State Machine)** 기법이라고 하는데, 아래는 복제된 상태 머신의 아키텍처입니다.

![](/images/Pasted%20image%2020220224161643.png)

합의 알고리즘은 클라이언트의 상태 머신 명령을 포함하는 복제된 로그를 관리합니다. 상태 머신은 로그에서 동일한 명령 시퀀스를 처리하므로 동일한 출력을 생성합니다.

### Leader의 선출

![](/images/Pasted%20image%2020220225101829.png)

RAFT 클러스터는 일반적으로 홀수의 클러스터를 가지고 있으며 각 노드들은 leader, follower, candiate 의 세가지 상태를 가집니다. RAFT 알고리즘 기반의 시스템 내 모든 노드는 기본적으로 follower 상태입니다. 모든 follower들은 leader가 될 수 있으며 candidate 가 되어야 leader가 될 자격이 주어집니다.
1. candidate 노드는 다른 노드들에게 자신을 leader로 뽑아 달라는 Request를 보냅니다.
2. 요청을 받은 노드들은 response를 보냅니다.
3. 클러스터 내의 과반수 이상의 노드에게서 response를 받게 되면 leader 로 선출됩니다.

## Log 복제

![](/images/Pasted%20image%2020220225101847.png)

이렇게 선출된 리더는 데이터 업데이트 요청을 받으면 다음과 같은 순서로 업데이트 합니다.
1. leader는 클라이언트로부터 log를 받아 자신의 Log Entries 데이터를 기록
2. 기록된 데이터는 즉시 commit 하지 않고 다른 follower에게 로그를 복제
3. 과반수 이상의 follower에게서 각자의 Log Entries에 변경사항을 기록할 때 까지 기다림
4. 과반수 이상의 follower에게서 기록이 완료되었다는 response를 받으면 leader 자신을 commit
5. leader가 commit 됨을 follower에게 알리면 follower들도 commit


### Quorum

RAFT 알고리즘을 통해서 leader를 선출하기위해 자신을 포함한 과반수 이상의 노드들의 응답을 받아야 합니다. Quorum은 합의를 유지하기 위한 최소한의 표를 의미하고 값을 구하는 공식은 (n/2)+_1_ 입니다.
다음은 클러스터의 노드에 따른 Quorum을 나타냅니다.

![](/images/Pasted%20image%2020220224173642.png)

위의 표에서 홀수개의 서버수에서 1개의 서버가 추가되면 쿼럼도 1 증가되는 것을 알 수 있습니다. 즉, 클러스터에서 짝수개의 서버가 운영이 된다면 가용성이 떨어진다는 것을 의미합니다. 

candidate 상태의 노드가 leader가 되기 위해서는 해당 클러스터의 노드 수에 따라 최소 쿼럼 수 만큼의 노드들에게서 응답을 받아야 합니다. 즉, leader가 되기 위해서 찬성하는 노드의 수가 반대하는 노드의 수보다 많다는 것을 보장하는 것입니다.

서버의 수가 3개일 때에는 1개의 서버가 문제가 있어도 나머지 1개의 follower와 자신을 포함하면 최소한의 Quorum인 2에 만족하기 때문에 문제없이 etcd는 운영되고 1개의 서버가 더 문제가 생기면 운영이 불가합니다.

### 질문
#### Q. Candidate 노드가 되는 기준은?

![](/images/Pasted%20image%2020220225105615.png)

follower들은 150ms~300ms 사이의 랜덤한 수치의 **Election Timeout**이 설정되어 있습니다. 
1. 자신의 Election Timeout이 끝나면 follower에서 candidate 상태로 전환됩니다. 그리고 다른 follower에게 leader 선출을 위한 request 를 보내게 됩니다.
2. follower들은 candidate 에서 보내온 request에 대해 response를 함과 동시에 자신의 Election Timeout을 초기화 합니다.
3. 과반수 이상의 follower에게서 response를 받은 candidate는 leader가 되게 됩니다.

![](/images/Pasted%20image%2020220225105633.png)

leader는 **Heartbeat Timeout** 설정이 적용됩니다. 이 타임아웃은 Election Timeout보다는 작은 시간 (150ms 이하)입니다. 
1. 타임아웃이 끝나면 Append Entries 메시지를 follower에게 보냅니다.
2. Append Entries 메시지를 받은 follower들은 리더에게 응답을 보내면서 자신의 Election Timeout을 초기화 합니다.
3. 그래서 leader가 살아있는 한 새로운 leader는 선출하지 않고 계속해서 leader를 하게 됩니다.
4. Append Entries 메시지를 받지 못하면 leader 가 없는 것으로 간주되어 Election Timeout이 완료된 노드가 candidate 상태가 되어 새로운 leader를 선출합니다.

#### Q. 여러 Candidate가 발생하면?

![](/images/Pasted%20image%2020220225112038.png)

모든 노드는 Election Timeout과 마찬가지로 **Election Term**을 가지고 있습니다.
1. candidate 노드가 request를 보낼 때 Election Term을 같이 보내고, follower들은 vote를 하기 전에 자신이 해당 term에 vote 했는가를 검사하고 vote 하지 않았다면 vote 응답을 보내고 Election Term을 업데이트 합니다. 즉 1대 선거인지, 2대 선거인지, 3대 선거인지를 받고, 해당 선거에 한번만 투표 가능하도록 체크하는 것입니다.
2. 만약 여러 candidate가 동시에 vote 요청을 하면, 각 follower들은 한번만 투표를 하기 때문에 과반 이상의 vote response를 받은 candidate가 leader가 됩니다.
3. 만약 동일한 개수의 response를 받으면 다시 Election Term 값을 늘이고 request를 보내고 과반 이상의 respose를 받을 때 까지 반복됩니다.



*Reference*
- https://raft.github.io/raft.pdf
- http://thesecretlivesofdata.com/raft/
- https://ssup2.github.io/theory_analysis/Raft_Consensus_Algorithm/
- https://towardsdatascience.com/raft-algorithm-explained-a7c856529f40
- https://www.polarsparc.com/xhtml/Raft.html
- https://foxutech.com/wp-content/uploads/2017/08/two-node-both-started-election.jpg
