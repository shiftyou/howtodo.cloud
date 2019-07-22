---
layout: post
title:  "쿠버네티스의 Ingress"
categories: kubernetes
tags: kubernetes ingress
---


# Ingress 란

HTTP나 HTTPS를 통하여 클러스터 내부의 서비스를 외부로 노출시키는 것이다.
트래픽 라우팅은 ingress resource 를 통해서 제어된다.
~~~
internet
    |
[Ingress]
--|--|--
[Service]
~~~

Ingress 는 다음의 일을 할 수 있다.
- Service 에 외부 URL을 제공한다.
- 트래픽을 로드밸런싱 한다.
- SSL/TLS를 terminate 한다.
- virtual hosting을 지정한다.

Ingress 는 임의로 포트나 프로토콜을 노출시키지 않는다. HTTP 및 HTTPS 이외의 서비스를 인터넷에 노출 시키면 일반적으로 Service.Type = NodePort 또는 Service.Type = LoadBalancer 유형의 서비스가 사용된다.

# Ingress 와 Service 의 차이
- Service
    애플리케이션의 endpoint를 추상화 한 것이 Service 이다. 애플리케이션은 Pod에서 동작하고, 여러 Pod 들은 Service 로 추상화 되어 제공된다.

- Ingress
    Service 들을 외부로 노출시킬때 Ingress 를 쓴다. 여러 Service 들은 Ingress 로 추상화 되어 제공된다.

# Ingress Coltroller

Ingress 로 환경을 설정하고 실제로 일은 Ingress Controller가 수행한다. 그래서 Ingress 리소스를 사용하기 위해서는 클러스터에서 ingress controller 가 수행되고 있어야 한다. 다른 타입의 컨트롤러와 다르게 ingress controller는 kube-controller-manager의 부분으로 속해있다. ingress controller는 자동으로 시작하지는 않는다.

