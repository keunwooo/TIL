스크립트

---

**VMware Player 설치**

vmware 공식홈 Workstation 릴리즈노트에서 버전별로 다운가능하며
(https://my.vmware.com/en/web/vmware/downloads/details?downloadGroup=PLAYER-1555&productId=800&rPId=55787)
여기에서 15.5.0 버전을 다운로드 및 설치함

ubuntu 설치
https://releases.ubuntu.com/18.04.5/?_ga=2.158996730.1931960103.1599736552-1888727197.1599548933

SSH Server 설치
terminal => sudo apt install openssh-server

---

**Bosh VM 배포**

PaaS-TA 설치 파일 다운로드
sudo apt install curl
mkdir workspace
cd workspace
curl -Lo paasta-5.0.zip https://nextcloud.paas-ta.org/index.php/s/Qi2zGPnGNEjb4Ax/download
unzip paasta-5.0.zip
mv bosh-lite paasta-5.0
zip파일을 정상적으로 풀렸는지 확인 후
rm paasta-5.0.zip   (zip 파일이 약 8GB 용량을 차지하므로 향후 vboxmange 로 save 정상적 수행 및 bosh 와 paasta 가 기동시 깨지지 않도록 하기 위함)

------(중요) paasta-5.0.zip 압축 풀은 후 바로 아래 사항 수정바랍니다------------------------
cd /home/ubuntu/workspace/paasta-5.0/deployment/paasta-deployment/
cp -pr deploy-bosh-lite.sh deploy-bosh-lite.sh_backup
vi deploy-bosh-lite.sh  에서
      xip.io 를  nip.io 로 수정해주시기 바랍니다. (수정할 곳은 파일안에서 아래와 같이 총 8군데임)
        -v system_domain=10.244.0.34.nip.io \
        -v uaa_login_logout_redirect_parameter_whitelist=["http://portal-web-user.10.244.0.34.nip.io","http://portal-web-user.10.244.0.34.nip.io/callback","http://portal-web-user.10.244.0.34.nip.io/login"] \
        -v uaa_login_links_passwd="http://portal-web-user.10.244.0.34.nip.io/resetpasswd" \
        -v uaa_login_links_signup="http://portal-web-user.10.244.0.34.nip.io/createuser" \
        -v uaa_clients_portalclient_redirect_uri="http://portal-web-user.10.244.0.34.nip.io,http://portal-web-user.10.244.0.34.nip.io/callback" \

---

**Bosh 설치**
curl -Lo ./bosh https://s3.amazonaws.com/bosh-cli-artifacts/bosh-cli-6.1.0-linux-amd64 
chmod +x ./bosh 
sudo mv ./bosh /usr/local/bin/bosh 
bosh -v

종속성 파일 설치
sudo apt-get install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt1-dev libxml2-dev libssl-dev libreadline7 libreadline-dev libyaml-dev libsqlite3-dev sqlite3 

VirtualBox 6.0 설치
sudo apt update
sudo apt upgrade 
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
sudo add-apt-repository "deb http://download.virtualbox.org/virtualbox/debian bionic contrib" 
sudo apt update 
sudo apt install virtualbox-6.0 
VBoxManage --version 

Deploy Bosh VM
cd ~/workspace/paasta-5.0/deployment/bosh-deployment
chmod 755 *.sh
./deploy-bosh-lite.sh

**Alias 설정**
bosh alias-env micro-bosh -e 10.0.1.6 --ca-cert <(bosh int warden/creds.yml --path /director_ssl/ca) 

**Bosh login 쉘 스크립트 생성**
vi login.sh

```
#!/usr/bin

export BOSH_CA_CERT=$(bosh int ./warden/creds.yml --path /director_ssl/ca)
export BOSH_CLIENT=admin
export BOSH_CLIENT_SECRET=$(bosh int ./warden/creds.yml --path /admin_password) 
```

**Bosh login 쉘 적용**
source login.sh



**Bosh login 확인**
bosh -e micro-bosh env



**jumpbox key 생성 (Bosh VM 로그인 용) **
bosh int warden/creds.yml --path /jumpbox_ssh/private_key > jumpbox.key 
chmod 600 jumpbox.key
ssh jumpbox@10.0.1.6 -i jumpbox.key



**Bosh로 배포된 프로그램 process 확인 (bosh vm or paas-ta vm ssh 접속 후 이용)**
sudo su
monit summary 



credhub cli 설치
cd ~
wget https://github.com/cloudfoundry-incubator/credhub-cli/releases/download/2.0.0/credhub-linux-2.0.0.tgz
tar -xvf credhub-linux-2.0.0.tgz
chmod +x credhub 
sudo mv credhub /usr/local/bin/credhub 
credhub --version



**credhub shell 생성**
cd ~/workspace/paasta-5.0/deployment/bosh-deployment
vi credhub_login.sh

```
#!/usr/bin

export CREDHUB_CLIENT=credhub-admin 
export CREDHUB_SECRET=$(bosh int --path /credhub_admin_client_secret warden/creds.yml) 
export CREDHUB_CA_CERT=$(bosh int --path /credhub_tls/ca warden/creds.yml) 
```

source credhub_login.sh



**credhub 로그인**
credhub login -s https://10.0.1.6:8844 --skip-tls-validation 



**vbox 환경저장**
vboxmanage controlvm $(bosh int warden/state.json --path /current_vm_cid) savestate



---

**PaaS-TA 배포**

**Virtual Box vm 복구**
vboxmanage startvm $(bosh int warden/state.json --path /current_vm_cid) --type headless



**Bosh login**
cd ~/workspace/paasta-5.0/deployment/bosh-deployment
source login.sh



**update cloud config (bosh-lite-cloud-config.yml 수정 필요)**
cd ~/workspace/paasta-5.0/deployment/cloud-config
vi bosh-lite-cloud-config.yml

```
asis
  - disk_size: 30720

tobe
- disk_size: 30720
```

bosh -e micro-bosh update-cloud-config bosh-lite-cloud-config.yml

**runtime config 등록**
cd ~/workspace/paasta-5.0/deployment/bosh-deployment
./update-runtime-config.sh
bosh -e micro-bosh configs

**stemcell 등록**
cd ~/workspace/paasta-5.0/stemcell/paasta
bosh -e micro-bosh upload-stemcell bosh-stemcell-315.64-warden-boshlite-ubuntu-xenial-go_agent.tgz
bosh -e micro-bosh stemcells

**PaaS-TA 배포**
cd ~/workspace/paasta-5.0/deployment/paasta-deployment
chmod +x ./*.sh
vi deploy-bosh-lith.sh

```
as-is
bosh -d paasta -n deploy paasta-deployment.yml

to-be
bosh -e micro-bosh -d paasta -n deploy paasta-deployment.yml
```

./deploy-bosh-lite.sh

IP route 설정 (bosh-lite를 이용할때)
sudo ip route add   10.244.0.0/16 via 10.0.1.6

---

**기타 실습**

**기타 알아두어야 할 Bosh 명령어들**
bosh -e micro-bosh instances
bosh -e micro-bosh env --details
bosh -e micro-bosh stemcells
bosh -e micro-bosh releases
bosh -e micro-bosh tasks --recent
bosh -e micro-bosh locks
bosh -e micro-bosh cancel-task 123
bosh -e micro-bosh -d paasta vms --vitals
bosh -e micro-bosh -d paasta ssh api

cf-cli 설치
wget -q -O - https://packages.cloudfoundry.org/debian/cli.cloudfoundry.org.key | sudo apt-key add -
echo "deb https://packages.cloudfoundry.org/debian stable main" | sudo tee /etc/apt/sources.list.d/cloudfoundry-cli.list
sudo apt update
sudo apt install cf-cli
cf -v

**cf login**
cf login --skip-ssl-validation
https://api.10.244.0.34.nip.io
(동영상에는 https://api.10.244.0.34.xip.io 로 되어 있으나 실행은 https://api.10.244.0.34.nip.io  으로 해주세요)

**user 생성**
cf create-user edu-user user

**org생성**
cf create-org edu-org
cf orgs

**space**
cf create-space -o edu-org edu-space
cf target -o edu-org -s edu-space  (아니면 cf target -o edu-org)
cf spaces

**org role 설정**
cf set-org-role edu-user edu-org OrgManager

**space role 설정**
cf set-space-role edu-user edu-org edu-space SpaceDeveloper

java8  설치
sudo apt update
sudo apt install openjdk-8-jdk
java -version

git 설치
sudo apt install git
git --version

spring-music 다운로드 및 빌드
cd ~/workspace
git clone https://github.com/cloudfoundry-samples/spring-music
cd spring-music/
./gradlew clean assemble

manifest 수정 -> 아무것도 조작안하고 나옴
vi manifest.yml

**cf target 변경**
cf target -o edu-user -s edu-space

spring-music 배포
cf push
cf apps

**로그 확인**
cf app spring-music
cf logs spring-music
cf logs spring-music --recent

cf를 이용한 ssh 터널링
cf ssh welcome-cf -L 9999:10.10.4.14:3306