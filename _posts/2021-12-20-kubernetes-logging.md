---
layout: post
title: "Kubernetes Logging"
date: 2021-12-30
categories: openshift
tags: openshift logging
image: Broadcast_Mail.png
---
# Kubernetes Logging

애플리케이션이 어떻게 동작하고 있는지를 쉽게 알 수 있는 방법은 로그이다. 일반적으로 로그는 stdout으로 남기거나 파일로 남기며 오류시에는 stderr로 남긴다. 로그를 애플리케이션이 동작하는 서버에 파일로 남겨서 관리자가 볼 수 있지만, 쿠버네티스 환경에서는 애플리케이션이 컨테이너로 동작하기 때문에 컨테이너 안에서 남긴 로그 파일은 컨테이너가 사라지면 같이 사라진다.

따라서 컨테이너화된 애플리케이션에 가장 쉽고 가장 많이 채택된 로깅 방법은 stdout 및 stderr에 쓰는 것이다. 아래와 같이 pod에서 stdout/stderr로 쓰여진 로그를 볼 수 있다. 

~~~sh 
kubectl logs {pod명}
~~~

# 노드 수준에서 로깅

컨테이너화된 애플리케이션의 로그는 stdout/stderr로 로그를 남기면 컨테이너 엔진은 logging driver에 의해서 로그들을 redirect애서 node의 어딘가에 저장을 한다. 아래 그림에는 log-file.log 에 저장을 한다. 그래서 log-file.log는 node에 저장되게 된다. 

![](https://d33wubrfki0l68.cloudfront.net/59b1aae2adcfe4f06270b99a2789012ed64bec1f/4d0ad/images/docs/user-guide/logging/logging-node-level.png)

컨테이너가 재시작되거나 삭제되면 kubelet은 이 로그를 삭제시킨다. 그리고 로그를 저장하는 node의 공간이 부족하면 위험하므로 logrotate가 로그들을 재정비한다.


컨테이너 및 Pod, 노드들은 언제든지 제거가 될 수 있기 때문에 독립적인 별도의 스토리지 및 생명주기가 있어야 한다.




# 클러스터 수준 로깅 아키텍처

클러스터 수준 로깅 



## 노드 로깅 에이전트 사용

![](https://d33wubrfki0l68.cloudfront.net/2585cf9757d316b9030cf36d6a4e6b8ea7eedf5a/1509f/images/docs/user-guide/logging/logging-with-node-agent.png)


## 로깅 에이전트와 함까 사이트카 컨테이너 사용

![](https://d33wubrfki0l68.cloudfront.net/5bde4953b3b232c97a744496aa92e3bbfadda9ce/39767/images/docs/user-guide/logging/logging-with-streaming-sidecar.png)


# CRI-O 로그 검사
CRI-O 컨테이너 엔진은 systemd service로 구현이 되어 있다 따라서 `journalctl` 명령어를 사용하여 CRI-O를 위한 로그 메시지를 살펴볼 수 있다.
cri-o의 메시지는 해당 pod에 접속한 후에 ....









----
Reference : *https://kubernetes.io/docs/concepts/cluster-administration/logging/*	

