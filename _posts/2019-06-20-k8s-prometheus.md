---
layout: post
title:  Prometheus
categories: kubernetes
tags: kubernetes promethus monitoring
---
# Prometheus 란

오픈소스 시스템 모니터링이다.   
CNCF에 Kubernetes 이후 두번째로 등록된 프로젝트로써 Kubernetes의 모니터링 용도로 많이 이용되며 다음의 특징을 가진다.

- Key/Value 쌍으로 식별되는 시계열 데이터가 있는 다차원 데이터 모델
- PromQL,이 차원을 활용하는 유연한 쿼리 언어 사용
- HTTP 푸시 (push)를 통해 발생
- 그래프 및 대시 보드 지원

그래서 시간별로 구분되는 로그를 처리하는 모니터링이라던지, 마이크로서비스같이 다이나믹한 서비스 지향 아키텍처의 다차원적인 데이터의 수집 모니터링으로 잘 어울린다. 그리고 장애 발생시에 즉각적으로 문제를 진단할 수 있는 안정성을 위해 설계되었기 때문에 네티워크 저장소나 다른 원격 서비스에 의존하지 않고 독립적으로 동작하여 다른 인프라가 고장났을 때라도 이를 모니터링 할 수 있다.

# 컴포넌트

- Prometheus Server : 시계열 데이터를 긁어서 저장하는 주요 모듈
- client libraries : 어플리케이션 코드를 계측하기 위한 모듈
- push gateway : 짧은 작업의 지원을 위한 모듈
- exporters : HAProxy, StatsD, Graphite 등의 서비스를 위한 특수 목적을 가진 모듈
- alertmanager : 경고를 처리하는 모듈
- 그 외 여러가지 툴들

대부분의 Promethus 컴포넌트는 GO 언어로 만들어져 있어 생성하고 배포하기 쉽다.

# 아키텍처

