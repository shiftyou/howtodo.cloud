---
layout: post
title:  "docker machine 에 대하여"
categories: docker
tags: docker-machine coherence
---

# Docker Machine에 대하여

docker-machine 은 가상호스트나 관리되는 호스트 서버에 Docker Engine을 설치하고 관리하는 툴이다.  
docker-machine 명령어를 통하여 호스트를 시작,검사,중지 및 재시작하고 Docker 클라이언트과 데몬을 업그레이드 하고 호스트와 통신하도록 Docker 클라이언트를 구성할 수 있다.

# Docker Engine 과 Docker Machine의 차이점

일반적으로 "Docker"라고 말하면 Docker Engine을 말한다. 이는 Server, REST API, Client 를 의미한다.

![Docker Engine](https://docs.docker.com/machine/img/engine.png)

Docker Machine은 Docker Engine이 있는 호스트를 프로비저닝 하고 관리하기 위한 도구이다.

![Docker Machine](https://docs.docker.com/machine/img/machine.png)

Machine을 사용하여 하나 이상의 가상시스템에 Docker Engine을 설치할 수가 있고, 이 가상시스템은 로컬일 수도 있고 리모트 일 수도 있다. 

# machine 생성하기

`docker-machine create` 이라는 명령어로 수행을 한다.  
이때 `-- driver` 플래그를 제공하는 머신을 선택한다.

그래서 Cloud Provider 에 따라 명령어를 작성하는데, AWS를 예를 들자면 다음과 같이 한다.
~~~
$ docker-machine create --driver amazonec2 --amazonec2-access-key AKI******* --amazonec2-secret-key 8T93C*******  aws-sandbox
~~~

machine driver는 다음의 항목이 가능하다.
~~~
Amazon Web Services
Microsoft Azure
Digital Ocean
Exoscale
Google Compute Engine
Linode (unofficial plugin, not supported by Docker)
Microsoft Hyper-V
OpenStack
Rackspace
IBM Softlayer
Oracle VirtualBox
VMware vCloud Air
VMware Fusion
VMware vSphere
VMware Workstation (unofficial plugin, not supported by Docker)
Grid 5000 (unofficial plugin, not supported by Docker)
Scaleway (unofficial plugin, not supported by Docker)
~~~
아쉽게도 OCI는 지원하지 않는다.

그래서 일단, 여기서 STOP!  
:-(
    
---

Reference
- https://docs.docker.com/machine/
