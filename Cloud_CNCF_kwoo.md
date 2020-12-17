## CNCF (CLOUD NATIVE COMPUTING FOUNDATION)

2015 년 7 월에 발표된 2016 년 1 월에 정식 출범 한 Cloud Native Computing Foundation (이하 CNCF)는 혼돈스러운 컨테이너와 관련된 다양한 기술적인 문제들을 오픈소스로 해결하는 하는 것을 목표로하고 있습니다.

Cloud Native Computing Foundation 는 대표적으로 Kubernetes 와 Prometheus 와 같은 클라우드 네이티브 오픈소스 기술들을 추진하고 관리하는 단체입니다.



2019년 7월 현재 Graduated Project는 다음과 같습니다.

- Kubernetes ( Orchestration )
- Prometheus ( Monitoring )
- Envoy (Service Proxy)
- CoreDNS (Service Discovery)
- containerd (Container Runtime)
- Fluentd (Logging)



CNCF에서는 프로젝트를 성숙도에 따라서 샌드박스 단계, 인큐베이팅 단계, 졸업 단계로 나눕니다.



졸업 프로젝트

- Kubernetes



**Envoy**

**Fluentd**

**Helm**

**etcd**

**Harbor**

Jaeger

CoreDNS

Prometheus

containerd

Rook

TiKV

TUF

Vitess



### Envoy

---

**Lift**사에서 제작한 프로젝트로, **Cloud Native Computing Foundation**(CNCF)의 세번째 Graduated Project

대형 MSA의 단일 Application과 Service를 위해 설계된 고성능 분산 c++프록시



> 다음의 목적을 가진다.
>
> *네트워크는 애플리케이션에* **투명**해야하며, 장애가 발생했을시 **어디에서 문제가 발생했는지 쉽게 파악**할 수 있어야 한다.



### - Envoy 배포 아키텍처

Envoy 프록시는 배포 위치에 따라서 다양한 기능을 수행할 수 있는데, 크게 다음과 같이 4가지 구조에 배포가 가능하다.