![](https://prometheus.io/assets/architecture.png)

좌측의 Jobs 나 exporters 를 는 target 으로 실제 모니터링 타겟이 되는 서버이다.
exporter를 수행하여 해당 모니터링 로그를 수집하고, Prometheus Server에서  pull 방식으로 해당 로그들을 http 를 통해서 가져오게 된다.

상위의 Service discovery는 서버의 ip 가 유동적으로 변동되는 환경에서 target를 찾기 위해서 존재하는 것이다. 


# 설치

다운로드할 파일들은 https://prometheus.io/download/ 에서 찾을 수 있다.  
위에서 소개한 각 모듈들을 모두 다운로드 할 수 있다. 각 모듈을 하나씩 설치하는 것 보다는 한꺼번에 패키지로 설치하는 것이 더 나은 방법이다.

Helm을 이용하여 설치를 하면 쉽다.  
OKE에서 Cluster를 구성하면 자동으로 Helm의 서버쪽 모듈인 tiller가 설치가 된다. 
클라이언트에 대한 설치는 [여기](https://www.howtodo.cloud/kubernetes/2019/04/15/k8s-helm.html)를 참조한다.


1. 설치되어있는 helm 을 확인한다.
    ~~~
    helm version
    ~~~
    다음과 같이 결과가 나온다.
    ~~~
    Client: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
    Server: &version.Version{SemVer:"v2.8.2", GitCommit:"a80231648a1473929271764b920a8e346f6de844", GitTreeState:"clean"}
    ~~~
    위의 예시에는 Client 의 버젼은 2.14.1이고 Server의 버젼은 2.8.2 이다.

    만약 helm 이 로컬컴퓨터에 없다면 설치를 한다.
    MacOS에서는 다음과 같이 설치한다.
    ~~~sh
    brew install kubernetes-helm
    ~~~

1. helm을 이용하여 prometheus를 Kubernetes에 설치한다.
    ~~~sh
    helm install stable/prometheus 
    ~~~

    만약 다음과 같이 오류가 난다면 버젼을 맞추어야 한다.
    ~~~
    Error: incompatible versions client[v2.14.1] server[v2.8.2]
    ~~~
    
    다음과 같이 수행하여 서버를 업그레이드 한다.

    ~~~
    $ helm init --upgrade

    $HELM_HOME has been configured at /Users/jonggyou/.helm.

    Tiller (the Helm server-side component) has been upgraded to the current version.
    ~~~
    버젼을 체크해 보면 동일함을 알 수 있다. 
    ~~~
    $ helm version

    Client: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
    Server: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
    ~~~
    이제 다시 prometheus를 설치한다.

    ~~~
    $ helm install stable/prometheus

    NAME:   foolhardy-cardinal
    LAST DEPLOYED: Fri Jun 21 15:10:03 2019
    NAMESPACE: default
    STATUS: DEPLOYED

    RESOURCES:
    ==> v1/ConfigMap
    NAME                                        DATA  AGE
    foolhardy-cardinal-prometheus-alertmanager  1     6s
    foolhardy-cardinal-prometheus-server        3     6s

    ==> v1/PersistentVolumeClaim
    NAME                                        STATUS   VOLUME  CAPACITY  ACCESS MODES  STORAGECLASS  AGE
    foolhardy-cardinal-prometheus-alertmanager  Pending  oci     6s
    foolhardy-cardinal-prometheus-server        Pending  oci     6s

    ==> v1/Pod(related)
    NAME                                                             READY  STATUS             RESTARTS  AGE
    foolhardy-cardinal-prometheus-alertmanager-8f6567c4f-m8497       0/2    Pending            0         5s
    foolhardy-cardinal-prometheus-kube-state-metrics-5bfbd94cclz8nd  1/1    Running            0         5s
    foolhardy-cardinal-prometheus-node-exporter-72ssg                1/1    Running            0         5s
    foolhardy-cardinal-prometheus-node-exporter-jz7m7                0/1    ContainerCreating  0         5s
    foolhardy-cardinal-prometheus-node-exporter-kgpth                1/1    Running            0         5s
    foolhardy-cardinal-prometheus-pushgateway-8447c8ffcd-b4wcd       0/1    Running            0         5s
    foolhardy-cardinal-prometheus-server-f8d7bc966-w9kb7             0/2    Pending            0         5s

    ==> v1/Service
    NAME                                              TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)   AGE
    foolhardy-cardinal-prometheus-alertmanager        ClusterIP  10.96.237.137  <none>       80/TCP    5s
    foolhardy-cardinal-prometheus-kube-state-metrics  ClusterIP  None           <none>       80/TCP    5s
    foolhardy-cardinal-prometheus-node-exporter       ClusterIP  None           <none>       9100/TCP  5s
    foolhardy-cardinal-prometheus-pushgateway         ClusterIP  10.96.56.159   <none>       9091/TCP  5s
    foolhardy-cardinal-prometheus-server              ClusterIP  10.96.163.190  <none>       80/TCP    5s

    (중략)
    ~~~

    특별하게 이름을 부여하려면 다음과 같이 설치한다.
    ~~~
    $ helm install --name my-release stable/prometheus
    ~~~

    잘 배포되어 수행되고 있는지 pods을 살펴본다.
    ~~~
    $ kubectl get pods
    
    NAME                                                              READY     STATUS    RESTARTS   AGE
    foolhardy-cardinal-prometheus-alertmanager-8f6567c4f-m8497        2/2       Running   0          2m
    foolhardy-cardinal-prometheus-kube-state-metrics-5bfbd94cclz8nd   1/1       Running   0          2m
    foolhardy-cardinal-prometheus-node-exporter-72ssg                 1/1       Running   0          2m
    foolhardy-cardinal-prometheus-node-exporter-jz7m7                 1/1       Running   0          2m
    foolhardy-cardinal-prometheus-node-exporter-kgpth                 1/1       Running   0          2m
    foolhardy-cardinal-prometheus-pushgateway-8447c8ffcd-b4wcd        1/1       Running   0          2m
    foolhardy-cardinal-prometheus-server-f8d7bc966-w9kb7              2/2       Running   0          2m
    ~~~
1. 서비스 접근하기

    Prometheus의 데시보드를 보기위하여 접속을 해 보도록 한다.  
    현재 서비스는 다음과 같다.
    ~~~
    $ kubectl get services

    NAME                                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    foolhardy-cardinal-prometheus-alertmanager         ClusterIP   10.96.237.137   <none>        80/TCP     17m
    foolhardy-cardinal-prometheus-kube-state-metrics   ClusterIP   None            <none>        80/TCP     17m
    foolhardy-cardinal-prometheus-node-exporter        ClusterIP   None            <none>        9100/TCP   17m
    foolhardy-cardinal-prometheus-pushgateway          ClusterIP   10.96.56.159    <none>        9091/TCP   17m
    foolhardy-cardinal-prometheus-server               ClusterIP   10.96.163.190   <none>        80/TCP     17m
    kubernetes                                         ClusterIP   10.96.0.1       <none>        443/TCP    35d
    ~~~

    prometheus-server가 현재 ClusterIP로 서비스 되고 있다. OKE는 LoadBalancer를 지원하니 LoadBalnacer로 타입을 바꾸어 접속해 보도록 한다.

    ~~~
    $ kubectl edit service foolhardy-cardinal-prometheus-server
    ~~~

    다음의 yaml 파일이 열린다.
    ~~~yaml
    # Please edit the object below. Lines beginning with a '#' will be ignored,
    # and an empty file will abort the edit. If an error occurs while saving this file will be
    # reopened with the relevant failures.
    #
    apiVersion: v1
    kind: Service
    metadata:
    creationTimestamp: 2019-06-21T06:10:04Z
    labels:
        app: prometheus
        chart: prometheus-8.9.2
        component: server
        heritage: Tiller
        release: foolhardy-cardinal
    name: foolhardy-cardinal-prometheus-server
    namespace: default
    resourceVersion: "6311418"
    selfLink: /api/v1/namespaces/default/services/foolhardy-cardinal-prometheus-server
    uid: 3608f424-93eb-11e9-a290-0a580aeda660
    spec:
    clusterIP: 10.96.163.190
    externalTrafficPolicy: Cluster
    ports:
    - name: http
        nodePort: 31235
        port: 80
        protocol: TCP
        targetPort: 9090
    selector:
        app: prometheus
        component: server
        release: foolhardy-cardinal
    sessionAffinity: None
    type: ClusterIP
    status:
    loadBalancer:
        ingress:
        - ip: 129.213.192.103
    ~~~
    위의 항목 중 아래에서 5번째인 **type: ClusterIP** 를 **type: LoadBalancer** 로 변경하고 저장하고 나오면 다음과 같이 변경되었음을 표시하는 로그가 나온다.
    ~~~
    service "foolhardy-cardinal-prometheus-server" edited
    ~~~

    이제 service를 살펴보면 pending 상태에서 외부 IP가 나타나게 된다. 로그에 나타난 외부 IP는 129.213.192.103 이며 포트는 80 포트이다.
    ~~~
    $ kubectl get services
    NAME                                               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
    foolhardy-cardinal-prometheus-alertmanager         ClusterIP      10.96.237.137   <none>        80/TCP         21m
    foolhardy-cardinal-prometheus-kube-state-metrics   ClusterIP      None            <none>        80/TCP         21m
    foolhardy-cardinal-prometheus-node-exporter        ClusterIP      None            <none>        9100/TCP       21m
    foolhardy-cardinal-prometheus-pushgateway          ClusterIP      10.96.56.159    <none>        9091/TCP       21m
    foolhardy-cardinal-prometheus-server               LoadBalancer   10.96.163.190   <pending>     80:31235/TCP   21m
    kubernetes                                         ClusterIP      10.96.0.1       <none>        443/TCP        35d

    $ kubectl get services
    NAME                                               TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)        AGE
    foolhardy-cardinal-prometheus-alertmanager         ClusterIP      10.96.237.137   <none>            80/TCP         21m
    foolhardy-cardinal-prometheus-kube-state-metrics   ClusterIP      None            <none>            80/TCP         21m
    foolhardy-cardinal-prometheus-node-exporter        ClusterIP      None            <none>            9100/TCP       21m
    foolhardy-cardinal-prometheus-pushgateway          ClusterIP      10.96.56.159    <none>            9091/TCP       21m
    foolhardy-cardinal-prometheus-server               LoadBalancer   10.96.163.190   129.213.192.103   80:31235/TCP   21m
    kubernetes                                         ClusterIP      10.96.0.1       <none>            443/TCP        35d
    ~~~

    이제 브라우저를 통해서 해당 IP에 접속해 본다.
    
    ![Alt text](https://monosnap.com/image/islqJqAuoan3C0Y2VCWcwbrWbiQLnZ)

1. Grafana 설치

Prometheus로 모니터링 할 수 있는 정보를 가져올 수 있다면 이를 시각화 할 필요가 있다. 기본적으로 graph 를 지원하나 만족감(?)을 느낄 수가 없고 이에 Grafana를 설치한다.

~~~
$ helm install stable/grafana

NAME:   trendsetting-clam
LAST DEPLOYED: Fri Jun 21 16:07:40 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME                                   AGE
trendsetting-clam-grafana-clusterrole  4s

(중략)

NOTES:
1. Get your 'admin' user password by running:

   kubectl get secret --namespace default trendsetting-clam-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

2. The Grafana server can be accessed via port 80 on the following DNS name from within your cluster:

   trendsetting-clam-grafana.default.svc.cluster.local

   Get the Grafana URL to visit by running these commands in the same shell:

     export POD_NAME=$(kubectl get pods --namespace default -l "app=grafana,release=trendsetting-clam" -o jsonpath="{.items[0].metadata.name}")
     kubectl --namespace default port-forward $POD_NAME 3000

3. Login with the password from step 1 and the username: admin
#################################################################################
######   WARNING: Persistence is disabled!!! You will lose your data when   #####
######            the Grafana pod is terminated.                            #####
#################################################################################
~~~
위와 같이 Grafana가 설치되었다.


1, Grafana 접속

로그에 나와있는 항목을 수행하여 Grafana에 접속을 해 본다.

먼저 패스워드를 알아낸다.
~~~
$ kubectl get secret --namespace default trendsetting-clam-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; 

bgdIt0mCiVvlmOdpQlgFP4nGeWf0DK9KWM5xLxhu
~~~

그 다음 해당 서비스의 포트를 포워딩 한다.
~~~
$ export POD_NAME=$(kubectl get pods --namespace default -l "app=grafana,release=trendsetting-clam" -o jsonpath="{.items[0].metadata.name}")

$ kubectl --namespace default port-forward $POD_NAME 3000

Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
~~~

그리고 localhost:3000 으로 접속을 한다.

![Alt text](https://monosnap.com/image/Iu88YcIwEnppsSmLtFKnnm6Gk50Opm)

로그인을 한다.
- username : admin
- password : 위에서 구한 패스워드 (bgdIt0mCiVvlmOdpQlgFP4nGeWf0DK9KWM5xLxhu)


자세한 Grafana 사용법은 다음에..

---
Reference:
- https://www.prometheusbook.com/MonitoringWithPrometheus_sample.pdf
- https://github.com/helm/charts/tree/master/stable/prometheus

