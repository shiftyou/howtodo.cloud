#  VM 접속

[여기](./connection.md)를 참조하여 VM에 접속합니다.

# docker 테스트

접속된 터미널로 다음의 명령어를 입력해 봅니다.
- docker version
- docker images
- docker ps -a

# docker image 만들기

## Step 1. 간단한 웹 프로그램
>source : https://nodejs.org/ko/docs/guides/nodejs-docker-webapp/

간단한 node.js 앱을 만들도록 하겠습니다.

1. Node.js 앱 생성

    우선, 모든 파일을 넣은 새로운 디렉터리를 만들겠습니다. 이 디렉터리 안에 애플리케이션과 의존성을 알려주는 `package.json` 파일을 생성하겠습니다.
    ~~~
    $ mkdir test
    $ cd test
    $ nano package.json
    ~~~

    에디터가 열리면 아래의 내용을 복사합니다. 붙여넣기는 putty 창에서 오른쪽 마우스를 클릭하면 됩니다.

    ~~~json
    {
        "name": "docker_web_app",
        "version": "1.0.0",
        "description": "Node.js on Docker",
        "author": "First Last <first.last@example.com>",
        "main": "server.js",
        "scripts": {
            "start": "node server.js"
        },
        "dependencies": {
            "express": "^4.16.1"
        }
    }
    ~~~
    F3 키를 누르거나 Ctrl-O 키를 눌러 저장합니다.<br>
    F2 키를 누르거나 Ctrl-X 키를 눌러 종료합니다.

    이제 Express.js 프레임워크로 웹앱을 정의하는 `server.js`를 만들겠습니다.

    ~~~
    $ nano server.js
    ~~~

    에디터가 열리면 아래의 내용을 복사합니다. 붙여넣기는 putty 창에서 오른쪽 마우스를 클릭하면 됩니다.

    ~~~js
    'use strict';

    const express = require('express');

    // 상수
    const PORT = 8080;
    const HOST = '0.0.0.0';

    // 앱
    const app = express();
    app.get('/', (req, res) => {
        res.send('Hello world\n');
    });

    app.listen(PORT, HOST);
    console.log(`Running on http://${HOST}:${PORT}`);
    ~~~

    F3 키를 누르거나 Ctrl-O 키를 눌러 저장합니다.<br>
    F2 키를 누르거나 Ctrl-X 키를 눌러 종료합니다.

    다음 단계에서 공식 Docker 이미지를 사용해서 Docker 컨테이너 안에서 이 앱을 실행하는 방법을 살펴보겠습니다. 먼저 앱의 Docker 이미지를 만들어야 합니다.

1. Dockerfile 생성

    Dockerfile이라는 빈 파일을 생성합니다.
    ~~~sh
    $ nano Dockerfile
    ~~~

    가장 먼저 해야 할 것은 어떤 이미지를 사용해서 빌드할 것인지를 정의하는 것입니다. 여기서는 Docker Hub에 있는 node의 최신 LTS(장기 지원) 버전인 8을 사용할 것입니다.
    ~~~dockerfile
    FROM node:8
    ~~~

    다음으로 이미지 안에 애플리케이션 코드를 넣기 위해 디렉터리를 생성할 것입니다. 이 디렉터리가 애플리케이션의 워킹 디렉터리가 됩니다.
    ~~~dockerfile
    # 앱 디렉터리 생성
    RUN mkdir -p /usr/src/app
    WORKDIR /usr/src/app
    ~~~

    이 이미지에는 이미 Node.js와 NPM이 설치되어 있으므로 npm 바이너리로 앱의 의존성을 설치하기만 하면 됩니다.
    ~~~dockerfile
    # 앱 의존성 설치
    COPY package*.json ./
    RUN npm install
    ~~~

    Docker 이미지 안에 앱의 소스코드를 넣기 위해 COPY 지시어를 사용합니다.
    ~~~dockerfile
    # 앱 소스 추가
    COPY . .
    ~~~

    앱이 8080포트에 바인딩 되어 있으므로 EXPOSE 지시어를 사용해서 docker 데몬에 매핑합니다.
    ~~~dockerfile
    EXPOSE 8080
    ~~~

    마지막으로 런타임을 정의하는 CMD로 앱을 실행하는 중요 명령어를 정의해야 합니다. 여기서는 서버를 구동하도록 node server.js을 실행하는 기본 npm start을 사용할 것입니다.
    ~~~dockerfile
    CMD [ "npm", "start" ]
    ~~~

    Dockerfile은 다음과 같아야 합니다.
    ~~~dockerfile
    FROM node:8

    # 앱 디렉터리 생성
    WORKDIR /usr/src/app

    # 앱 의존성 설치
    COPY package*.json ./
    RUN npm install

    # 앱 소스 추가
    COPY . .

    EXPOSE 8080
    CMD [ "npm", "start" ]
    ~~~

    F3 키를 누르거나 Ctrl-O 키를 눌러 저장합니다.<br>
    F2 키를 누르거나 Ctrl-X 키를 눌러 종료합니다.

