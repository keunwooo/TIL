

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

![](https://t1.daumcdn.net/cfile/tistory/226BD63C575EBE5910)



### Fluentd를 이용한 로그 수집 아키텍쳐

각 서버에, Fluentd를 설치하면, 서버에서 기동되고 있는 서버(또는 애플리케이션)에서 로그를 수집해서 중앙 로그 저장소 (Log Store)로 전송 하는 방식이다.

![](https://t1.daumcdn.net/cfile/tistory/2672EE3C575EBE6507)



위의 그림은 가장 기본적인 구조로 Fluentd가 로그 수집 에이전트 역할만을 하는 구조인데, 이에 더해서 다음과 같이 각 서버에서 Fluentd에서 수집한 로그를 다른 Fluentd로 보내서 이 Fluentd가 최종적으로 로그 저장소에 저장하도록 할 수 도 있다.



![](https://t1.daumcdn.net/cfile/tistory/2278223C575EBE6606)



중간에 fluentd를 넣는 이유는, 이 fluentd가 앞에서 들어오는 로그들을 수집해서 로그 저장소에 넣기 전에 로그 트래픽을 Throttling (속도 조절)을 해서 로그 저장소의 용량에 맞게 트래픽을 조정을 할 수 있다.

또는 다음 그림과 같이 로그를 여러개의 저장소에 복제해서 저장하거나 로그의 종류에 따라서 각각 다른 로그 저장소로 라우팅이 가능하다.



![](https://t1.daumcdn.net/cfile/tistory/22638D3C575EBE6816)



### Fluentd 내부 구조

Fluentd는 크게 다음 그림과 같이 Input,Parser,Engine,Filter,Buffer,Ouput,Formatter 7개의 컴포넌트로 구성이 된다.  7개의 컴포넌트중 Engine을 제외한 나머지 6개는 플러그인 형태로 제공이 되서 사용자가 설정이 가능하다. 

일반적인 데이타 흐름은 Input → Engine → Output 의 흐름으로 이루어 지고,  Parser, Buffer, Filter, Formatter 등은 설정에 따라서 선택적으로 추가 또는 삭제할 수 있다. 

![](https://t1.daumcdn.net/cfile/tistory/2252EF3C575EBE5B24)





### 데이터 구조



데이타는 크게 3가지 파트로 구성된다. Time, tag, record

- Time : 로그데이타의 생성 시간
- Record : 로그 데이타의 내용으로 JSON형태로 정의된다.
- Tag : 이게 가장 중요한데, 데이타의 분류이다. 각 로그 레코드는 tag를 통해서 로그의 종류가 정해지는데, 이 tag에 따라서 로그에 대한 필터링,라우팅과 같은 플러그인이 적용 된다.





![](https://t1.daumcdn.net/cfile/tistory/25757A3C575EBE5607)







### Helm

---

Kubernetes 패키지 배포를 가능하게 하는 tool

Helm은 복잡한 k8s 리소스들을 간편하게 관리할 수 있도록 도와주는 툴입니다.

하나의 커맨드로 클러스터 내에 리소스들을 설치하고 변경사항을 반영 할 수 있으며, 이러한 변경사항들은 리비전으로 관리할 수 있습니다. 또한, `.tar.gz`확장자로 클러스터 리소스 정의를 패키징하여 원격 저장소를 통해 공유 할 수 있도록 도와줍니다.

헬름은 패키지 관리 도구고, 차트가 리소스를 하나로 묶은 패키지에 해당한다. 헬름으로 차트를 관리하는 목적은 자칫 번잡해지기 쉬운 매니페스트 파일을 관리하기 쉽게 하기 위한 것이다.



### Helm 특징

- 복잡한 어플리케이션 배포 관리
  - Kubernetes 오케스트레이션된 애플리케이션의 배포는 매우 복잡할 수 있습니다. Kubernetes 환경에서 helm 차트는 복잡한 애플리케이션의 배포를 코드로 관리 하여 자동으로 배포할 수 있도록 제공 합니다. 애플리케이션의 빠른 배포를 통하여 다양한 테스트 환경 배포 및 운영 환경 배포 시간을 줄여 개발에 집중 하도록 합니다.
- Hooks
  - Kubernetes 환경에서 helm 차트로 설치, 업그레이드,삭제 그리고 롤백과 같은 애플리케이션 생명주기의 개입할 수 있는 기능을 Hook을 통하여 제공 합니다.
- 릴리즈 관리
  - Helm으로 배포된 애플리케이션은 하나의 릴리즈로 불립니다. 해당 릴리즈는 배포된 애플리케이션의 버전 관리를 가능하도록 합니다.



![](https://tech.osci.kr/assets/images/86027123/0.png)



- **Helm Chart :** Kubernetes에서 리소스를 만들기 위한 템플릿 화 된 yaml 형식의 파일
- **Helm (Chart) Repository :** Helm Repository는 해당 리포지토리에 있는 모든 차트의 모든 메타데이터를 포함하는 저장소. 상황에 따라서, Public Repository를 사용 하거나 내부에 Private Repository를 구성할 수 있습니다.
- **Helm Client(cli) :** 외부의 저장소에서 Chart를 가져 오거나, gRPC로 Helm Server 와 통신하여 요청을 하는 역할을 합니다.
- **Helm Server(tiller) :** Helm Client의 요청을 처리하기 위하여 대기하며, 요청이 있을 경우 Kuberernetes에 Chart를 설치하고 릴리즈를 관리 합니다.



헬름은 클라이언트(cli)와 서버(쿠버네티스 클러스터에 설치되는 틸러)로 구성된다. 클라이언트는 서버를 대상으로 명령을 지시하는 역할을 한다. 서버는 클라이언트에서 전달받은 명령에 따라 쿠버네티스 클러스터에 패키지 설치, 업데이트, 삭제 등의 작업을 수행한다.

 

쿠버네티스는 서비스나 디플로이먼트, 인그레스 같은 리소스를 생성하고 매니페스트 파일을 적용하는 방식으로 애플리케이션을 배포한다. 이 매니페스트 파일을 생성하는 템플릿을 여러 개 패키징한 것이 차트다. 차트는 헬름 리포지토리에 tgz 파일로 저장되며, 틸러가 매니페스트를 생성하는 데 사용한다.



![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FmSsLg%2FbtqzZcTaaYW%2F6sKKvbhnx0A8xY5ZgO8FGk%2Fimg.png)







### etcd

---

etcd는 **분산 key-value store**다. CoreOS에서 coreos 인스턴스의 클러스터를 관리하기 위해서 사용했으며, 구글의 클러스터 컨테이너 관리 소프트웨어인 **Kubernetes**의 백엔드 시스템으로 사용하면서 더 유명해 졌다. 오픈소스로 GitHub에서 다운로드 해서 사용 할 수 있다. etcd는 네트워크로 연결된 노드들 중 리더를 선정해서 클러스터를 관리한다. 데이터는 분산 저장되기 때문에 리더를 포함한 시스템의 오류에 대한 내성을가진다.

주목할 만한 점은 etcd가 [컨테이너 오케스트레이션](https://www.redhat.com/ko/topics/containers/what-is-container-orchestration)을 위한 사실상의 표준 시스템인 [쿠버네티스](https://www.redhat.com/ko/topics/containers/what-is-kubernetes)의 기본 데이터 저장소라는 것입니다.

쿠버네티스의 기본 데이터 저장소인 etcd는 모든 쿠버네티스 클러스터 상태를 저장하고 복제합니다.

 [클라우드 네이티브 애플리케이션](https://www.redhat.com/ko/topics/cloud-native-apps)은 etcd를 사용함으로써 더욱 일관성 있는 가동 시간을 유지하고 개별 서버에 장애가 발생한 상황에서도 작동 상태를 유지할 수 있습니다. 애플리케이션은 etcd에서 데이터를 읽고 쓰며, 이를 통해 설정 데이터를 배포하여 노드 설정에 대한 이중화 및 복구 능력을 제공합니다.

쿠버네티스의 기본 데이터 저장소인 etcd는 모든 쿠버네티스 클러스터 상태를 저장하고 복제합니다. 



### etcd 오퍼레이터

사람의 운영 지식을 대표하는 오퍼레이터를 사용하면 쿠버네티스 또는 [Red Hat OpenShift](https://www.redhat.com/ko/technologies/cloud-computing/openshift)와 같은 쿠버네티스 컨테이너 플랫폼에서 etcd를 더 손쉽게 사용할 수 있습니다. etcd 오퍼레이터는 [오퍼레이터 프레임워크](https://www.redhat.com/ko/blog/introducing-operator-framework-building-apps-kubernetes) 내에서 etcd를 관리하고 etcd 클러스터 설정 및 관리를 간소화합니다.

[etcd 오퍼레이터](https://developers.redhat.com/courses/openshift-operators/manage-etcd/)는 단일 명령으로 설치하고 사용자가 etcd 클러스터를 생성, 설정 및 관리하는 단순한 선언적 설정을 사용해 etcd의 복잡성을 설정하고 관리할 수 있도록 합니다.

etcd 오퍼레이터는 다음과 같은 기능을 제공합니다.

- **생성/제거** - 각 etcd 멤버에 대해 설정 세팅을 반복적으로 지정하는 대신에 사용자는 클러스터의 크기를 지정하기만 하면 됩니다.
- **크기 조정** - 사용자는 사양의 크기만 수정하면 되며, etcd 오퍼레이터가 클러스터 멤버 배포, 제거 및/또는 재설정 작업을 맡아 처리합니다.
- **백업** - etcd 오퍼레이터는 백업을 자동으로 투명하게 수행합니다. 사용자는 백업 정책만 지정하면 됩니다. 예를 들면, *30분마다 백업하고 마지막 3번의 백업을 보관합니다.*
- **업그레이드** - 다운타임 없이 etcd를 업그레이드하는 것은 매우 중요하지만 까다로운 작업입니다. etcd 오퍼레이터를 이용하면 운영이 간소화되고 일반적인 업그레이드 오류를 방지할 수 있습니다.



etcd 오퍼레이터는 사람 오퍼레이터의 행동을 관찰, 분석, 동작이라는 3가지 단계로 시뮬레이션합니다.

1. 오퍼레이터는 쿠버네티스 API를 사용해 현재 클러스터 상태를 관찰합니다.

2. 그런 다음, 원하는 상태와 현재 상태 간의 차이를 파악합니다.

3. 끝으로, etcd 클러스터 관리 API 또는 쿠버네티스 API 중 하나 또는 둘 다를 통해 차이를 수정합니다.

   

etcd는 하나의 마스터(master)와 하나 이상의 팔로워(follower)로 구성된다.



![](https://coreos.com/assets/images/media/Etcd-Replication.png)







### Harbor

---



>What is Harbor?
>
>Harbor is an open source registry that secures artifacts with policies and role-based access control, ensures images are scanned and free from vulnerabilities, and signs images as trusted. Harbor, a CNCF Graduated project, delivers compliance, performance, and interoperability to help you consistently and securely manage artifacts across cloud native compute platforms like Kubernetes and Docker.
>
>
>
>Harbor는 역할 기반 접근제어, 이미지 취약점 스캐닝,이미지 서명 등의 기능을 갖춘 오픈소스 컨테이너 레지스트리이다.
>
>CNCF의 졸업 프로젝트인 Harbor는 Kubernetes와 Docker와 같은 클라우드 네이티브 플랫폼에서 이미지를 안전하고 일관적으로 관리할 수 있는 컬플라이언스와 성능,상호 운영성을 제공합니다.



도커를 사용하다 보면 Docker Registry가 반드시 필요하다. 그 중에서도 기업에서는 private Docker Registry가 반드시 필요하다고 할 수 있다. 

기존에는 Sonatype nexus를 사용했지만 Nexus는 Docker Registry가 주된 기능이 아닌 통합 Repository 개념으로 세분화된 기능을 제공하고 있지는 않다. Docker로 사용하기 위해서는 https 사용할 수 없어 추가적인 설정이 필요하고, Dokcer Hub를 사용한다면 무료로 사용하면 public으로 공개해야 하고 private로 사용하면 비용을 지불하는 문제가 발생한다.

 Harbor는 Private Docker Registry에 적합하게 설계되어 있어 기존 Docker 환경에 추가하여 간단하게 유지 보수가 가능하고 각 projects 별로 구분 관리가 가능하다는 점, 계정별 접근 관리가 가능한 여러 장점을 얻을 수 있기 때문에 Docker를 사용한 서비스 구성 및 운영에 있어서 적합하다.



Harbor는 웹 UI와 역할 기반 접근 제어(Role Based Access Control)을 주요 기능으로 제공한다



![](https://engineering.linecorp.com/wp-content/uploads/2019/11/privatedocker3.png)















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





