---
layout: post
title:  "VirtualBox에서 minikube 실행하기"
categories: k8s
tags: kubernetes minikube virtualbox
---
# VirtualBox 에서 minikube 실행하기

윈도우, 리눅스, 맥에서 원래 minikube는 Virtualbox를 사용한다.  
Host OS에서 VirtualBox를 설치하고 vm으로 minikube를 사용하는 것이다.  
그런데, 윈도우에서 VirtualBox로 리눅스VM을 사용하고,  
그 리눅스VM 안에서 minikube를 사용하려고 하니, 또 그 안에 VirtualBox를 설치해야 한다.

이를 피하기 위한 방법이다.

먼저 minikube를 다운로드 받는다.

~~~
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
~~~

그리고 위치를 옮겨준다.
~~~
sudo cp minikube /usr/local/bin && rm minikube
~~~

시작을 단순히 다음과 같이 하면 VirtualBox가 없다고 오류를 낸다.
~~~
minikube start
~~~
>오류 문구는 다음에..

이를 해결하기 위해서는 --vm-driver=none 옵션을 주어서 시작하면 된다.

그러나!!

또 문제가 발생하는 것이 이렇게 하면 .kube 이하 디렉토리가 모두 root 로 생성이 된다.  
그래서 root 권한이 필요하게 된다.

이를 막기 위해서!! 다음과 같은 환경변수를 minikube start 를 하기 전에 수행한다.
~~~
export CHANGE_MINIKUBE_NONE_USER=true
~~~

항상 적용되도록 .bashrc 에 넣는다.
~~~
echo export CHANGE_MINIKUBE_NONE_USER=true >> ~/.bashrc
~~~

minikube는 다음과 같이 실행한다.
~~~
sudo -E minikube start --vm-driver=none
~~~

그 후 kubectl로 확인한다.
~~~
kubectl cluster-info
~~~

---
>참고 : https://medium.com/@vovaprivalov/setup-minikube-on-virtualbox-7cba363ca3bc