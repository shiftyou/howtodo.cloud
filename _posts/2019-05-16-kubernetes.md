---
layout: post
title:  "Kubernetes 사용하기"
categories: kubernetes
tags: kubernetes
---

쿠버네티스 클러스터를 구성하기 위하여 VM에 있는 minikube를 사용하도록 한다.

먼저 클러스터가 구성되어 있는지 살펴본다.

~~~
$ kubectl cluster-info

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
The connection to the server 10.0.2.15:8443 was refused - did you specify the right host or port?
~~~

현재 클러스터 구성이 되어있지 않음을 알 수 있다. 그래서 minikube를 이용하여 쿠버네티스 클러스터를 구성한다.

~~~
$ sudo -E minikube start --vm-driver=none

[sudo] password for user1: 
😄  minikube v1.0.1 on linux (amd64)
💡  Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
🔄  Restarting existing none VM for "minikube" ...
⌛  Waiting for SSH access ...
📶  "minikube" IP address is 10.0.2.15
🐳  Configuring Docker as the container runtime ...
🐳  Version of container runtime is 18.09.6
✨  Preparing Kubernetes environment ...
❌  Unable to load cached images: loading cached images: loading image /home/user1/.minikube/cache/images/k8s.gcr.io/kube-proxy_v1.14.1: stat /home/user1/.minikube/cache/images/k8s.gcr.io/kube-proxy_v1.14.1: no such file or directory
🚜  Pulling images required by Kubernetes v1.14.1 ...
🔄  Relaunching Kubernetes v1.14.1 using kubeadm ... 
⌛  Waiting for pods: apiserver proxy etcd scheduler controller dns
📯  Updating kube-proxy configuration ...
🤔  Verifying component health ......
🤹  Configuring local host environment ...

⚠️  The 'none' driver provides limited isolation and may reduce system security and reliability.
⚠️  For more information, see:
👉  https://github.com/kubernetes/minikube/blob/master/docs/vmdriver-none.md

💗  kubectl is now configured to use "minikube"
🏄  Done! Thank you for using minikube!
~~~

잘 구성되었는지 살펴본다.
~~~
$ kubectl cluster-info

Kubernetes master is running at https://10.0.2.15:8443
KubeDNS is running at https://10.0.2.15:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
~~~

위와 같이 잘 구성되었음을 알 수 있다.

# kubectl get nodes

현재 쿠버네티스 클러스터의 노드를 살펴보는 명령어이다.
~~~
$ kubectl get nodes

NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   2d10h   v1.14.1
~~~

현재 한개의 노드가 존재한다.  
minikube는 단일 노드의 쿠버네티스 환경이다.  만약 클라우드 벤더를 사용하거나 멀티노드로 구성을 한다면 노드의 리스트가 여려개 보여진다.



# kubectl describe nodes

자세한 정보를 보려면 describe 옵션을 준다.
~~~
$ kubectl describe node minikube

Name:               minikube
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=minikube
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/master=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Fri, 10 May 2019 01:44:18 +0900
Taints:             <none>
Unschedulable:      false
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Sun, 12 May 2019 12:11:13 +0900   Fri, 10 May 2019 01:48:28 +0900   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Sun, 12 May 2019 12:11:13 +0900   Fri, 10 May 2019 01:48:28 +0900   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Sun, 12 May 2019 12:11:13 +0900   Fri, 10 May 2019 01:48:28 +0900   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Sun, 12 May 2019 12:11:13 +0900   Fri, 10 May 2019 01:48:38 +0900   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  10.0.2.15
  Hostname:    minikube
Capacity:
 cpu:                2
 ephemeral-storage:  32961296Ki
 hugepages-2Mi:      0
 memory:             8168088Ki
 pods:               110
Allocatable:
 cpu:                2
 ephemeral-storage:  30377130344
 hugepages-2Mi:      0
 memory:             8065688Ki
 pods:               110
System Info:
 Machine ID:                 2f1155c546ff4a35b4cb86e074ebb62e
 System UUID:                94ae9e50-1ea9-4272-bdb2-8c8e0b263fe7
 Boot ID:                    39810bad-7053-49bc-b4a6-346816a7b228
 Kernel Version:             4.18.0-18-generic
 OS Image:                   Ubuntu 18.04.2 LTS
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://18.9.6
 Kubelet Version:            v1.14.1
 Kube-Proxy Version:         v1.14.1
Non-terminated Pods:         (9 in total)
  Namespace                  Name                                CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                  ----                                ------------  ----------  ---------------  -------------  ---
  kube-system                coredns-fb8b8dccf-x85pq             100m (5%)     0 (0%)      70Mi (0%)        170Mi (2%)     2d10h
  kube-system                coredns-fb8b8dccf-zlzls             100m (5%)     0 (0%)      70Mi (0%)        170Mi (2%)     2d10h
  kube-system                etcd-minikube                       0 (0%)        0 (0%)      0 (0%)           0 (0%)         2d10h
  kube-system                kube-addon-manager-minikube         5m (0%)       0 (0%)      50Mi (0%)        0 (0%)         2d10h
  kube-system                kube-apiserver-minikube             250m (12%)    0 (0%)      0 (0%)           0 (0%)         2d10h
  kube-system                kube-controller-manager-minikube    200m (10%)    0 (0%)      0 (0%)           0 (0%)         2d10h
  kube-system                kube-proxy-vtczr                    0 (0%)        0 (0%)      0 (0%)           0 (0%)         46h
  kube-system                kube-scheduler-minikube             100m (5%)     0 (0%)      0 (0%)           0 (0%)         2d10h
  kube-system                storage-provisioner                 0 (0%)        0 (0%)      0 (0%)           0 (0%)         2d10h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                755m (37%)  0 (0%)
  memory             190Mi (2%)  340Mi (4%)
  ephemeral-storage  0 (0%)      0 (0%)
