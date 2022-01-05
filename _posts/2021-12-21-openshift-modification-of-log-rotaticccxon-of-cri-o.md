---
layout: post
title: "CRI-O를 사용하는 Pods의 log rotation 설정"
date: 2021-12-30
categories: openshift
tags: openshift logging
image: Broadcast_Mail.png
---
CRI-O의 log rotation을 수정하기

# Issue
- CRI-O를 사용하는 pods의 log rotation 설정

# Resolution

두 개의 `machineconfig` 가 kubelet을 위해 존재한다.
~~~
$ oc get machineconfig | grep -i kubelet
01-master-kubelet                                           d5599de7a6b86ec385e0f9c849e93977fcb4eeb8   2.2.0             22h
01-worker-kubelet                                           d5599de7a6b86ec385e0f9c849e93977fcb4eeb8   2.2.0             22h
~~~

OCP4에서 CRI-O 컨테이너 엔진을 사용한다.
pod 로그파일의 크기와 번호는 각 node의 `kubelet`에서 관리한 다음 설정을 CRI-O로 전달되며, 다음의 설정으로 기본 구성되어져 있다.
~~~
--container-log-max-size 50Mi (default 10Mi)
--container-log-max-files 5 (default 5)
~~~
kubelet 설정은 /etc/kubernetes/kubelet.conf 에 위치해 있으며 기본 파라메터는 CRI-O에 의해서 설정되어있다.
~~~
containerLogMaxSize: 50Mi
contianerLogMaxFiles: 5
~~~


worker node의 kubelet의 설정을 변경하려면 custom `kubelet`을 설정한다.   
먼저 'custom-kubelet=logrotation' 이라는 태그를 가지는 worker machineconfigpool을 지정한다.
~~~
$ oc label machineconfigpool worker custom-kubelet=logrotation
machineconfigpool.machineconfiguration.openshift.io/worker labeled
~~~

그리고 logrotation.yaml 을 만든다.
~~~yaml 
apiVersion: machineconfiguration.openshift.io/v1
kind: KubeletConfig
metadata:
  name: cr-logrotation
spec:
  machineConfigPoolSelector:
    matchLabels:
      custom-kubelet: logrotation
  kubeletConfig:
    containerLogMaxFiles: 58
    container-log-max-size: 800Mi
~~~

수행한다.
~~~
$ oc create -f logrotation.yaml
kubeletconfig.machineconfiguration.openshift.io/cr-logrotation created
~~~

검증한다.
~~~
$ oc get kubeletconfig
NAME             AGE
cr-logrotation   6s

$ oc get kubeletconfig -o yaml
apiVersion: v1
items:
- apiVersion: machineconfiguration.openshift.io/v1
  kind: KubeletConfig
  metadata:
    creationTimestamp: "2020-03-24T11:46:11Z"
    finalizers:
    - 99-worker-75c6f8de-867f-471e-8a09-09d05ee48e0d-kubelet
    generation: 1
    name: cr-logrotation
    resourceVersion: "373981"
    selfLink: /apis/machineconfiguration.openshift.io/v1/kubeletconfigs/cr-logrotation
    uid: e0dc2521-25f5-4982-8592-3ec83a6139c9
  spec:
    kubeletConfig:
      container-log-max-size: 800Mi                  ««««««««««««««««««««««««««« MODIFIED.
      containerLogMaxFiles: 58                      ««««««««««««««««««««««««««« MODIFIED.
    machineConfigPoolSelector:
      matchLabels:
        custom-kubelet: logrotation                   ««««««««««««««««««««««««««« MODIFIED.
  status:
    conditions:
    - lastTransitionTime: "2020-03-24T11:46:11Z"
      message: Success
      status: "True"
      type: Success
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
~~~

이제 `machineconfig`는 3개가 보일 것이다.
~~~
$ oc get machineconfig | grep -i kubelet
01-master-kubelet                                           d5599de7a6b86ec385e0f9c849e93977fcb4eeb8   2.2.0             22h
01-worker-kubelet                                           d5599de7a6b86ec385e0f9c849e93977fcb4eeb8   2.2.0             22h
99-worker-75c6f8de-867f-471e-8a09-09d05ee48e0d-kubelet      d5599de7a6b86ec385e0f9c849e93977fcb4eeb8   2.2.0             74s
~~~