1. .dockerignore 파일

    Dockerfile과 같은 디렉터리에 .dockerignore 파일을 만듭니다.
    
    ~~~
    $ nano .dockerignore
    ~~~

    그리고 다음 내용으로 만드세요.
    ~~~dockerfile
    node_modules
    npm-debug.log
    ~~~

    F3 키를 누르거나 Ctrl-O 키를 눌러 저장합니다.<br>
    F2 키를 누르거나 Ctrl-X 키를 눌러 종료합니다.

    이는 Docker 이미지에 로컬 모듈과 디버깅 로그를 복사하는 것을 막아서 이미지 내에서 설치한 모듈을 덮어쓰지 않게 합니다.

1. 이미지 빌드

    작성한 Dockerfile이 있는 디렉터리로 가서 Docker 이미지를 빌드하는 다음 명령어를 실행하세요. -t 플래그로 이미지에 태그를 추가하기 때문에 나중에 docker images 명령어로 쉽게 찾을 수 있습니다.
    ~~~sh
    $ docker build -t ${USER}/node-web-app .
    ~~~

    Docker가 당신이 빌드한 이미지를 보여줄 것입니다.
    ~~~sh
    # 컨테이너 이미지의 리스트를 봅니다.
    $ docker images
    REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
    user1/node-web-app   latest              19353c1ef997        18 minutes ago      896MB
    node                 8                   4f01e5319662        2 weeks ago         893MB
    ~~~

1. 이미지 실행

    -d로 이미지를 실행하면 분리 모드로 컨테이너를 실행해서 백그라운드에서 컨테이너가 돌아가도록 합니다. -p 플래그는 공개 포트를 컨테이너 내의 비밀 포트로 리다이렉트합니다. 앞에서 만들 이미지를 실행하세요.
    ~~~sh
    $ docker run -p 808+사용자번호:8080 -d ${USER}/node-web-app
    ~~~

    앱의 로그를 출력하세요.
    ~~~sh
    # 컨테이너 수행합니다.
    $ docker run -p 8081:8080 -d ${USER}/node-web-app
    937a89a4045699746db568a2ebc095d7da5530a4ebec1ffe496a53611712650e

    # 컨테이너 아이디를 확인합니다
    $ docker ps
    CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS                    NAMES
    937a89a40456        user1/node-web-app   "npm start"         21 seconds ago      Up 20 seconds       0.0.0.0:8081->8080/tcp   clever_wright

    # 앱 로그를 출력합니다
    $ docker logs 937a89a40456

    > docker_web_app@1.0.0 start /usr/src/app
    > node server.js

    Running on http://0.0.0.0:8080
    ~~~

    컨테이너 안에 들어가 봐야 한다면 exec 명령어를 사용할 수 있습니다.
    ~~~sh
    # 컨테이너에 들어갑니다
    $ docker exec -it <컨테이너ID> /bin/bash
    ~~~

1. 테스트

    앱을 테스트하려면 Docker 매핑된 앱 포트를 확인합니다.
    ~~~sh
    # 사용중인 컨테이너 리스트를 봅니다.
    $ docker ps
    CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS                    NAMES
    937a89a40456        user1/node-web-app   "npm start"         9 minutes ago       Up 9 minutes        0.0.0.0:8081->8080/tcp   clever_wright
    ~~~

    위 예시에서 Docker가 컨테이너 내의 8080 포트를 머신의 8081 포트로 매핑했습니다.

    이제 curl로 앱을 호출할 수 있습니다.(필요하다면 sudo apt-get install curl로 설치하세요.)
    ~~~sh
    $ curl -i localhost:8081
    HTTP/1.1 200 OK
    X-Powered-By: Express
    Content-Type: text/html; charset=utf-8
    Content-Length: 12
    ETag: W/"c-M6tWOb/Y57lesdjQuHeB1P/qTV0"
    Date: Mon, 25 Feb 2019 02:10:44 GMT
    Connection: keep-alive

    Hello world
    ~~~

    간단한 Node.js 애플리케이션을 Docker로 실행하는데 이 튜토리얼이 도움되었길 바랍니다.

1. 컨테이너 중지

    수행중인 컨테이너를 중지합니다.
    ~~~sh
    $ docker stop <컨테이너ID> 
    ~~~

1. 컨테이너 삭제

    중지된 컨테이너를 삭제합니다. 컨테이이너를 삭제하는 것이지 컨테이너 이미지를 삭제하는 것은 아닙니다.
    ~~~sh
    $ docker rm <컨테이너ID> 
    ~~~
1. 컨테이너 확인

    현재 수행중인 컨테이너의 리스트를 확인합니다.
    ~~~
    $ docker ps -a
    ~~~