Events:
  Type    Reason                   Age                    From                  Message
  ----    ------                   ----                   ----                  -------
  Normal  Starting                 2d10h                  kubelet, minikube     Starting kubelet.
  Normal  NodeAllocatableEnforced  2d10h                  kubelet, minikube     Updated Node Allocatable limit across pods
  Normal  NodeHasSufficientMemory  2d10h (x8 over 2d10h)  kubelet, minikube     Node minikube status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    2d10h (x8 over 2d10h)  kubelet, minikube     Node minikube status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     2d10h (x7 over 2d10h)  kubelet, minikube     Node minikube status is now: NodeHasSufficientPID
  Normal  Starting                 2d10h                  kube-proxy, minikube  Starting kube-proxy.
  Normal  Starting                 2d10h                  kubelet, minikube     Starting kubelet.
  Normal  NodeHasSufficientMemory  2d10h (x2 over 2d10h)  kubelet, minikube     Node minikube status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    2d10h (x2 over 2d10h)  kubelet, minikube     Node minikube status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     2d10h (x2 over 2d10h)  kubelet, minikube     Node minikube status is now: NodeHasSufficientPID
  Normal  NodeNotReady             2d10h                  kubelet, minikube     Node minikube status is now: NodeNotReady
  Normal  NodeAllocatableEnforced  2d10h                  kubelet, minikube     Updated Node Allocatable limit across pods
  Normal  Starting                 2d10h                  kube-proxy, minikube  Starting kube-proxy.
  Normal  NodeReady                2d10h                  kubelet, minikube     Node minikube status is now: NodeReady
  Normal  Starting                 46h                    kubelet, minikube     Starting kubelet.
  Normal  NodeAllocatableEnforced  46h                    kubelet, minikube     Updated Node Allocatable limit across pods
  Normal  NodeHasSufficientMemory  46h (x8 over 46h)      kubelet, minikube     Node minikube status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    46h (x8 over 46h)      kubelet, minikube     Node minikube status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     46h (x7 over 46h)      kubelet, minikube     Node minikube status is now: NodeHasSufficientPID
  Normal  Starting                 46h                    kube-proxy, minikube  Starting kube-proxy.
  Normal  Starting                 10m                    kubelet, minikube     Starting kubelet.
  Normal  NodeHasSufficientMemory  10m (x8 over 10m)      kubelet, minikube     Node minikube status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    10m (x8 over 10m)      kubelet, minikube     Node minikube status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     10m (x7 over 10m)      kubelet, minikube     Node minikube status is now: NodeHasSufficientPID
  Normal  NodeAllocatableEnforced  10m                    kubelet, minikube     Updated Node Allocatable limit across pods
  Normal  Starting                 9m43s                  kube-proxy, minikube  Starting kube-proxy.
~~~



# hello 이미지 만들어서 레지스트리에 올리기

hello 디렉토리를 만들고 node.js 로 동작하는 샘플을 만들어 보도록 한다.

~~~
$ rm -rf hello
$ mkdir hello
$ cd hello
~~~

server.js
~~~
var http = require('http');
var os = require('os');

var handleRequest = function(request, response) {
  console.log('Received request for URL: ' + request.url);
  response.writeHead(200);
  response.end('Hello World!' + os.hostname());
};
var www = http.createServer(handleRequest);
www.listen(8000);
console.log(os.hostname() + " Server listening..");
~~~

hello.dockerfile (혹은 Dockerfile)
~~~
FROM node:8
EXPOSE 8000
COPY server.js .
CMD node server.js
~~~

hello 이미지를 만든다. (개인의 docker hub 아이디를 쓴다.)
~~~
$ docker build -t jonggyou/hello -f hello.dockerfile .

혹은

$ docker build -t jonggyou/hello .
~~~

레지스트리 등록한다. (push가 되지 않으면 로그인을 먼저 한다)
~~~
$ docker push jonggyou/hello
~~~



# kubectl run {앱이름} -image={이미지명}
docker 와 마찬가지로 가장쉽게 컨테이너를 쉽게 실행하는 방법은 run 옵션이다.

~~~
$ kubectl run hello --image=jonggyou/hello

kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/hello created
~~~

hello 라는 이름의 애플리케이션으로 hello-world 라는 도커 이미지를 실행한 것이라는 의미이다.  
그러나 실제로 hello-world 문구가 출력이 되지 않았다. stdout의 출력은 logs 옵션으로 살펴볼 수 있다.



