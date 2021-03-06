---
layout: post
title:  "Helm으로 mysql 설치하기"
categories: kubernetes
tags: helm
---



개발과 테스트의 환경을 구성하는 시간을 줄이고 프로그램에 좀 더 집중하기 위하여 필요한 것이 Helm 이다.   
node.js의 npm 이나 리눅스의 yum, apt 같은 쿠버네티스의 패키지 메니저라고 보면 된다.  
Chart, Repository, Release를 사용하여 반복적인 응용 프로그램의 설치, 구성 및 버젼관리 방법을 제공한다.

- Chart : 쿠버네티스 클러스터에서 실행되고 검색 가능한 메타데이터로 YAML 파일로 정의된 쿠버네시티 리소스 패키지
- Repository : 검색가능한 Helm Chart의 묶음
- Release : 쿠버네티스 클러스터에 배포된 Helm Chart 인스턴스

아키텍처는 두가지가 구성요소로 되어 있다.

![아키텍처](https://cdn.app.compendium.com/uploads/user/e7c690e8-6ff9-102a-ac6d-e4aebca50425/5a29c3c1-7c6b-41fa-8082-bdc8a36177c9/Image/c64c01d08df64f4420e81f962fd13a23/screen_shot_2018_09_11_at_4_48_19_pm.png)

- The Tiller Server: 쿠버네티스 API와 상호 작용하는 Helm 서버
- The Helm Client : 커맨드 라인 Helm 클라이언트

# 설치

## The Tiller Server 
Oracle Container Engine for Kubernetes (OKE) 를 통해서 쿠베너티스 클러스터를 Quick Create로 생성하면 자동으로 Helm Tiller는 설치된다. 

## The Helm Client
설치는 여러가지 패키지 매니저로 설치할 수 있다.
- [Homebrew](https://brew.sh/) : brew install kubernetes-helm.
- [Chocolatey](https://chocolatey.org/) : choco install kubernetes-helm.
- [Scoop](https://scoop.sh/) : scoop install helm.
- [GoFish](https://gofi.sh/) : gofish install helm.


brew 로 설치를 하면 다음과 같다.
~~~sh
$ brew install kubernetes-helm

Updating Homebrew...
==> Auto-updated Homebrew!
Updated 1 tap (homebrew/core).
==> Downloading https://homebrew.bintray.com/bottles/kubernetes-helm-2.13.1.moja
==> Downloading from https://akamai.bintray.com/30/30f412ad5b85b63edbd373cfca01f
######################################################################## 100.0%
==> Pouring kubernetes-helm-2.13.1.mojave.bottle.tar.gz
==> Caveats
Bash completion has been installed to:
  /usr/local/etc/bash_completion.d

zsh completions have been installed to:
  /usr/local/share/zsh/site-functions
==> Summary
🍺  /usr/local/Cellar/kubernetes-helm/2.13.1: 51 files, 84.2MB
==> `brew cleanup` has not been run in 30 days, running now...
$
 ~~~

# 초기화

Helm 이 준비되었으니, 초기화를 한다.
~~~sh
$ helm init --history-max 200

$HELM_HOME has been configured at /Users/jonggyou/.helm.
Warning: Tiller is already installed in the cluster.
(Use --client-only to suppress this message, or --upgrade to upgrade Tiller to the current version.)
Happy Helming!
~~~

위의 명령어는 원래 쿠버네티스 클러스터의 `kubectl config current-context`에 tiller를 설치하는 것이나, OKE에서는 이미 설치가 되어 있어서 이미 설치가 되어 있다고 알려준다.

Tiller를 업그레이드 하려면 다음과 같이 명령한다.
~~~sh
$ helm init --upgrade

$HELM_HOME has been configured at /Users/jonggyou/.helm.

Tiller (the Helm server-side component) has been upgraded to the current version.
Happy Helming!
~~~

helm의 chart를 최신 리스트로 업데이트 한다.
~~~sh
$ helm repo update

Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
~~~

어떤 솔루션을 설치할 수 있는지는 search 명령어로 알 수 있다.
~~~sh
$ helm search mysql

NAME                            	CHART VERSION	APP VERSION	DESCRIPTION
stable/mysql                    	0.15.0       	5.7.14     	Fast, reliable, scalable, and easy to use open-source rel...
stable/mysqldump                	2.4.0        	2.4.0      	A Helm chart to help backup MySQL databases using mysqldump
stable/prometheus-mysql-exporter	0.3.2        	v0.11.0    	A Helm chart for prometheus mysql exporter with cloudsqlp...
stable/percona                  	0.3.4        	5.7.17     	free, fully compatible, enhanced, open source drop-in rep...
stable/percona-xtradb-cluster   	0.6.6        	5.7.19     	free, fully compatible, enhanced, open source drop-in rep...
stable/phpmyadmin               	2.2.0        	4.8.5      	phpMyAdmin is an mysql administration frontend
stable/gcloud-sqlproxy          	0.6.1        	1.11       	DEPRECATED Google Cloud SQL Proxy
stable/mariadb                  	5.11.1       	10.1.38    	Fast, reliable, scalable, and easy to use open-source rel...

$ helm search tomcat

NAME         	CHART VERSION	APP VERSION	DESCRIPTION
stable/tomcat	0.2.0        	7          	Deploy a basic tomcat application server with sidecar as ...

$ helm search apache

NAME                	CHART VERSION	APP VERSION	DESCRIPTION
stable/hadoop       	1.1.0        	2.9.0      	The Apache Hadoop software library is a framework that al...
stable/ignite       	1.0.0        	2.7.0      	Apache Ignite is an open-source distributed database, cac...
stable/jenkins      	0.37.3       	lts        	Open source continuous integration server. It supports mu...
stable/kafka-manager	1.1.2        	1.3.3.22   	A tool for managing Apache Kafka.
stable/superset     	1.1.5        	0.28.1     	Apache Superset (incubating) is a modern, enterprise-read...
~~~

# 설치

mysql을 설치해 보도록 한다.
~~~sh
$ helm install stable/mysql
~~~

다음과 같이 출력된다.
~~~sh
NAME:   banking-squid
LAST DEPLOYED: Tue Apr 16 10:15:19 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                      DATA  AGE
banking-squid-mysql-test  1     3s

==> v1/PersistentVolumeClaim
NAME                 STATUS   VOLUME  CAPACITY  ACCESS MODES  STORAGECLASS  AGE
banking-squid-mysql  Pending  oci     3s

==> v1/Pod(related)
NAME                                 READY  STATUS   RESTARTS  AGE
banking-squid-mysql-685b45dfc-4dbkm  0/1    Pending  0         3s

==> v1/Secret
NAME                 TYPE    DATA  AGE
banking-squid-mysql  Opaque  2     4s

==> v1/Service
NAME                 TYPE       CLUSTER-IP    EXTERNAL-IP  PORT(S)   AGE
banking-squid-mysql  ClusterIP  10.96.194.55  <none>       3306/TCP  3s

==> v1beta1/Deployment
NAME                 READY  UP-TO-DATE  AVAILABLE  AGE
banking-squid-mysql  0/1    1           0          3s


NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
banking-squid-mysql.default.svc.cluster.local

To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default banking-squid-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h banking-squid-mysql -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following command to route the connection:
    kubectl port-forward svc/banking-squid-mysql 3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}

~~~

같은 `helm install stable/mysql` 을 수행하면 다른 이름의 mysql이 배포된다. install을 할 때마다 이름이 다르게 배포되어 여러번 배포할 수 있다.

현재 helm으로 배포되어 있는 항목을 보면 다음과 같다.
~~~sh
$ helm ls

NAME         	REVISION	UPDATED                 	STATUS  	CHART       	APP VERSION	NAMESPACE
banking-squid	1       	Tue Apr 16 10:15:19 2019	DEPLOYED	mysql-0.15.0	5.7.14     	default
~~~

배포 되었는지 확인 해 보면 다음과 같다.
~~~sh
$ kubectl get all

NAME                                      READY     STATUS    RESTARTS   AGE
pod/banking-squid-mysql-685b45dfc-4dbkm   1/1       Running   0          1m

NAME                          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/banking-squid-mysql   ClusterIP   10.96.194.55   <none>        3306/TCP   1m
service/kubernetes            ClusterIP   10.96.0.1      <none>        443/TCP    11m

NAME                                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/banking-squid-mysql   1         1         1            1           1m

NAME                                            DESIRED   CURRENT   READY     AGE
replicaset.apps/banking-squid-mysql-685b45dfc   1         1         1         1m
~~~
pod, service, deployment, replicaset 이 모두 등록 되어 있다.

install 할때 나온 순서대로 실행을 해 본다.
1. MYSQL_ROOT_PASSWORD 얻기
    ~~~sh
    $ MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default banking-squid-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)
    ~~~

    설정되어있는지 확인을 해 본다.
    ~~~sh
    $ echo $MYSQL_ROOT_PASSWORD
    EMum3INBxr
    ~~~

1. MYSQL 클라이언트로 사용할 Ubuntu 리눅스 pod을 생성하고 실행한다.
    ~~~sh
    $ kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il
    ~~~

    위의 명령어는 ubuntu:16.04 이미지를 가지고 컨테이너를 만들고 bash를 수행한 것이다. 다른 창을 열어서 확인 해 보면 다음과 같이 pod 이 생성된 것을 알 수 있다.
    ~~~sh 
    $ kubectl get pods

    NAME                                      READY     STATUS    RESTARTS   AGE
    pod/banking-squid-mysql-685b45dfc-4dbkm   1/1       Running   0          58m
    pod/ubuntu                                1/1       Running   0          53s
    ~~~

    혹시 해당 pod을 빠져나갔다면 다음과 같이 수행한다.
    ~~~sh
    $ kubectl exec -it ubuntu -- bash -il
    ~~~
    앞의 run 으로 수행한 것은 images를 가져와서 새로운 pod을 만드는 것이고,  
    exec 로 수행한 것은 기존에 만들어진 pod에 접속하는 것이다.

    화면에 응답이 없으면 Enter를 쳐본다. 다음과 같이 프롬프트가 뜬다.
    ~~~sh
    root@ubuntu:/# 
    ~~~

1. mysql 클라이언트를 설치한다.

    
    ~~~sh
    root@ubuntu:/# apt-get update && apt-get install mysql-client -y
    ~~~

1. 마지막으로 mysql에 접속해 본다.

    ~~~sh
    root@ubuntu:/# mysql -h banking-squid-mysql -p

    Enter password:
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 22152
    Server version: 5.7.14 MySQL Community Server (GPL)

    Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    mysql>
    ~~~
    패스워드는 이전에 MYSQL_ROOT_PASSWORD 환경변수로 기록되어 있다.

1. 로컬 컴퓨터에서 접속

    로컬 컴퓨터에서 접속하기 위해서는 해당 service의 포트를 포워딩 해준다.  
    우선 환경변수로 사용할 항목은 다음과 같다.
    - MYSQL_HOST=127.0.0.1
    - MYSQL_PORT=3306

    현재 service는 다음과 같다.
    ~~~sh
    $ kubectl get svc

    NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
    banking-squid-mysql   ClusterIP   10.96.194.55   <none>        3306/TCP   1d
    kubernetes            ClusterIP   10.96.0.1      <none>        443/TCP    1d
    ~~~

    그리고 포트포워딩은 다음과 같이 한다. 
    ~~~sh
    $ kubectl port-forward svc/banking-squid-mysql 3306

    Forwarding from 127.0.0.1:3306 -> 3306
    Forwarding from [::1]:3306 -> 3306
    ~~~

    그 후, 로컬 머신에서 mysql client를 사용하여 접속이 가능하다.
    ~~~sh
    $ mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
    ~~~~

이로써 mysql을 kubernetes에 설치하고 접속이 가능한 환경을 구성하였다.

---

Reference
- [Take the Helm: Kubernetes Package Management](https://blogs.oracle.com/cloudnative/helm-kubernetes-package-management)
- [Creating a Cluster with Oracle Cloud Infrastructure Container Engine for Kubernetes](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/oke-full/index.html)
- [Helm for Git](https://github.com/helm/helm)
- [Helm Quickstart Guide](https://helm.sh/docs/using_helm/#quickstart-guide)
