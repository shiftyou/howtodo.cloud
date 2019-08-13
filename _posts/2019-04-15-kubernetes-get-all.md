---
layout: post
title:  "kubectl로 모든 리소스를 한꺼번에 보기"
categories: kubernetes 
tags: kubectl get
---

모든 리소스를 삭제하려면 다음과 같이 명령한다.
~~~
kubectl get all
~~~

all 은 모든 리소스에 대한 all.

그라나 특정 리소스(secret 등)은 보이지 않는다.

