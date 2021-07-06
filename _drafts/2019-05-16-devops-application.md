---
layout: post
title:  "Jenkins 이용하여 Docker Image 만들기"
categories: devops docker
tags: cicd docker jenkins devops
---

Jenkins를 이용하여 개발된 애플리케이션을 Docker Image를 만들고 Registry에 올리는 과정을 설명한다.

# 애플리케이션 개발

main.js
~~~js
// load the http module
var http = require('http');

// configure our HTTP server
var server = http.createServer(function (request, response) {
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.end("Hello World\n");
});

// listen on localhost:8000
server.listen(8000);
console.log("Server listening at http://127.0.0.1:8000/");
~~~

package.json
~~~json
{
  "name": "hellonode",
  "version": "1.0.0",
  "description": "A Hello World HTTP server",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node main.js"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/jonggyou/project1/"
  },
  "keywords": [
    "node",
    "docker",
    "dockerfile"
  ],
  "license": "ISC"
}
~~~


# 컨테이너 이미지 생성

Dockerfile
~~~dockerfile
# use a node base image
FROM node:7-onbuild

# set a health check
HEALTHCHECK --interval=5s \
            --timeout=5s \
            CMD curl -f http://127.0.0.1:8000 || exit 1

# tell docker what port to expose
EXPOSE 8000
~~~

# 컨테이너 생성

docker build 를 사용하여 myapp 이라는 컨테이너를 생성한다.
~~~
$ docker build -t myapp .

Sending build context to Docker daemon  98.82kB
Step 1/3 : FROM node:7-onbuild
7-onbuild: Pulling from library/node
ad74af05f5a2: Pull complete 
2b032b8bbe8b: Pull complete 
a9a5b35f6ead: Pull complete 
3245b5a1c52c: Pull complete 
afa075743392: Pull complete 
9fb9f21641cd: Pull complete 
3f40ad2666bc: Pull complete 
49c0ed396b49: Pull complete 
7af304825012: Pull complete 
Digest: sha256:e506d4de7f21fc0cf51e2d2f922eb0349bd2c07f39dd6335e4338f92c9408994
Status: Downloaded newer image for node:7-onbuild
# Executing 5 build triggers
 ---> Running in 123e8375d3d7
Removing intermediate container 123e8375d3d7
 ---> Running in 0f1b3f049759
Removing intermediate container 0f1b3f049759
 ---> Running in dae11b14fb5a
npm info it worked if it ends with ok
npm info using npm@4.2.0
npm info using node@v7.10.1
npm info lifecycle getintodevops-hellonode@1.0.0~preinstall: getintodevops-hellonode@1.0.0
npm info linkStuff getintodevops-hellonode@1.0.0
npm info lifecycle getintodevops-hellonode@1.0.0~install: getintodevops-hellonode@1.0.0
npm info lifecycle getintodevops-hellonode@1.0.0~postinstall: getintodevops-hellonode@1.0.0
npm info lifecycle getintodevops-hellonode@1.0.0~prepublish: getintodevops-hellonode@1.0.0
npm info lifecycle getintodevops-hellonode@1.0.0~prepare: getintodevops-hellonode@1.0.0
npm info ok 
npm info it worked if it ends with ok
npm info using npm@4.2.0
npm info using node@v7.10.1
npm WARN using --force I sure hope you know what you are doing.
npm info ok 
Removing intermediate container dae11b14fb5a
 ---> 65f52e52f426
Step 2/3 : HEALTHCHECK --interval=5s             --timeout=5s             CMD curl -f http://127.0.0.1:8000 || exit 1
 ---> Running in 4e8b069d2b65
Removing intermediate container 4e8b069d2b65
 ---> 08857e1c58c3
Step 3/3 : EXPOSE 8000
 ---> Running in 3f8f7247f249
Removing intermediate container 3f8f7247f249
 ---> a6bb420ef484
Successfully built a6bb420ef484
Successfully tagged myapp:latest
~~~

잘 만들어졌는지 확인한다.
~~~
$ docker images myapp
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
myapp               latest              a6bb420ef484        About a minute ago   660MB
~~~

테스트 해 본다.
~~~
$ docker run -it -p 8000:8000 --name myapp myapp

npm info it worked if it ends with ok
npm info using npm@4.2.0
npm info using node@v7.10.1
npm info lifecycle hellonode@1.0.0~prestart: hellonode@1.0.0
npm info lifecycle hellonode@1.0.0~start: hellonode@1.0.0

> hellonode@1.0.0 start /usr/src/app
> node main.js

Server listening at http://127.0.0.1:8000/
~~~

웹브라우저나 curl로 테스트 해 본다.
~~~
$ curl localhost:8000
Hello world
~~~

Ctrl-C 를 눌러 종료한다.

# 이미지 저장소에 컨테이너 이미지 등록

docker image registry에 접근 하기위하여 docker login 을 한다.
~~~
$ docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: <username. not email>
Password: <password>
WARNING! Your password will be stored unencrypted in /home/user1/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
~~~

기존에 로그인을 했으면 자동으로 로그인 된다.


