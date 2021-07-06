---
layout: post
title:  "Deployment 와 DeploymentConfig"
categories: openshift
tags: openshift deployment deploy
---

# Deployment 와 DeploymentConfig

- Deployment : 하나 이상의 ReplicaSet 포함
- DeploymentConfig : 하나 이상의 ReplicationController를 포함


## Deployment

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-openshift
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-openshift
  template:
    metadata:
      labels:
        app: hello-openshift
    spec:
      containers:
      - name: hello-openshift
        image: openshift/hello-openshift:latest
        ports:
        - containerPort: 80
~~~
- 하나 이상의 ReplicaSet 포함
- 배포 라이프 사이클을 위한 기능 지원
- Controller Manager가 배포 프로세스를 관리



## DeploymentConfig
~~~yaml
apiVersion: v1
kind: DeploymentConfig
metadata:
  name: frontend
spec:
  replicas: 5
  selector:
    name: frontend
  template: { ... }
  triggers:
  - type: ConfigChange 
  - imageChangeParams:
      automatic: true
      containerNames:
      - helloworld
      from:
        kind: ImageStreamTag
        name: hello-openshift:latest
    type: ImageChange  
  strategy:
    type: Rolling      
~~~
- 하나 이상의 ReplicationController 포함
- 배포 라이프 사이클을 위한 기능 지원
- Deployer pod가 배포 프로세스를 관리



---
References
- https://docs.openshift.com/container-platform/4.7/applications/deployments/what-deployments-are.html
