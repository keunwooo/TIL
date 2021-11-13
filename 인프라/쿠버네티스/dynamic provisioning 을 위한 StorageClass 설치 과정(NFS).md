# Kubernetes storage Class 설정(NFS)

**실습환경**

- nfs 서버용 vm (4 core/8GB/ 80GB) 1대

- 쿠버네티스(master 3대, worker 3대)

  

**nsf 서버 **

- nfs 서버 설치

  ```shell
  apt-get update && sudo apt-get install -y nfs-kernel-server
  ```

- 공유폴더 생성 및 권한 설정

  ```shell
  mkdir -m 1777 /share
  ```

- nfs 폴더 접근 권한 설정

  ```shell
  vim /etc/exports
  
  /share/ *(rw,sync,no_root_squash,subtree_check)
  
  exportfs -ra
  exportfs
  ```

  - https://sungpil94.tistory.com/198 권한참조

- 2049 포트 방화벽 확인 및 설정 필요

  ```shell
   netstat -lntp |grep 2049
  ```



**nfs 클라이언트(쿠버네티스 vm 설정)**

- nsf 클라이언트 설치(모든 노드에 설정 필요)

  ```shell
   apt-get -y install nfs-common
   mkdir -p /data 
   mount -t nfs 101.79.1.160:/share /data
  #mount -t nfs {nfs 서버 ip}:{nfs 서버 공유 폴더} {nfs 클라이언트 연동 폴더}
   df -h # 연동 확인
  ```

- nfs 연동 확인

  - 폴더에 들어가서 파일에 rw 되는지 확인하새요

- 스토리지 클래스, 프로비저너 설치

  ```shell
  helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
  helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
      --set nfs.server=101.79.1.160 \
      --set nfs.path=/share
  ```

- 스토리지클래스, 프로비저너, sa 등 생성 확인(pod running 확인)

  ```
  kubectl get sc,deployment,pod
  ```

- test(pvc를 생성하여 pv 자동생성 확인)

  ```shell
  vim test-claim.yaml
  
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: nfs-pvc-test
  spec:
    storageClassName: nfs-client # kubectl get sc 로 이름 확인
    accessModes:
      - ReadWriteMany #  must be the same as PersistentVolume
    resources:
      requests:
        storage: 1Gi
  ```
  - kubectl get sc,pv,pvc 해서 bound 됐는지 확인

