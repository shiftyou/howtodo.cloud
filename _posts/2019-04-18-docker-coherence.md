---
layout: post
title:  "Docker 에서 Oracle Coherence 실행하기"
categories: docker
tags: docker coherence
---
# Docker 에서 Oracle Coherence 실행하기

## 기본 저장가능한 DefaultCacheServer로 시작하기
Oracle 이 배포하는 Coherence의 이미지를 수행하기 위한 모든 Argument는 다음과 같다.
~~~
docker run [docker-args] \
   [-e COH_WKA=<wka-address>] \
   [-e JAVA_OPTS=<opts>] \
   [-e COH_EXTEND_PORT=<port>] \
   [-v <lib-dir>:/lib ] \
   [-v <config-dir>:/conf] \
   store/oracle/coherence:12.2.1.3 [type] [args]
~~~
위에 보는 모든 Argument는 모두 옵션이다.  

기본적으로 다음과 같이 수행할 수 있다.
~~~
docker run -d store/oracle/coherence:12.2.1.3
~~~

## Clustering

도커와 같은 컨테이너 환경은 멀티캐스트 Coherence를 사용할 수가 없기 때문에 Well-Know-Address를 사용하여야 한다. **COH_WKA** 환경이 이를 지원는 것이며 다음과 같이 사용한다.
~~~
docker run -d -e COH_WKA=foo.oracle.com store/oracle/coherence:12.2.1.3
~~~~
`foo.oracle.com` 주소가 클러스터를 찾는데 사용될 것이며 이는 DNS lookup

<kbd>Reference
- https://hub.docker.com/_/oracle-coherence-12c
- https://github.com/oracle/docker-images/tree/master/OracleCoherence
