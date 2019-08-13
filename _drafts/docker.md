---
title: "Docker"
date: 2017-08-10T16:07:55+09:00
draft: false

tags: ["DevOps", "docker"]
categories: ["Development"]

---
# Docker

![Docker](https://www.docker.com/sites/default/files/social/docker-facebook-share.png "source: https://www.docker.com/sites/default/files/social/docker-facebook-share.png")

- [Q. Docker의 base image 는 꼭 필요한가?](#Q1) 
- [Q. Docker의 base image 만드는 방법은?](#Q2)
- [Q. Docker의 image 는 어떤것이 제공되나?](#03)
- [Q. Docker의 가장작은 image 는?](#Q4)
- [Q. Docker Container 끼리 연결하는 방법은?](#Q5)
- [Q. Docker Container 끼리 통신하는 방법은?](#Q6)

# 기본 개념
간단하게 그림으로 먼저 설명해 보자

## 가상화 머신(Virtual Machine) Diagram
![Virtual Machine diagram](https://www.docker.com/sites/default/files/VM%402x.png "source:https://www.docker.com/sites/default/files/VM%402x.png")
가상 컴퓨터는 게스트 운영 체제를 실행한다. 하이퍼바이저 위에 각 박스마다 게스트 운영 체제를 구동하고 어플리케이션은 이 OS위에서 운영된다.
디스크 이미지 및 응용 프로그램 상태는 OS 설정과 설치된 시스템에 종속적이며, 디스크에 모든 것이 닮긴 형태이므로 OS 보안 패치 및 기타 사항을 쉽게 잃을 수 있고 복제하기가 쉽다.

## 컨테이너(Container) Diagram
![Container Diagram](https://www.docker.com/sites/default/files/Container%402x.png "source:https://www.docker.com/sites/default/files/Container%402x.png")
컨테이너는 하이퍼바이저 대신 컨테이너 엔진(Docker 엔진)이 존재하고 실제적인 OS는 Host OS만 존재한다. Host OS의 커널을 통해 리소스를 제어하게 된다. 
컨테이너는 단일 커널을 공유 할 수 있으며 컨테이너 이미지에 있어야하는 정보는 실행 파일과 패키지 종속성이며 호스트 시스템에 절대 설치할 필요가 없다. 이러한 프로세스는 네이티브 프로세스와 같이 실행되며, Linux에서 ps를 실행하여 활성 프로세스를 확인하는 것처럼 docker ps와 같은 명령을 실행하여 개별적으로 프로세스를 관리 할 수 있다. 그리고, 모든 종속성을 포함하기 때문에 구성 얽힘이 없다. 컨테이너 화 된 앱은 "어디에서나 실행된다."

위 그림에서 보듯이 VM은 OS단위이고, Container는 라이브러리 단위이다.


같이 알아야 할 것들..
- Container - Docker 데몬위에서 돌아가는 하나의 컨테이너들 (docker container ls 로 확인)
- Service - Container 들의 묶음 (docker service ls 으로 확인)
- Stack 


일단 기본적인 명령어들..
~~~sh
$docker build -t friendlyname .  # Create image using this directory's Dockerfile
$docker run -p 4000:80 friendlyname  # Run "friendlyname" mapping port 4000 to 80
$docker run -d -p 4000:80 friendlyname         # Same thing, but in detached mode
$docker container ls                                # List all running containers
$docker container ls -a             # List all containers, even those not running
$docker container stop <hash>           # Gracefully stop the specified container
$docker container kill <hash>         # Force shutdown of the specified container
$docker container rm <hash>        # Remove specified container from this machine
$docker container rm $(docker container ls -a -q)         # Remove all containers
$docker image ls -a                             # List all images on this machine
$docker image rm <image id>            # Remove specified image from this machine
$docker image rm $(docker image ls -a -q)   # Remove all images from this machine
$docker login             # Log in this CLI session using your Docker credentials
$docker tag <image> username/repository:tag  # Tag <image> for upload to registry
$docker push username/repository:tag            # Upload tagged image to registry
$docker run username/repository:tag                   # Run image from a registry
~~~
~~~bash
$docker exec -it  <image id> /bin/bash                       # execute bash shell
~~~

## Docker 실행 문법

- docker login
- docker tag <image> username/repository:tag
- docker push username/repository:tag
- docker build
    - docker build -t \<dockerfile> : Dockerfile로 부터 image를 만듦
    - docker build --tag hello . : 현재 디렉토리의 Dockerfile로 부터 이미지를 만듦. 태그는 hello
- docker run
    - docker run username/repository:tag
    - docker run -i : STDIN 을 연결
    - docker run -t : TTY를 확보 (일반적으로 -it 로 씀)
    - docker run -v : Volume 을 마운트 (ex: docker run -v $PWD:/build username/repository:tab)
    - docker run --rm : 수행하고 난 뒤 exit를 하면 자동으로 container 를 삭제
    
        ```bash
        $docker run -p 4000:90 friendlyname
        $docker run -d -p 4000:80 friendlyname
        ```
- docker container
    - docker container ls
    - docker container ls -a
    - docker container stop \<hash>
    - docker container kill \<hash>
    - docker container rm \<hash>

        ```sh
        $docker container rm $(docker container ls -a -q)
        ```
- docker image
    - docker image ls -a
    - docker image rm \<image id>
    - docker image rm $(docker image ls -a -q)
- docker swarm
    - docker swarm init
    - docker swarm join
    - docker swarm leave

        ```bash
        $docker swarm leave --force .
        ```
    - docker swarm update
- docker stack

    A stack is a collection of services that make up an application in a specific environment. Learn more about stacks for Docker Cloud here. A stack file is a file in YAML format that defines one or more services, similar to a docker-compose.yml file for Docker Compose but with a few extensions. The default name for this file is docker-cloud.yml.

    - docker stack deploy
        
        ```bash
        $docker stack deploy -c docker-compose.yml getstartedlab
        ```
    - docker stack ls
    - docker stack ps
    - docker stack rm

        ```bash
        $docker stack rm getstartedlab
        ```
    - docker stack services
- docker service
    - docker service create
    - docker service inspect
    - docker service logs
    - docker service ls
    - docker service ps \<service>
    - docker service rm
    - docker service scale
    - docker service update
- docker inspect
    
    ```bash
    $docker inspect --format='{{.Status.ContainerStatus.ContainerID}}' <task>

    $docker inspect --format="{{index .Config.Labels \"com.docker.swarm.task.id\"}}" <container>
    ```




# Q&A
<a name="Q01"></a>
## Q. Docker의 base image 만드는 방법은?

Docker의 base image를 만드는 방법은 두가지가 있다.
첫번째는 전체 Full image를 만드는 방법이고, 두번째는 Docker가 제공하는 최소한의 이미지인 scratch를 이용하는 방법이다.

scratch 는 Docker에서 사용하는 가장 작은 이미지이다.
Dockerfile을 다음과 같이 구성한다.
```dockerfile
FROM scratch
ADD hello /
CMD ["/hello"]
```
**scratch** 이미지로 부터 hello 라는 디렉토리를 만들고 이동하는 docker 이다.

처음부터 빈 이미지에서 출발하는 것이 아니라 대부분이 무언가 필요에 의해서 미리 설치된 Docker Image를 기본으로 자신의 애플리케이션을 설치하기 때문에 [Docker Repository](http://hub.docker.com)에서 검색한 후에 설치하는 것이 좋다.

개인이 만든 이미지 보다는 공식적으로 Distributed된 이미지들을 다운로드 받는 것이 좋다.



<a name="Q02"></a>
## Q. Docker의 빈 images는?

Docker의 빈 이미지를 **scratch** 이미지라고 부른다. 빈 이미지는 다음과 같이 만든다.
```sh
$tar cv --files-from /dev/null | sudo docker import - scratch
```
scratch 이미지는 /dev/null 로 만들어져 아무것도 없기 때문에 아무것도 실행 할 것이 없다. 가장 작은 단위의 이미지인 것이다.

docker 홈페이지의 Get Started 에서 처음으로 수행하는 Docker는 다음과 같다.
```sh
$docker run hello-world
```
이것은 scratch 이미지에서 hello-world를 찍는 아주 간단한 것만 존재한다.


<a name="Q03"></a>
## Q. Docker의 image 는 어떤것이 제공되나?

http://hub.docker.com 은 docker의 공식적인 저장소이다. 

