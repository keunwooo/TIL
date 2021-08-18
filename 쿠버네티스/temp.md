# 설치



k8s

```
#! /bin/bash
# sudo로 실행 필요
# docker 설치
apt-get update
apt-get -y install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb [arch=arm64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get -y update
apt-get -y install docker-ce docker-ce-cli containerd.io

# Cgroup를 systemd로 설정
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d
systemctl daemon-reload
systemctl restart docker
sudo systemctl enable docker

# 일반유저 Docker 사용
usermod -aG docker vraptor

apt install bash-completion

curl https://raw.githubusercontent.com/docker/docker-ce/master/components/cli/contrib/completion/bash/docker -o /etc/bash_completion.d/docker.sh

# K8s 설치
# 네트워크 설정
cat <<EOF | tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system

apt-get update && apt-get install -y apt-transport-https curl

curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

apt-get update

apt-get install -y kubelet kubeadm kubectl

apt-mark hold kubelet kubeadm kubectl

source /usr/share/bash-completion/bash_completion

echo 'source <(kubectl completion bash)' >> /home/vraptor/.bashrc

kubectl completion bash >/etc/bash_completion.d/kubectl
```



or

k3s

```
curl -sfL https://get.k3s.io | sh -
```



---

### deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
        name: nginx-deployment
        labels:
                app: nginx
spec:
        replicas: 3
        selector:
                matchLabels:
                        app: nginx
        template:
                metadata:
                        labels:
                         app: nginx
                spec:
                        containers:
                        - name: nginx
                          image: nginx:1.7.9
                          ports:
                          - containerPort: 80

```

---

# service

```
apiVersion: v1
kind: Service
metadata:
 name: http-go-svc
spec:
 ports:
 - port: 80
   targetPort: 8080
 selector:
  app: http-go
  
```

kubectl expose deployment hello-world --type=LoadBalancer --name=my-service

---

# ingress

