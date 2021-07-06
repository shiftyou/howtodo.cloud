---
layout: post
title:  "Kubernetes Cluster 구성하기"
categories: kubernetes
tags: kubeadm
---

사용환경
- Oracle Cloud Infrastructure 
- Ubuntu 16.04

Subnet
- Master 
	- 10.0.10.0/16


## Master 

Master로 사용할 인스턴스의 Public IP로 접속을 한다.  
우분투 이미지로 만들었을 시에 username을  ubuntu로 접근해야 한다.
접속 완료되면 root로 바꿔서 다음작업을 수행한다.
~~~
$ sudo su -
~~~

먼저 라이브러리를 업데이트 및 업그레이드 한다.
~~~
$ apt-get update
$ apt-get upgrade -y
~~~

도커를 설치한다.
~~~
$ apt-get install docker.io
~~~

kubernetes 를 설치하기 위해 필요한 소스와 툴을 설치한다.
~~~
$ apt-get install -y apt-transport-https curl 

$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - 

$ cat <<EOF >/etc/apt/sources.list.d/kubernetes.list 
deb https://apt.kubernetes.io/ kubernetes-xenial main 
EOF
~~~
kubeadm, kubelet, kubectl을 설치한다.
~~~
$ apt-get update
$ apt-get install -y kubelet kubeadm kubectl
$ apt-mark hold kubelet kubeadm kubectl
~~~

도커를 systemd 로 cgroupdrive 지정한다.
~~~
$ cat > /etc/docker/daemon.json <<EOF 
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

$ systemctl daemon-reload
$ systemctl restart docker
~~~

kubeadm을 초기화 한다. 이때 옵션으로 pod 이 사용할 cidr를 지정한다.
~~~
$ kubeadm init  --pod-network-cidr 10.0.0.0/16 
~~~

위의 초기화가 끝나고 나면 kubernetes를 위한 docker image를 다운로드 받아 실행하게 된다. 그러나 coredns 에서 pending 인 상태로 있게된다.
이를 해결하기 위하여 다음을 수행한다.

kubectl을 수행하기 위해서 설정한다.
- 일반사용자
	~~~
	$ mkdir -p $HOME/.kube 
	$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
	$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
	~~~
- root 일 경우
	~~~
	$ export KUBECONFIG=/etc/kubernetes/admin.conf
	~~~

현재 상태를 살펴본다.
~~~
$ kubectl get pods -A

NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-5c98db65d4-5jmmv         0/1     Pending   0          4h38m
kube-system   coredns-5c98db65d4-tqdpj         0/1     Pending   0          4h38m
kube-system   etcd-master                      1/1     Running   0          4h37m
kube-system   kube-apiserver-master            1/1     Running   0          4h38m
kube-system   kube-controller-manager-master   1/1     Running   0          4h38m
kube-system   kube-proxy-n8f76                 1/1     Running   0          4h38m
kube-system   kube-scheduler-master            1/1     Running   0          4h37m
~~~

위와 같이 kube-system 네임스페이스에서 여러가지 컨테이너가 동작함을 알 수 있다. coredns 라는 컨테이너는 현재 Pending 상태이다. 이를 해결하기 위해서 다음을 수행한다.

coredns의 yaml 파일에서 loop를 제외한다. 다음 명령을 하여 보여지는 yaml 파일에서 loop 항목을 #loop 로 수정한다.
~~~
$ kubectl edit cm coredns -n kube-system
~~~
    
flannel CNI를 설치한다.
~~~
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml 

podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds-amd64 created
daemonset.apps/kube-flannel-ds-arm64 created
daemonset.apps/kube-flannel-ds-arm created
daemonset.apps/kube-flannel-ds-ppc64le created
daemonset.apps/kube-flannel-ds-s390x created
~~~

잠시 기다리면 모든 컨테이너가 정상적으로 수행될 것이다.  
현재 모든 컨테이너가 수행중인지 확인한다.
~~~
$ kubectl get pods -A 

NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-5c98db65d4-5jmmv         1/1     Running   0          4h44m
kube-system   coredns-5c98db65d4-tqdpj         1/1     Running   0          4h44m
kube-system   etcd-master                      1/1     Running   0          4h43m
kube-system   kube-apiserver-master            1/1     Running   0          4h44m
kube-system   kube-controller-manager-master   1/1     Running   0          4h44m
kube-system   kube-flannel-ds-amd64-gxvrk      1/1     Running   0          32s
kube-system   kube-proxy-n8f76                 1/1     Running   0          4h44m
kube-system   kube-scheduler-master            1/1     Running   0          4h43m
~~~

## Worker

Worker 노드에서는 다음과 같이 kubernetes cluster에 join 한다.
~~~
$ kubeadm join 10.0.10.6:6443 --token t5eilq.o7je7mmlyf75z8xc --discovery-token-ca-cert-hash sha256:ce97b157890433e2bca8458d5ea3c84764c23834277d380cf9e4131c601eee57
~~~

- token 은 다음과 같이 알 수 있다.
	
	~~~
	$ kubeadm token list
	~~~

- token-ca-cert-hash는 다음과 같이 알 수 있다.

	~~~
	$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null |    openssl dgst -sha256 -hex | sed 's/^.* //'
	~~~


# 참고
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
- https://github.com/coreos/flannel
