# 설치 스크립트

master 스크립트

```sh
sudo apt-get update
sudo apt-get -y upgrade
sudo apt-get install -y apt-transport-https curl
sudo apt install ca-certificates
sudo apt install software-properties-common

sudo apt install docker.io -y
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install kubelet kubeadm kubectl -y

sudo apt-mark hold kubelet kubeadm kubectl

kubeadm init

mkdir -p $HOME/.kube
sudo \cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



worker 스크립트

```
sudo apt-get update

sudo apt install -y docker.io
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

​                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       

CNI

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

kubectl apply -f https://docs.projectcalico.org/manifests/canal.yaml
```



추가 설치

```
#헬름
sudo snap install helm --classic

#istio
(추가 예정)

#subctl
curl -Ls https://get.submariner.io | bash
sudo cp ~/.local/bin/subctl /usr/bin
sudo rm -rf ~/.local/bin/

#kubectx
git clone https://github.com/ahmetb/kubectx
sudo cp -r kubectx/kube* /usr/bin/
sudo rm -rf ./kubectx

#calicoctl
curl -O -L  https://github.com/projectcalico/calicoctl/releases/download/v3.18.1/calicoctl
chmod +x calicoctl
mv calicoctl /usr/bin

#MetalLB                   
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/metallb.yaml
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```



추가 설정

```
#HostName 변경
read -p "hostname Change is (ex k8s-master): " uhost
sudo hostnamectl set-hostname $uhost

#MetalLB config.yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 10.244.0.0/16
EOF

#Join Token 명령어 생성
LEN=`kubeadm token list | awk 'NR==2{print $1}' | awk '{ print length($1) }'`
if [ $LEN > 0 ];then
        TOKEN=`kubeadm token list | awk 'NR==2{print $1}' | awk 'length($1) > 0 { print $1 }'`
else
        kubeadm token create
        TOKEN=`kubeadm token list | awk 'NR==2{print $1}' | awk 'length($1) > 0 { print $1 }'`
fi

HASH=`openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'`
HOST=`hostname -I | awk {'print $1'}`

cat <<EOF > ./join.sh
kubeadm join ${HOST}:6443 --token ${TOKEN} --discovery-token-ca-cert-hash sha256:${HASH}
EOF

#방화벽 내리기
(추가예정)
```



ansible.yaml

```yaml
---
- hosts: master
  become_user: yes
  tasks:
  - name: update
    apt:
     update_cache: yes
```



reset

```
 kubeadm reset -f &&
  rm -rf /etc/cni /etc/kubernetes /var/lib/dockershim /var/lib/etcd /var/lib/kubelet /var/run/kubernetes ~/.kube/
```