# kubectl get pods **or** kubectl get po

컨테이너는 pod 이라는 가장 작은 단위로 담겨서 쿠버네티스에서 수행이 된다. 현재 pod 을 살펴보자.
~~~
$ kubectl get po

NAME                     READY   STATUS    RESTARTS   AGE
hello-7f454bcc8f-7l2qk   1/1     Running   0          3m30s
~~~

현재 이름이 hello-7f454bcc8f-7l2qk 라는 이름으로 pod 이 생성되어서 running 중이다.

# kubectl describe pod {POD이름}

특정 pod에 대한 자세한 내용을 보려면 describe 옵션을 사용한다.

~~~
$ kubectl describe pod hello-7f454bcc8f-7l2qk 

Name:               hello-7f454bcc8f-7l2qk
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               minikube/10.0.2.15
Start Time:         Tue, 14 May 2019 21:15:59 +0900
Labels:             pod-template-hash=7f454bcc8f
                    run=hello
Annotations:        <none>
Status:             Running
IP:                 172.17.0.4
Controlled By:      ReplicaSet/hello-7f454bcc8f
Containers:
  hello:
    Container ID:   docker://848e5360c257d608a7f5fbe947727531e2e58291dee1c6651b00a2783153ada9
    Image:          jonggyou/hello
    Image ID:       docker-pullable://jonggyou/hello@sha256:5e7d4c17376e81b005c255e19d67a07c1821728a8a3da38c4782eba91f67298e
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 14 May 2019 21:16:03 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-pf9pl (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-pf9pl:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-pf9pl
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  4m41s  default-scheduler  Successfully assigned default/hello-7f454bcc8f-7l2qk to minikube
  Normal  Pulling    4m40s  kubelet, minikube  Pulling image "jonggyou/hello"
  Normal  Pulled     4m37s  kubelet, minikube  Successfully pulled image "jonggyou/hello"
  Normal  Created    4m37s  kubelet, minikube  Created container hello
  Normal  Started    4m37s  kubelet, minikube  Started container hello

~~~

출력된 정보로 알 수 있는 내용들이다.

- 수행중인 컨테이너의 내부 아이피는 172.17.0.4 이다.
- ReplicaSet으로 애플리케이션이 컨트롤 된다.
- 이미지는 jonggyou/hello 이다.
- 그 외 다수


# kubectl get deployments

pod은 실제로 수행중인 컨테이너이고, 이를 수행하기 위해서 배포된 앱을 살펴보자

~~~
$ kubectl get deployments

NAME    READY   UP-TO-DATE   AVAILABLE   AGE
hello   1/1     1            1           7m5s
~~~

# kubectl describe deployment {앱이름}

~~~
$ kubectl describe deployment hello

Name:                   hello
Namespace:              default
CreationTimestamp:      Tue, 14 May 2019 21:15:59 +0900
Labels:                 run=hello
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=hello
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=hello
  Containers:
   hello:
    Image:        jonggyou/hello
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   hello-7f454bcc8f (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  7m30s  deployment-controller  Scaled up replica set hello-7f454bcc8f to 1
~~~


# kubectl logs {앱이름}
~~~
$ kubectl logs deployment/hello

hello-7f454bcc8f-7l2qk Server listening..
~~~


# kubectl get replicaset **or** kubectl get rs

ReplicaSet으로 등록된 리스트가 나타난다.  기본적으로 run 옵션을 사용하여 수행하면 ReplicaSet 으로 등록된다.
~~~
$ kubectl get rs

NAME               DESIRED   CURRENT   READY   AGE
hello-7f454bcc8f   1         1         1       7m59s
~~~

# kubectl describe rs {ReplicaSet이름}

ReplicaSet에 대한 자세한 정보가 나타난다.
~~~
$ kubectl describe rs hello-7f454bcc8f

Name:           hello-7f454bcc8f
Namespace:      default
Selector:       pod-template-hash=7f454bcc8f,run=hello
Labels:         pod-template-hash=7f454bcc8f
                run=hello
Annotations:    deployment.kubernetes.io/desired-replicas: 1
                deployment.kubernetes.io/max-replicas: 2
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/hello
Replicas:       1 current / 1 desired
Pods Status:    1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  pod-template-hash=7f454bcc8f
           run=hello
  Containers:
   hello:
    Image:        jonggyou/hello
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  8m33s  replicaset-controller  Created pod: hello-7f454bcc8f-7l2qk

~~~

# kubectl delete po {POD이름}

POD을 삭제하기 위해서는 delete po 를 사용한다. 하지만,  PO는 컨테이너가 수행되는 주체이고 이는 상세정보에서도 알 수 있듯이 ReplicaSet 의해서 관리된다.

~~~
$ kubectl get po
NAME                     READY   STATUS    RESTARTS   AGE
hello-7f454bcc8f-7l2qk   1/1     Running   0          8m59s

$ kubectl delete po hello-7f454bcc8f-7l2qk
pod "hello-7f454bcc8f-7l2qk" deleted

$ kubectl get po
NAME                     READY   STATUS    RESTARTS   AGE
hello-7f454bcc8f-6bgxh   1/1     Running   0          67s
~~~

위와 같이 pod를 삭제해도 또 생긴다. 이것은 ReplicaSet 에 의해서 1개의 pod을 계속적으로 유지하기 때문에 삭제된 pod를 대신해서 새로운 pod이 생성된다.

# kubectl delete deployment {앱이름}

실제 애플리케이션을 삭제하기 위해서는 deployment 에서 삭제를 한다.
~~~
$ kubectl delete deployment hello
deployment.extensions "hello" deleted
~~~

삭제되었는지 확인한다.
~~~
$ kubectl get deployments
No resources found.
~~~

다시 pod 을 확인한다.
~~~
$ kubectl get po
NAME                     READY   STATUS        RESTARTS   AGE
hello-7f454bcc8f-6bgxh   1/1     Terminating   0          2m8
~~~

아직 존재한다. 하지만 몇초후에 확인을 해 보면 삭제되었음을 알 수 있다.
~~~
$ kubectl get pod
No resources found.
~~~


# 3개의 pod 생성

애플리케이션 전체를 삭제하고자 할 때 사용한다.  
먼저 3개의 애플리케이션을 실행한다.
~~~
$ kubectl run hello1 --image=jonggyou/hello
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/hello1 created

$ kubectl run hello2 --image=jonggyou/hello
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/hello2 created

$ kubectl run hello3 --image=jonggyou/hello
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/hello3 created
~~~


# kubectl get all

현재 수행중인 전체 리소스를 확인한다.
~~~
$ kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
hello1-988dff9cc-t42j4    1/1     Running   0          2m21s
hello2-8695787df5-q6srb   1/1     Running   0          2m17s
hello3-7594dbbb9c-4lgsh   1/1     Running   0          2m12s
~~~

현재 배포된 상태를 확인한다.
~~~
$ kubectl get deployments
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
hello1   1/1     1            1           2m18s
hello2   1/1     1            1           2m14s
hello3   1/1     1            1           2m9s
~~~

한번에 전체 상태를 확인한다.
~~~
NAME                          READY   STATUS    RESTARTS   AGE
pod/hello1-988dff9cc-t42j4    1/1     Running   0          118s
pod/hello2-8695787df5-q6srb   1/1     Running   0          114s
pod/hello3-7594dbbb9c-4lgsh   1/1     Running   0          109s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d19h

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello1   1/1     1            1           118s
deployment.apps/hello2   1/1     1            1           114s
deployment.apps/hello3   1/1     1            1           109s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/hello1-988dff9cc    1         1         1       118s
replicaset.apps/hello2-8695787df5   1         1         1       114s
replicaset.apps/hello3-7594dbbb9c   1         1         1       109s
~~~

# kubectl delete deployment -all

전체 애플리케이션을 삭제한다.
~~~
$ kubectl delete deployment --all
deployment.extensions "hello1" deleted
deployment.extensions "hello2" deleted
deployment.extensions "hello3" deleted
~~~

삭제가 완료되었는지 확인한다.
~~~
$ kubectl get all

NAME                          READY   STATUS        RESTARTS   AGE
pod/hello1-988dff9cc-t42j4    1/1     Terminating   0          3m26s
pod/hello2-8695787df5-q6srb   1/1     Terminating   0          3m22s
pod/hello3-7594dbbb9c-4lgsh   1/1     Terminating   0          3m17s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d19h
~~~

Terminating 중이다.. 다시한번 확인한다.
~~~
$ kubectl get all

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d19h
~~~
pod이 모두 사라졌음을 알 수 있다.


# Deployment 생성

기존에 생성한 jonggyou/hello 이미지를 이용하여 실습을 해 본다.

- CLI를 이용하여 deployment를 생성한다.

  ~~~
  $ kubectl create deployment myhello --image=jonggyou/hello
  deployment.apps/myhello created
  ~~~

  현재 상태를 확인한다.
  ~~~
  $ kubectl get all

  NAME                           READY   STATUS              RESTARTS   AGE
  pod/myhello-756f7f969c-lksfl   0/1     ContainerCreating   0          3s

  NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
  service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d19h

  NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/myhello   0/1     1            0           3s

  NAME                                 DESIRED   CURRENT   READY   AGE
  replicaset.apps/myhello-756f7f969c   1         1         0       3s

  ~~~


- 다음과 같이 yaml 파일로 생성할 수도 있다.   먼저 hello-deploy.yaml을 만든다.  

  hello-deploy.yaml
  ~~~
  apiVersion: apps/v1beta1
  kind: Deployment
  metadata:
    name: hello-deployment
  spec:
    replicas: 1
    template:
      metadata:
        labels:
          run: hello
      spec:
        containers:
        - name: hello
          image: jonggyou/hello:latest
          ports:
          - containerPort: 8000
  ~~~

  그리고 hello-deploy.yaml을 이용하여 Deployment 를 생성한다.
  ~~~
  $ kubectl create -f hello-deploy.yaml

  deployment.apps/hello-deployment created
  ~~~

  현재 상태를 확인한다.

  ~~~
  NAME                                    READY   STATUS    RESTARTS   AGE
  pod/hello-deployment-6d8454d847-qqwsh   1/1     Running   0          35s
  pod/myhello-756f7f969c-lksfl            1/1     Running   0          3m48s

  NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
  service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d20h

  NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/hello-deployment   1/1     1            1           35s
  deployment.apps/myhello            1/1     1            1           3m48s

  NAME                                          DESIRED   CURRENT   READY   AGE
  replicaset.apps/hello-deployment-6d8454d847   1         1         1       35s
  replicaset.apps/myhello-756f7f969c            1         1         1       3m48s
  ~~~

# Service 생성

- CLI를 이용해 다음과 같이 Service를 생성할 수 있다.
  ~~~
  $ kubectl expose deployment myhello --port=8000

  service/myhello exposed
  ~~~

  상태를 확인해 본다.
  ~~~
  $ kubectl get all

  NAME                                    READY   STATUS    RESTARTS   AGE
  pod/hello-deployment-6d8454d847-qqwsh   1/1     Running   0          3m29s
  pod/myhello-756f7f969c-lksfl            1/1     Running   0          6m42s

  NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
  service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP    4d20h
  service/myhello      ClusterIP   10.96.64.149   <none>        8000/TCP   60s

  NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/hello-deployment   1/1     1            1           3m29s
  deployment.apps/myhello            1/1     1            1           6m42s

  NAME                                          DESIRED   CURRENT   READY   AGE
  replicaset.apps/hello-deployment-6d8454d847   1         1         1       3m29s
  replicaset.apps/myhello-756f7f969c            1         1         1       6m42s
  ~~~

- 그리고 역시 yaml 파일을 이용하여 생성할 수도 있다.

  hello-service.yaml
  ~~~
  apiVersion: v1
  kind: Service
  metadata:
    name: hello-service
    labels:
      run: hello
  spec:
    type: NodePort
    ports:
    - port: 8000
      protocol: TCP
    selector:
      run: hello
  ~~~

  hello-service.yaml 파일을 이용하여 Service를 생성한다.
  ~~~
  $ kubectl create -f hello-service.yaml

  service/hello-service created
  ~~~

  상태를 확인 해 본다.
  ~~~
  $ kubectl get all

  NAME                                    READY   STATUS    RESTARTS   AGE
  pod/hello-deployment-6d8454d847-qqwsh   1/1     Running   0          8m42s
  pod/myhello-756f7f969c-lksfl            1/1     Running   0          11m

  NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
  service/hello-service   NodePort    10.111.112.145   <none>        8000:30365/TCP   21s
  service/kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP          4d20h
  service/myhello         ClusterIP   10.96.64.149     <none>        8000/TCP         6m13s

  NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/hello-deployment   1/1     1            1           8m42s
  deployment.apps/myhello            1/1     1            1           11m

  NAME                                          DESIRED   CURRENT   READY   AGE
  replicaset.apps/hello-deployment-6d8454d847   1         1         1       8m42s
  replicaset.apps/myhello-756f7f969c            1         1         1       11m

  ~~~

# Deployment, Service 동시 생성

yaml 을 이용하면 한번에 두 객체를 생성할 수 있다.  
단순히 두 yaml 을 합치기만 하면 된다.

hello.yaml
~~~
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: hello-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        run: hello
    spec:
      containers:
      - name: hello
        image: jonggyou/hello
        ports:
        - containerPort: 8000

---
apiVersion: v1
kind: Service
metadata:
  name: hello-service
  labels:
    run: hello
spec:
  type: NodePort
  ports:
  - port: 8000
    protocol: TCP
  selector:
    run: hello
~~~

마찬가지로 create 옵션을 사용한다.
~~~
$ kubectl create -f hello.yaml 

Error from server (AlreadyExists): error when creating "hello.yaml": deployments.apps "hello-deployment" already exists
Error from server (AlreadyExists): error when creating "hello.yaml": services "hello-service" already exists
~~~
현재 같은 이름이 구성되어 있기에 오류를 낸다.

# 서비스 접근하기

현재 Service를 삺펴보면
~~~
$ kubectl get svc

NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
hello-service   NodePort    10.111.112.145   <none>        8000:30365/TCP   5m16s
kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP          4d20h
myhello         ClusterIP   10.96.64.149     <none>        8000/TCP         11m

~~~

hello-service 가 32316 포트로 접근이 가능하다.

VM의 minikube는 다른 VM을 사용하고 있지 않으므로 로컬에서 접속이 가능하다.

~~~
$ curl localhost:32316
Hello World!hello-deployment-6d8454d847-c9gql
~~~


# 전체 객체 보기

일반적으로 현재 상태를 보기 위해서 다음의 명령들을 내린다.
~~~
$ kubectl get service
$ kubectl get po
$ kubectl get deployment
$ kubectl get rc
~~~

이를 한꺼번에 보기 위해서 다음과 같이 명령한다.
~~~
$ kubectl get all
~~~


# Scale out

현재의 Deployment 상태를 보면 다음과 같다.
~~~
$ kubectl get deployment

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
hello-deployment   1/1     1            1           56m
~~~

이것은 서버가 한개라는 의미이고 여러개의 서버를 둘 수 있다.  
3개의 서버로 늘려본다.
~~~
$ kubectl scale deployment hello-deployment --replicas=3

deployment.extensions/hello-deployment scaled
~~~

다시 Deployment를 확인하면 다음과 같이 3개가 되었음을 알 수 있다.
~~~
$ kubectl get deployment

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
hello-deployment   3/3     3            3           5m7s
~~~

3개의 deployment가 있다는 것은 pod이 3개임을 의미한다.
~~~
$ kubectl get pod

NAME                                READY   STATUS    RESTARTS   AGE
hello-deployment-769d4b96f5-552ff   1/1     Running   0          2m34s
hello-deployment-769d4b96f5-mcgkd   1/1     Running   0          62m
hello-deployment-769d4b96f5-t6fq4   1/1     Running   0          2m34s
~~~
보는 바와 같이 pod 이 3개가 있다.

다시 전체 리소스를 보면 다음과 같다.
~~~
$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/hello-deployment-769d4b96f5-552ff   1/1     Running   0          15s
pod/hello-deployment-769d4b96f5-mcgkd   1/1     Running   0          60m
pod/hello-deployment-769d4b96f5-t6fq4   1/1     Running   0          15s

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/hello-service   NodePort    10.110.201.142   <none>        8000:32316/TCP   60m
service/kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP          60m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-deployment   3/3     3            3           60m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-deployment-769d4b96f5   3         3         3       60m
~~~

서비스는 여전히 한개이지만, 3개의 POD으로 포워딩 할 것이다. 확인을 해 보도록 한다.
~~~
$ curl http://localhost:32316
Hello World!hello-deployment-769d4b96f5-552ff

$ curl http://localhost:32316
Hello World!hello-deployment-769d4b96f5-t6fq4

$ curl http://localhost:32316
Hello World!hello-deployment-769d4b96f5-t6fq4

$ curl http://localhost:32316
Hello World!hello-deployment-769d4b96f5-t6fq4

$ curl http://localhost:32316
Hello World!hello-deployment-769d4b96f5-t6fq4

$ curl http://localhost:32316
Hello World!hello-deployment-769d4b96f5-mcgkd

$ curl http://localhost:32316
Hello World!hello-deployment-769d4b96f5-t6fq4

$ curl http://localhost:32316
Hello World!hello-deployment-769d4b96f5-mcgkd

$ curl http://localhost:32316
Hello World!hello-deployment-769d4b96f5-mcgkd
~~~
위의 결과와 같이 다른 hostname 이 출력됨을 알 수 있다.

# rolling update

~~~
$ kubectl run myhello --image=jonggyou/hello 
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/myhello created
~~~

~~~
$ kubectl scale deployment myhello --replicas=4
deployment.extensions/myhello scaled
~~~

~~~
$ kubectl get all
NAME                          READY   STATUS              RESTARTS   AGE
pod/myhello-55d859d77-8lf82   0/1     ContainerCreating   0          9s
pod/myhello-55d859d77-cbw45   1/1     Running             0          9s
pod/myhello-55d859d77-lkshz   1/1     Running             0          66s
pod/myhello-55d859d77-mp58b   1/1     Running             0          9s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2m9s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myhello   3/4     4            3           66s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/myhello-55d859d77   4         4         3       66s

~~~

~~~
$ kubectl set image deployment/myhello myhello=jonggyou/hello:v2
deployment.extensions/myhello image updated
~~~

~~~
$ kubectl get po --watch
NAME                       READY   STATUS              RESTARTS   AGE
myhello-55d859d77-8lf82    1/1     Terminating         0          2m50s
myhello-55d859d77-cbw45    1/1     Terminating         0          2m50s
myhello-55d859d77-lkshz    1/1     Terminating         0          3m47s
myhello-55d859d77-mp58b    1/1     Terminating         0          2m50s
myhello-6c6f6c6945-65f7r   0/1     ContainerCreating   0          4s
myhello-6c6f6c6945-7pnqq   1/1     Running             0          6s
myhello-6c6f6c6945-djf4n   1/1     Running             0          11s
myhello-6c6f6c6945-rsz4w   1/1     Running             0          12s
myhello-6c6f6c6945-65f7r   1/1     Running             0          6s
myhello-55d859d77-8lf82    0/1     Terminating         0          3m9s
myhello-55d859d77-8lf82    0/1     Terminating         0          3m13s
myhello-55d859d77-8lf82    0/1     Terminating         0          3m13s
myhello-55d859d77-cbw45    0/1     Terminating         0          3m16s
myhello-55d859d77-cbw45    0/1     Terminating         0          3m17s
myhello-55d859d77-cbw45    0/1     Terminating         0          3m17s
myhello-55d859d77-mp58b    0/1     Terminating         0          3m18s
myhello-55d859d77-mp58b    0/1     Terminating         0          3m19s
myhello-55d859d77-mp58b    0/1     Terminating         0          3m19s
myhello-55d859d77-lkshz    0/1     Terminating         0          4m17s
myhello-55d859d77-lkshz    0/1     Terminating         0          4m18s
myhello-55d859d77-lkshz    0/1     Terminating         0          4m18s
~~~

# kubectl edit deployment 

~~~
$ kubectl edit deployment myhello
deployment.extensions/myhello edited
~~~

~~~
$ kubectl get po --watch
NAME                       READY   STATUS              RESTARTS   AGE
myhello-55d859d77-rzgbg    0/1     ContainerCreating   0          3s
myhello-6c6f6c6945-djf4n   1/1     Running             0          9m32s
myhello-6c6f6c6945-rsz4w   1/1     Running             0          9m33s
myhello-55d859d77-rzgbg    1/1     Running             0          5s
myhello-6c6f6c6945-djf4n   1/1     Terminating         0          9m34s
myhello-55d859d77-8qsng    0/1     Pending             0          0s
myhello-55d859d77-8qsng    0/1     Pending             0          0s
myhello-55d859d77-8qsng    0/1     ContainerCreating   0          0s
myhello-55d859d77-8qsng    1/1     Running             0          5s
myhello-6c6f6c6945-rsz4w   1/1     Terminating         0          9m40s
myhello-6c6f6c6945-djf4n   0/1     Terminating         0          10m
myhello-6c6f6c6945-djf4n   0/1     Terminating         0          10m
myhello-6c6f6c6945-djf4n   0/1     Terminating         0          10m
myhello-6c6f6c6945-rsz4w   0/1     Terminating         0          10m
myhello-6c6f6c6945-rsz4w   0/1     Terminating         0          10m
myhello-6c6f6c6945-rsz4w   0/1     Terminating         0          10m

~~~

# kubectl delete all --all

모든 deployment 를 삭제하려면 다음과 같이 명령한다.
~~~
$ kubectl delete deployment --all
~~~

모든 service를 삭제하려면 다음과 같이 명령한다.
~~~
$ kubectl delete service --all
~~~

모든것을 한꺼번에 삭제하려면 다음과 같이 명령한다.
~~~
$ kubectl delete all --all
~~~


# Label

~~~
$ kubectl get po --show-labels

NAME                                READY   STATUS    RESTARTS   AGE   LABELS
hello-deployment-769d4b96f5-552ff   1/1     Running   0          14m   pod-template-hash=769d4b96f5,run=hello
hello-deployment-769d4b96f5-mcgkd   1/1     Running   0          74m   pod-template-hash=769d4b96f5,run=hello
hello-deployment-769d4b96f5-t6fq4   1/1     Running   0          14m   pod-template-hash=769d4b96f5,run=hello
~~~

~~~
$ kubectl get po -l run

NAME                                READY   STATUS    RESTARTS   AGE
hello-deployment-769d4b96f5-552ff   1/1     Running   0          14m
hello-deployment-769d4b96f5-mcgkd   1/1     Running   0          74m
hello-deployment-769d4b96f5-t6fq4   1/1     Running   0          14m
~~~

~~~
$ kubectl label po hello-deployment-769d4b96f5-552ff name=app1

pod/hello-deployment-769d4b96f5-552ff labeled
~~~

~~~
$ kubectl label po hello-deployment-769d4b96f5-mcgkd name=app2

pod/hello-deployment-769d4b96f5-mcgkd labeled
~~~

~~~
$ kubectl get po -l name

NAME                                READY   STATUS    RESTARTS   AGE
hello-deployment-769d4b96f5-552ff   1/1     Running   0          16m
hello-deployment-769d4b96f5-mcgkd   1/1     Running   0          75m
~~~

~~~
$ kubectl get po -l '!name'

NAME                                READY   STATUS    RESTARTS   AGE
hello-deployment-769d4b96f5-t6fq4   1/1     Running   0          16m
~~~

~~~
$ kubectl get po -l name=app1

NAME                                READY   STATUS    RESTARTS   AGE
hello-deployment-769d4b96f5-552ff   1/1     Running   0          16m
~~~


~~~
$ kubectl delete po -l name
$ kubectl delete po -l name=app1
~~~

# Namespace


~~~
$ kubectl get ns
NAME              STATUS   AGE
default           Active   4d20h
kube-node-lease   Active   4d20h
kube-public       Active   4d20h
kube-system       Active   4d20h
~~~

~~~
$ kubectl create namespace test
namespace/test created
~~~

~~~
$ kubectl get ns
NAME              STATUS   AGE
default           Active   4d20h
kube-node-lease   Active   4d20h
kube-public       Active   4d20h
kube-system       Active   4d20h
test              Active   36s
~~~

~~~
$ kubectl create -f hello.yaml -n test
deployment.apps/hello-deployment created
service/hello-service created
~~~

~~~
$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6m54s
~~~

~~~
$ kubectl get all -n test
NAME                                    READY   STATUS    RESTARTS   AGE
pod/hello-deployment-6d8454d847-dv8xp   1/1     Running   0          10s

NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/hello-service   NodePort   10.96.88.216   <none>        8000:31913/TCP   10s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-deployment   1/1     1            1           10s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-deployment-6d8454d847   1         1         1       10s
~~~

~~~
$ kubectl delete service hello-service 
Error from server (NotFound): services "hello-service" not found
~~~

~~~
$ kubectl delete service hello-service -n test
service "hello-service" deleted
~~~

~~~
$ kubectl get all -n test
NAME                                    READY   STATUS    RESTARTS   AGE
pod/hello-deployment-6d8454d847-dv8xp   1/1     Running   0          2m20s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-deployment   1/1     1            1           2m20s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-deployment-6d8454d847   1         1         1       2m20s
~~~

~~~
$ kubectl delete ns test
namespace "test" deleted
~~~

~~~
$ kubectl get all -n test
No resources found.
~~~

~~~
$ kubectl get ns
NAME              STATUS   AGE
default           Active   4d20h
kube-node-lease   Active   4d20h
kube-public       Active   4d20h
kube-system       Active   4d20h
~~~

hello-namespace.yaml
~~~
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace

---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: hello-deployment
  namespace: my-namespace
spec:
  replicas: 1
  template:
    metadata:
      labels:
        run: hello
    spec:
      containers:
      - name: hello
        image: jonggyou/hello
        ports:
        - containerPort: 8000

---
apiVersion: v1
kind: Service
metadata:
  name: hello-service
  namespace: my-namespace
  labels:
    run: hello
spec:
  type: NodePort
  ports:
  - port: 8000
    protocol: TCP
  selector:
    run: hello
~~~

~~~
$ kubectl create -f hello-namespace.yaml 
namespace/my-namespace created
deployment.apps/hello-deployment created
service/hello-service created
~~~

~~~
$ kubectl get ns
NAME              STATUS   AGE
default           Active   4d20h
kube-node-lease   Active   4d20h
kube-public       Active   4d20h
kube-system       Active   4d20h
my-namespace      Active   6s
~~~

~~~
$ kubectl get all 
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6m20s
~~~

~~~
$ kubectl get all -n my-namespace
NAME                                    READY   STATUS    RESTARTS   AGE
pod/hello-deployment-84975b6bf8-ngvnh   1/1     Running   0          23s

NAME                    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/hello-service   NodePort   10.103.212.240   <none>        8000:32226/TCP   23s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-deployment   1/1     1            1           24s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-deployment-84975b6bf8   1         1         1       23s
~~~




~~~
$ kubectl get ns
NAME              STATUS   AGE
default           Active   2d20h
kube-node-lease   Active   2d20h
kube-public       Active   2d20h
kube-system       Active   2d20h

$ kubectl get po --namespace default
NAME                                READY   STATUS    RESTARTS   AGE
hello-deployment-769d4b96f5-552ff   1/1     Running   0          19m
hello-deployment-769d4b96f5-mcgkd   1/1     Running   0          79m
hello-deployment-769d4b96f5-t6fq4   1/1     Running   0          19m

$ kubectl get po -n default
NAME                                READY   STATUS    RESTARTS   AGE
hello-deployment-769d4b96f5-552ff   1/1     Running   0          19m
hello-deployment-769d4b96f5-mcgkd   1/1     Running   0          79m
hello-deployment-769d4b96f5-t6fq4   1/1     Running   0          19m

$ kubectl get po -n kube-system
NAME                               READY   STATUS             RESTARTS   AGE
coredns-fb8b8dccf-x85pq            0/1     CrashLoopBackOff   118        2d20h
coredns-fb8b8dccf-zlzls            0/1     CrashLoopBackOff   117        2d20h
etcd-minikube                      1/1     Running            4          2d20h
kube-addon-manager-minikube        1/1     Running            4          2d20h
kube-apiserver-minikube            1/1     Running            4          2d20h
kube-controller-manager-minikube   1/1     Running            14         2d20h
kube-proxy-vtczr                   1/1     Running            3          2d8h
kube-scheduler-minikube            1/1     Running            4          2d20h
storage-provisioner                1/1     Running            6          2d20h
~~~

~~~
$ kubectl get all -n kube-system
NAME                                   READY   STATUS             RESTARTS   AGE
pod/coredns-fb8b8dccf-x85pq            0/1     CrashLoopBackOff   112        4d20h
pod/coredns-fb8b8dccf-zlzls            0/1     CrashLoopBackOff   111        4d20h
pod/etcd-minikube                      1/1     Running            5          4d20h
pod/kube-addon-manager-minikube        1/1     Running            5          4d20h
pod/kube-apiserver-minikube            1/1     Running            5          4d20h
pod/kube-controller-manager-minikube   1/1     Running            15         4d20h
pod/kube-proxy-vtczr                   1/1     Running            4          4d9h
pod/kube-scheduler-minikube            1/1     Running            5          4d20h
pod/storage-provisioner                1/1     Running            8          4d20h

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   4d20h

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/kube-proxy   1         1         1       1            1           <none>          4d20h

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   0/2     2            0           4d20h

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-fb8b8dccf   2         2         0       4d20h
~~~

~~~
$ kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         minikube   minikube   minikube   
~~~

~~~
$ kubectl config set-context --current --namespace test
Context "minikube" modified.
~~~

~~~
$ kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         minikube   minikube   minikube   test
~~~