Registry에 저장하기 위하여 태그명을 {dock.com의 username}/myapp 으로 바꿔준다.  
그냥 myapp 만의 이름으로 docker image registry에 등록은 docker.io/library 에 등록이 되므로 등록을 할 수 없다.  
자신의 registry에 등록하기 위하여 **{dock.com의 username}/** 를 tag에 붙여준다.
~~~
$ docker tag myapp jonggyou/myapp
~~~

확인해 본다.
~~~
$ docker images jonggyou/myapp
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jonggyou/myapp      latest              326381f9bf6c        3 minutes ago       660MB
~~~

Registry에 등록한다.
~~~
$ docker push jonggyou/myapp

The push refers to repository [docker.io/jonggyou/myapp]
fd1d70d4be70: Pushed 
d01ce6c11e8e: Pushed 
40c4d8abe838: Pushed 
2895be281ac1: Pushed 
ab90d83fa34a: Pushed 
8ee318e54723: Pushed 
e6695624484e: Pushed 
da59b99bbd3b: Pushed 
5616a6292c16: Pushed 
f3ed6cb59ab0: Pushed 
654f45ecb7e3: Pushed 
2c40c66f7667: Pushed 
latest: digest: sha256:11c85a6160abc32643071adf1443c4318df46dbed91ff3a28dade9b269a3f0dd size: 2836
~~~

등록이 완료되면 hub.docker.com 에서 이미지를 볼 수 있다.

![](https://shiftyou.github.io/cloudnative/images/application1.png)


# Jenkins로 컨테이너 이미지 생성 및 레지스트리에 자동으로 등록

지금까지 로컬머신에서 명령한 모든것이 CI/CD 툴인 jenkins로 가능하다.

## docker.com 접속을 위한 Credentail 생성

jenkins에서 docker.com 에 접속하기 위해서 username/password 를 저장해 놓는다

1. Credentials 를 선택하고 gloval을 선택한다.

    ![](https://shiftyou.github.io/cloudnative/images/application2.png)

1. 화면이 나오면 좌측의 Add Credentials 를 선택한다.

    ![](https://shiftyou.github.io/cloudnative/images/application3.png)

    - Username : docker.com 의 로그인 유저이름 입력
    - Password : docker.com 의 로그인 패스워드 입력
    - ID : "docker-hub" 입력  **(이 부분이 jenkins 내에서 docker hub로 접속하기 위한 포인트)**
    - Description : "docker hub" 입력

1. 완료되면 다음과 같은 화면이 나온다.

    ![](https://shiftyou.github.io/cloudnative/images/application4.png)

1. 같은 방법으로 github 에 접속하기 위한 Credential도 만든다.
    - Username : github.com 의 로그인 유저이름 입력
    - Password : github.com 의 로그인 패스워드 입력
    - ID : "github" 입력
    - Description : "github" 입력


## Pipeline 생성

이번 단계는 pipeline을 생성해서 최종 docker registry에 이미지를 push 하는 과정이다.

1. Jenkins 홈에서 좌측의 New Item을 클릭한다.

    ![](https://shiftyou.github.io/cloudnative/images/application5.png)

1. 이름에 "myapp"을 입력하고 "Pipeline"을 선택한 다음 OK를 누른다.

    ![](https://shiftyou.github.io/cloudnative/images/application6.png)


1. 아래 Pipeline에 다음과 같이 설정한다.

    ![](https://shiftyou.github.io/cloudnative/images/application9.png)
    

    - Definition : Pipeline script from SCM
    - SCM : Git 를 선택
        - Repository URL : 자신의 Github 프로젝트의 URL
        - Credentials : 앞 스텝에서 만들었던 Github 로그인용 Credential 선택

    Save 를 누른다.

## Jenkinsfile 생성

Jenkinsfile을 github에서 읽도록 pipeline에서 설정을 했기 때문에 Jenkinsfile을 만들어야 한다.

1. 아래와 같이 Jenkinsfile을 만든다.
    ~~~
    node {
        def app

        stage('Clone repository') {
            /* Let's make sure we have the repository cloned to our workspace */

            checkout scm
        }

        stage('Build image') {
            /* This builds the actual image; synonymous to
            * docker build on the command line */

            app = docker.build("jonggyou/myapp-jenkins")
        }

        stage('Test image') {
            app.inside {
                sh 'echo "Tests passed"'
            }
        }

        stage('Push image') {
            /* Finally, we'll push the image with two tags:
            * First, the incremental build number from Jenkins
            * Second, the 'latest' tag.
            * Pushing multiple tags is cheap, as all the layers are reused. */
            docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
                app.push("${env.BUILD_NUMBER}")
                app.push("latest")
            }
        }
    }
    ~~~

    - docker.build() 항목에서 자신의 docker hub계정을 앞에 적어준다.
    - docker.withRegistry() 항목에서 자신이 만든 credential의 id를 적어준다.

1. github에 push 하기
    ~~~
    $ git add .

    $ git commit -m update

    [master b87a651] update
    1 file changed, 1 insertion(+), 4 deletions(-)

    $ git push
    
    Username for 'https://github.com': kimjonggyou@gmail.com
    Password for 'https://kimjonggyou@gmail.com@github.com': 
    Counting objects: 3, done.
    Delta compression using up to 2 threads.
    Compressing objects: 100% (3/3), done.
    Writing objects: 100% (3/3), 304 bytes | 304.00 KiB/s, done.
    Total 3 (delta 2), reused 0 (delta 0)
    remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
    To https://github.com/jonggyou/project1
    f5a647b..b87a651  master -> master
    ~~~

1. Jenkins 에서 Build Now 를 눌러 동작하기

    좌측 메뉴의 Build Now 를 눌러 파이프라인을 실행한다.

    ![](https://shiftyou.github.io/cloudnative/images/application10.png)

    각 단계마다 걸린 시간과 성공여부가 나타나게 된다.

1. docker hub 확인

    docker hub 를 확인 해 보면 mysql-jenkins 이미지가 등록되어있음을 알 수 있다.

    ![](https://shiftyou.github.io/cloudnative/images/application11.png)


# 쿠버네티스에 애플리케이션 배포


# 스케일 아웃
