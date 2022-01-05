---
layout: post                          # (require) default post layout
title: "Inside OpenShift"                   # (require) a string title
date: 2021-12-14       # (require) a post date
categories: openshift          # (custom) some categories, but makesure these categories already exists inside path of `category/`
tags: openshift inside architecture                      # (custom) tags only for meta `property="article:tag"`
image: Broadcast_Mail.png             # (custom) image only for meta `property="og:image"`, save your image inside path of `static/img/_posts`
---

![](https://assets.openshift.com/hubfs/Google%20Drive%20Integration/Copy%20of%20OCP-OpensourceBLOG-1.png)

OpenShift 4는 100% 오픈 소스이며 몇 가지 주요 CNCF 오픈 소스 프로젝트로 구성됩니다.

## CoreOS

[Linux](https://coreos.com/) - OpenShift는 CoreOS에서 실행되며 Red Hat Enterprise Linux 에코시스템의 지원을 이용합니다. CoreOS 내에는 다음이 있습니다.

[Cri-o](https://cri-o.io/) - OCI(Open Container Initiative) 호환 런타임을 사용하기 위한 Kubernetes CRI(Container Runtime Interface)의 구현입니다. CRI-O는 Kubernetes의 런타임으로 Docker를 사용하는 것보다 가볍습니다.

[Podman](https://podman.io/) - Linux에서 OCI 컨테이너를 개발, 관리 및 실행하기 위한 데몬이 없는 컨테이너 엔진입니다.

[Skopeo](https://github.com/containers/skopeo) - 서로 다른 유형의 컨테이너 저장소 간에 컨테이너 이미지를 이동하기 위한 도구입니다. 예를 들어 컨테이너 레지스트리 docker.io, quay.io 와 내부 컨테이너 레지스트리 또는 다른 유형의 로컬 스토리지 간에 컨테이너 이미지를 복사할때 사용합니다.

[Buildah](https://buildah.io/) - OCI 컨테이너 이미지를 빌드하는 도구입니다.

## 자동화된 작업

[Operator](https://operatorhub.io/) 프레임워크는 애플리케이션 수명 주기의 Day 1 및 Day 2 자동화입니다. 많은 파트너가 허브에서 사용할 운영자를 구축하고 OpenShift에 배포할 수 있습니다.

필요한 애플리케이션을 설치하고 관리하는 부분을 Operator를 통해서 쉽게 할 수가 있습니다. 

## 클러스터 서비스

[Prometheus](https://prometheus.io/) - 오픈 소스 메트릭, 경고 및 모니터링 솔루션입니다.

[Grafana](https://github.com/grafana/grafana) - 메트릭을 쿼리, 시각화, 경고 및 사용자화를 통해서 이해할 수 있도록 해 줍니다.

[Elastic Search](https://github.com/elastic/elasticsearch) - 분산 RESTful 검색 엔진

[FluentD](https://github.com/fluent/fluentd) - 다양한 데이터 소스에서 이벤트를 수집하고 파일, RDBMS, NoSQL, IaaS, SaaS, Hadoop에 기록하여 로깅 인프라를 통합합니다.

[Kibana](https://github.com/elastic/kibana) - Elasticsearch용 브라우저 기반 분석 및 검색 대시보드.

## 개발자 서비스

[Jenkins](https://www.jenkins.io/) - 오픈 소스 자동화 서버인 Jenkins는 모든 프로젝트의 구축, 배포 및 자동화를 지원하는 수백 가지 플러그인을 제공합니다.

[Tekton](https://github.com/tektoncd/pipeline) - Tekton Pipelines 프로젝트는 CI/CD 스타일 파이프라인을 선언하기 위한 k8s 스타일 리소스를 제공합니다.

[코드 준비 작업 공간](https://developers.redhat.com/products/codeready-workspaces/overview) - 웹 기반 IDE인 Eclipse Che

## 응용 서비스

[Istio](https://istio.io/) - 서비스를 연결, 보호, 제어 및 관찰합니다.

[Kiali](https://kiali.io/) - Istio 기반 서비스 메시용 관리 콘솔은 대시보드, 구성 및 검증 기능으로 서비스 메시를 운영하기 위한 관찰 가능성을 제공합니다. 자세한 메트릭, 강력한 유효성 검사, Grafana 액세스 및 Jaeger와의 분산 추적 통합을 제공합니다.

[Jaeger](https://www.jaegertracing.io/) - 오픈 소스, 종단 간 분산 추적. 복잡한 분산 시스템의 트랜잭션 모니터링 및 문제 해결

## 컨테이너 레지스트리

[Quay](https://quay.io/) - 컨테이너 이미지 호스트, 스캔, 서명

## FaaS 서비스(서비스로서의 기능)

[Knative](https://knative.dev/) - 최신 서버리스 워크로드를 배포 및 관리하기 위한 Kubernetes 기반 플랫폼입니다. 이벤트 기반, scale-to-zero 애플리케이션 모델

[Keda](https://github.com/kedacore/keda) - 이벤트 기반 Kubernetes 워크로드를 위한 세분화된 자동 크기 조정(0부터/까지 포함). KEDA는 Kubernetes Metrics Server 역할을 하며 사용자가 전용 Kubernetes 사용자 지정 리소스 정의를 사용하여 자동 크기 조정 규칙을 정의할 수 있도록 합니다.

## PaaS 서비스

[S2i](https://github.com/openshift/source-to-image) - 소스 코드를 컨테이너 이미지로 변환), 이미지 스트림 (컨테이너 레이어의 변경 사항 추적)

## 스토리지

OpenShift 컨테이너 스토리지

[CSI 플러그인](https://github.com/container-storage-interface) - 스토리지 옵션용 플러그인

[Ceph(스토리지)](https://ceph.io/ceph-storage/) - 객체 기반 스토리지

[Rook(Ceph Operator)](https://rook.io/) - 분산 스토리지 시스템을 자가 관리, 자가 확장, 자가 치유 스토리지 서비스로 만듭니다. 배포, 부트스트랩, 구성, 프로비저닝, 확장, 업그레이드, 마이그레이션, 재해 복구, 모니터링 및 리소스 관리와 같은 스토리지 관리자의 작업을 자동화합니다.

## 네트워킹

[OVS](https://www.openvswitch.org/) - OpenvSwitch - 프로그래밍 방식 확장을 통해 네트워크 자동화를 활성화하는 동시에 표준 관리 인터페이스 및 프로토콜을 계속 지원합니다.

[OVN](https://github.com/ovn-org/ovn) - 가상 네트워크 구성을 OpenFlow로 변환하고 Open vSwitch에 설치하는 Open Virtual Network 데몬입니다. 흐름이 아닌 논리적 라우터 및 논리적 스위치와 함께 작동하는 Open vSwitch보다 상위 계층 추상화를 제공합니다.

CNI 플러그인 - 다양한 L2/L3 리눅스 네트워킹

## 가상화

[KubeVirt](https://kubevirt.io/) - Kubernetes에서 기본 Linux 또는 Windows VM 실행

OpenShift는 100% 인증된 Kubernetes입니다.

----

Reference : https://cloud.redhat.com/blog/whats-inside-openshift-4
