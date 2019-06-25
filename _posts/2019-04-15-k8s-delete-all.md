---
layout: post
title:  "kubectl로 모든 리소스를 한꺼번에 삭제하기"
categories: kubernetes 
tags: kubectl delete
---


모든 리소스를 삭제하려면 다음과 같이 명령한다.
~~~
kubectl delete all --all
~~~

첫번째 all 은 모든 리소스에 대한 all,  
두번째 --all 은 이름대신 모든 리소스

그라나 secret 은 삭제되지 않는다.

