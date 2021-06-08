# CKA



context  사용

```
kubectl config use-context wk8s
```



```
kubectl config current-context
```



---

Pod 사용

```
kubectl run some-pod --image=nginx --dry-run=client -o yaml > some-pod.yaml
```



deployment 사용

```
kubectl create deployment some-deploy --image=nginx \
  --dry-run=client -o yaml > some-deploy.yaml
```



service 생성

```
kubectl expose pod some-pod \
  --type=ClusterIP \
  --port=80 \
  --target-port=8000 \
  --dry-run=client -o yaml > some-pod-svc.yaml
  
---
kubectl expose deployment some-deploy \
  --type=ClusterIP \
  --port=80 \
  --target-port=8000 \
  --dry-run=client -o yaml > some-deploy-svc.yaml
```



---

kubeadm 클러스터 구축



---

etcd 백업 복구

```
etcdctl \
  # --endpoints=https://[127.0.0.1]:2379 \  # 기본값
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /tmp/some-snapshot.db
```



```
etcdctl \
  # --endpoints=https://[127.0.0.1]:2379 \  # 기본값
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --name=master \
  --data-dir /var/lib/etcd-from-backup \
  --initial-cluster=master=https://127.0.0.1:2380 \
  --initial-advertise-peer-urls=https://127.0.0.1:2380 \
  snapshot restore /tmp/some-snapshot.db
```



---

클러스터 장애 해결

클러스터가 문제가 있는 경우를 크게 2가지로 생각해볼 수 있다.

1. `kubectl`을 통해 정보를 확인할 수 없는 경우
2. `kubectl get nodes` 결과에서 특정 노드 상태가 `READY`가 아닌 경우

(1번)은 당연히 kube-apiserver 문제일 가능성이 크다. 하지만 그 전에 `kubectl`이 사용하는 설정 파일에 문제가 없는지 확인해보자.
연습 문제에서는 kube-apiserver URL에 포트가 정확하지 않은 경우가 있었다. (2379 -> 6443)
만약 마스터 노드에서 `docker ps` 명령어로 실행중인 컨트롤 플레인 컴포넌트 중 kube-apiserver가 보이지 않는다면 Static POD 실행 오류로 짐작할 수 있다.
보통 Static POD 명세는 `/etc/kubenetes/manifests` 폴더 안에 있지만, 이 경로를 결정하는 것은 kubelet 옵션이라는 점을 반드시 기억해야 한다.

(2번) 경우는 가장 먼저 kubelet 실행 상태가 정상인지 확인하는 것이 우선이다. kubelet 프로세스가 정상인지 실행 옵션에 문제가 있는지 확인해보자.