현재 Kubernetes에서 공식적으로 제공하는 Ingress Controller 는 다음의 플랫폼에 배포되어 있다. 
- ingress-gce : Google Compute Engine 용. GCE를 이용하면 자동으로 사용됨
- ingress-nginx : 직접 설치해서 사용할 수 있는 ingress controller (https://kubernetes.github.io/ingress-nginx/)


# 아키텍처

아래의 순서대로 수행을 하면 다음과 같은 아키텍처가 된다.

![Alt text](https://monosnap.com/image/xqkfU47kpgqfQ8JIOsF1cYCJBM6ucF)



# ingress-nginx 배포

다음의 명령어로 ingress-nginx 를 사용할 수 있다.
~~~
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml

namespace "ingress-nginx" created
configmap "nginx-configuration" created
configmap "tcp-services" created
configmap "udp-services" created
serviceaccount "nginx-ingress-serviceaccount" created
clusterrole.rbac.authorization.k8s.io "nginx-ingress-clusterrole" created
role.rbac.authorization.k8s.io "nginx-ingress-role" created
rolebinding.rbac.authorization.k8s.io "nginx-ingress-role-nisa-binding" created
clusterrolebinding.rbac.authorization.k8s.io "nginx-ingress-clusterrole-nisa-binding" created
deployment.apps "nginx-ingress-controller" created
~~~

위와같이 명령을 하면 ingress-nginx 라는 namesapce가 생기고, 이 namespance에 ingress controller를 배포하게 된다. 
~~~
$ kubectl get namespace

NAME            STATUS    AGE
default         Active    62d
ingress-nginx   Active    10m
kube-public     Active    62d
kube-system     Active    62d
~~~

그리고 ingress-nginx라는 namespace에 어떤 pod 이 동작하는지 볼 수 있다.
~~~
$ kubectl get pods -n ingress-nginx

NAME                                        READY     STATUS    RESTARTS   AGE
nginx-ingress-controller-6df4d8b446-6q76s   1/1       Running   0          9m
~~~

# Helm 을 이용한 배포
helm 을 이용하여 ingress-nginx를 배포할 수 있다.
~~~
helm install stable/nginx-ingress --name my-nginx
~~~

# ingress-nginx 노출하기

배포된 ingress-nginx를 Service로 배포한다.
~~~
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud-generic.yaml

service "ingress-nginx" created
~~~

잘 배포되었는지 체크한다.
~~~
$ kubectl get service -n ingress-nginx
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE

ingress-nginx   LoadBalancer   10.96.229.129   129.213.74.19   80:32594/TCP,443:31073/TCP   4m
~~~

내용의 EXTERNAL-IP 로 호출을 해 보면 잘 동작함을 알 수 있다.
~~~
$ curl 129.213.74.19

<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>openresty/1.15.8.1</center>
</body>
</html>
~~~
아직까지는 호출될 애플리케이션이 없어서 오류가 나온다.

# TLS Secret 생성
~~~
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=nginxsvc/O=nginxsvc"

Generating a 2048 bit RSA private key
........................................................+++
........................................................................................+++
writing new private key to 'tls.key'
-----

$ kubectl create secret tls tls-secret --key tls.key --cert tls.crt

secret "tls-secret" created
~~~

# 애플리케이션 배포

테스트로 사용할 애플리케이션을 배포한다.  
애플리케이션은 nginx 를 사용하도록 한다. 아래는 이를 위한 myapp.yaml 파일이다.  replicas를 3으로 두어 3개의 Pod을 생성하고 80포트로 리슨하는 애플리케이션이다. Service 는 8088 포트로 리슨하며 Pod의 80 포트로 연결이 된다.
~~~
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
spec:
  selector:
    app: myapp
  ports:
    - port: 8088
      targetPort: 80
  type: ClusterIP
~~~

myapp.yaml 을 사용하여 애플리케이션을 생성한다.
Deployment 의 이름은 myapp 이며, Service의 이름은 myapp-svc 이다.
~~~
$ kubectl create -f myapp.yaml

deployment.apps "myapp" created
service "myapp-svc" created
~~~


# Ingress 사용

해당 서비스는 ClusterIP 로 서비스 되고 있다.


~~~
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: myapp-ing
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - secretName: tls-secret
  rules:
  - http:
      paths:
      - backend:
          serviceName: myapp-svc
          servicePort: 8088

~~~

~~~
kubectl create -f myapp-ingress.yaml
ingress.extensions "myapp-ing" created
~~~

~~~
curl  http://129.213.74.19
<html>
<head><title>308 Permanent Redirect</title></head>
<body>
<center><h1>308 Permanent Redirect</h1></center>
<hr><center>openresty/1.15.8.1</center>
</body>
</html>
~~~
-k option : not verify the SSL certificates.
~~~
curl -k https://129.213.74.19
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
~~~

-L option : automatically folow the location header
~~~
curl -kL  http://129.213.74.19
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
~~~

~~~
curl -ikL http://129.213.74.19
HTTP/1.1 308 Permanent Redirect
Via: 1.1 10.188.53.7 (McAfee Web Gateway 7.8.2.7.0.28225)
Date: Fri, 19 Jul 2019 07:33:20 GMT
Server: openresty/1.15.8.1
Location: https://129.213.74.19/
Connection: Keep-Alive
Content-Type: text/html
Content-Length: 177

HTTP/2 200
server: openresty/1.15.8.1
date: Fri, 19 Jul 2019 07:33:21 GMT
content-type: text/html
content-length: 612
vary: Accept-Encoding
last-modified: Tue, 25 Jun 2019 12:19:45 GMT
etag: "5d121161-264"
accept-ranges: bytes
strict-transport-security: max-age=15724800; includeSubDomains

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
~~~

![Alt text](https://monosnap.com/image/TJcCudFAgq5RcMnX5Y3uEJdL1Q1Cd0)



동작하는 상태를 체크해 본다.
~~~
kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx --watch
~~~



# 참고

배포된 ingress-nginx를 사용하기 위해서 환경별로 필요한 항목이 있다.
- minikube
    ~~~
    minikube addon enable ingress
    ~~~

- Docker for Mac 

    version 18.06.0-ce 버젼부터 Docker for Mac 에서 Kubernetes가 사용 가능하다.
    ~~~
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud-generic.yaml
    ~~~

- Oracle - OKE (Docker for Mac 와 동일)
    ~~~
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud-generic.yaml
    ~~~

- Google - GKE (Docker for Mac 와 동일)
    ~~~
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud-generic.yaml
    ~~~

- Azure (Docker for Mac 와 동일)
    ~~~
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud-generic.yaml
    ~~~

- AWS
    AWS에서는 Elastic Load Balancer(ELB)가 Type=LoadBalancer 의 Service 뒤에 NGINX Ingress Controller 로 사용되고 있다. L4 및 L7울 사용할 수 있다.
    - Layer 4 : 80 포트와 443 포트용으로 TCP 프로토콜로 리슨한다.
    - Layer 7 : 80 포트와 terminate TLS용으로 HTTP 프로토콜로 리슨한다.

    LayerELB를 사용하기 위해서 다음의 명령을 수행한다.
    ~~~
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/aws/service-l4.yaml
    
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/aws/patch-configmap-l4.yaml
    ~~~

- Bere-metal
    NodePort를 사용한다.
    ~~~
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/service-nodeport.yaml
    ~~~




# 참고
- https://kubernetes.github.io/ingress-nginx/deploy/
- https://github.com/kubernetes/ingress-nginx/blob/master/README.md
- https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/multiple-ingress.md#multiple-ingress-controllers
- https://github.com/kubernetes/ingress-gce/blob/master/docs/faq/README.md#how-do-i-run-multiple-ingress-controllers-in-the-same-cluster
- https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/#additional-controllers
- https://docs.cloud.oracle.com/iaas/Content/ContEng/Tasks/contengsettingupingresscontroller.htm