![Envoy 배포](https://t1.daumcdn.net/cfile/tistory/991F05505BF8066419)



### Front envoy proxy

특정 서비스가 아니라, 전체 시스템 앞의 위치하는 프록시로, 클라이언트에서 들어오는 호출을 받아서 각각의 서비스로 라우팅을 한다. URL 기반으로 라우팅을 하는 기능 이외에도, TLS(SSL) 처리를 하는 역할들을 할 수 있다. 통상적으로 nginx나 apache httpd가 리버스프록시로 이 용도로 많이 사용되었다. 

### Service to service ingress listener

특정 서비스 앞에 위치하는 배포 방식으로 서비스로 들어오는 트래픽에 대한 처리를 하는데, 트래픽에 대한 버퍼링이나 Circuit breaker 와 같은 역할을 수행한다.

### Service to service egress listener

특정 서비스 뒤에서 서비스로부터 나가는 트래픽을 통제 하는데, 서비스로 부터 호출 대상이 되는 서비스에 대한 로드 밸런싱, 호출 횟수 통제 (Rate limiting)와 같은 기능을 수행한다.

### External service egress listener

내부서비스에서 외부 서비스로 나가는 트래픽을 관리하는 역할인데, 외부 서비스에 대한 일종의 대행자(Delegator)와 같은 역할을 한다.



시스템 앞 부분이나 또는 시스템을 구성하는 서비스의 앞뒤에 배치할 수 있는 구조지만, 서비스 앞뒤로 붙는다고 실제로 배포를 할때 하나의 서비스 앞뒤로 두개의 envoy proxy를 배치하지는 않는다.

다음과 같이 하나의 서비스에 하나의 Envoy를 배치 한후, ingress/egress 두 가지 용도로 겸용해서 사용한다.

![](https://t1.daumcdn.net/cfile/tistory/99E48D445BF806640C)





### - Envoy 설정 구조

다음은 Envoy 설정 파일을 살펴 보자

Envoy의 설정은 크게 아래 그림과 같이 크게 Listener, Filter, Cluster 세가지 파트로 구성된다. 

![](https://t1.daumcdn.net/cfile/tistory/9937BD4B5BF8066422)



- Listener
  Listener는 클라이언트로 부터 프로토콜을 받는 부분으로, TCP Listener, HTTP Listener 등이 있다. 
- Filter
  Filter는 Listener 로 부터 많은 메시지를 중간 처리하는 부분으로, 압축이나 들어오는 Traffic 에 대한 제한 작업등을 한후, Router를 통해서 적절한 클러스터로 메시지를 라우팅 하는 역할을 한다. 
- Cluster
  Cluster는 실제로 라우팅이 될 대상 서버(서비스)를 지정한다. 



이렇게 Listener를 통해서 메시지를 받고, Filter를 이용하여 받은 메시지를 처리한 후에, 라우팅 규칙에 따라서 적절한 Cluster로 라우팅을 해서 적절한 서비스로 메시지를 보내는 형식이다. 



### - 주요기능



### Out of process architecture

Envoy proxy는 그 자체로 메모리사용량이 적은 고성능의 서버입니다.
모든 프로그래밍 언어, 프레임워크와 함께 실행될 수 있습니다.

이는 다양한 언어,프레임워크를 함께 사용하는 Architecture에 유용히 사용될 수 있습니다.

### L3/L4 Architecture

Envoy의 주요 기능은 L7이지만 핵심은 L3/L4 네트워크 프록시 입니다.
TCP프록시, HTTP프록시, TLS인증과 같은 다양한 작업을 지원합니다.

### L7 Architecture

버퍼링, 속도제한, 라우팅/전달 등과 같은 다양한 작업을 수행할 수 있게 합니다.

### HTTP/2 , gRPC 지원

HTTP/1.1은 물론 HTTP/2도 지원합니다.
이는 HTTP/1.1과 HTTP/2 클라이언트와 서버간 모든 조합을 연결할 수 있음을 의미합니다.

### Advanced Load Balancing

자동 재시도, circuit break, 외부 속도 제한 서비스를 통한 글로벌 속도제한, 이상치 탐지 등의 기능을 제공합니다.



### Etc

- health check 지원
- 프론트/엣지 프록시 지원
- 관리용으로 다양한 통계 정보 제공
- MongoDB, DynamoDB L7을 지원







### Fluentd

---



### Helm

---



### etcd

---



### Harbor

---







인큐베이팅 프로젝트

Argo

KubeEdge

Buildpacks

Linkerd

CloudEvents

NATS

CNI

Notary

Contour

Open Policy Agent

Cortex

OpenTracing



CRI-O

CRI-O는 OCI (Open Container Initiative) 호환 런타임을 사용할 수 있도록 Kubernetes CRI (Container Runtime Interface)를 구현 한 것입니다. Docker를 kubernetes의 런타임으로 사용하는 것에 대한 간단한 대안입니다. 이를 통해 Kubernetes는 포드 실행을위한 컨테이너 런타임으로 OCI 호환 런타임을 사용할 수 있습니다. 현재는 컨테이너 런타임으로 runc 및 Kata 컨테이너를 지원하지만 원칙적으로 모든 OCI 준수 런타임을 연결할 수 있습니다.

CRI-O는 OCI 컨테이너 이미지를 지원하며 모든 컨테이너 레지스트리에서 가져올 수 있습니다. Docker, Moby 또는 rkt를 Kubernetes의 런타임으로 사용하는 것보다 가벼운 대안입니다.



CRI-O is made up of several components that are found in different GitHub repositories.

CRI-O는 서로 다른 GitHub 저장소에있는 여러 구성 요소로 구성됩니다.



- [OCI compatible runtime ](https://github.com/opencontainers/runtime-tools) OCI 호환 런타임
- [containers/storage](https://github.com/containers/storage) 
- [containers/image](https://github.com/containers/image)
- [networking (CNI)](https://github.com/containernetworking/cni)
- [container monitoring (conmon)](https://github.com/containers/conmon)
- security is provided by several core Linux capabilities



![CRI-O](https://cri-o.io/assets/images/architecture.png)



Operator Framework

Dragonfly

SPIFFE

Falco

SPIRE

gRPC

Thanos



샌드박스 프로젝트

Artifact Hub

Backstage

BFE

Bridge

cert-manager

Chaos Mesh

ChubaoFS

Cloud Custodian

Cloud Development Kit for Kubernetes

CNI-Genie

Crossplane

Dex

Flux

in-toto

k3s

KEDA

Keptn

Keylime

KubeVirt

KUDO

Kuma

kyverno

LitmusChaos

Longhorn

metal3-io

Network Service Mesh

Open Service Mesh

OpenEBS

OpenKruise

OpenMetrics

OpenTelemetry

OpenYurt

Parsec

Porter

Pravega

SchemaHero

Serverless Workflow Specification

Service Mesh Interface

Strimzi

Telepresence

Thnkerbell

Tremor

Virtual kublet

Volcano





