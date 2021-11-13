# 기록 & 정리

```
helm search repo
helm install test owkin/hlf-ca
```

```
 helm install test owkin/hlf-ca --namespace test -f values.yaml
 helm delete test --namespace test
```



```
Run the following commands to...
1. Get the name of the pod running the Fabric CA Server:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=hlf-ca,release=test" -o jsonpath="{.items[0].metadata.name}")

2. Get the application URL:
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl port-forward --namespace default $POD_NAME 8080:7054

3. Display local (admin "client" enrollment) certificate, if it has been created:
  kubectl exec --namespace default $POD_NAME -- cat /var/hyperledger/fabric-ca/msp/signcerts/cert.pem

4. Enroll the bootstrap admin identity:
  kubectl exec --namespace default $POD_NAME -- bash -c 'fabric-ca-client enroll -d -u http://$CA_ADMIN:$CA_PASSWORD@$SERVICE_DNS:7054'

5. Update the chart without resetting a password:
  export CA_ADMIN=$(kubectl get secret --namespace default test-hlf-ca--ca -o jsonpath="{.data.CA_ADMIN}" | base64 --decode; echo)
  export CA_PASSWORD=$(kubectl get secret --namespace default test-hlf-ca--ca -o jsonpath="{.data.CA_PASSWORD}" | base64 --decode; echo)
  helm upgrade test stable/hlf-ca --namespace default -f my-values.yaml --set adminUsername=$CA_ADMIN,adminPassword=$CA_PASSWORD

```

```
kubectl logs -n test ${POD_NAME} | grep "Listening on"
```

```
hlf-blockchain-org1
org1-hlf-ca
```

````
 mysql -u root -p
````

```
kubectl exec --namespace ${NAMESPACE} org1-hlf-ca-5cff8f67c9-bvxwz -- sh -c 'fabric-ca-client identity list'
```

```
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c 'fabric-ca-client identity list'
```

```
HLF_ORG="org1"
_UUID="452b2679-11b9-4144-b8cc-bc22c0fc9cf4"
NAMESPACE="hlf-blockchain-${HLF_ORG}"
ORG_NAME="hlf--${HLF_ORG}"

CA_RELEASE="${HLF_ORG}-hlf-ca"
CA_PATH="/data/hlf/${NAMESPACE}/${CA_RELEASE}"
ORD_RELEASE="${HLF_ORG}-hlf-ord"
PEER_RELEASE="${HLF_ORG}-hlf-peer"

CA_POD_NAME=$(kubectl get pods --namespace ${NAMESPACE}-${_UUID} -l "app=hlf-ca,release=${CA_RELEASE}" -o jsonpath="{.items[0].metadata.name}")
CA_ADMIN=$(kubectl get secret --namespace ${NAMESPACE}-${_UUID} ${CA_RELEASE}--ca -o jsonpath="{.data.CA_ADMIN}" | base64 --decode; echo)
CA_PASSWORD=$(kubectl get secret --namespace ${NAMESPACE}-${_UUID} ${CA_RELEASE}--ca -o jsonpath="{.data.CA_PASSWORD}" | base64 --decode; echo)
```



---



```
#!/bin/bash

RED=`tput setaf 1`
GREEN=`tput setaf 2`
NC=`tput sgr0`

#apt install jq -y


read -r -p "${GREEN}Input Your Organization Name (example. org1, org2) : ${NC}" HLF_ORG

NAMESPACE="hlf-blockchain-${HLF_ORG}"
ORG_NAME="hlf--${HLF_ORG}"

CA_RELEASE="${HLF_ORG}-hlf-ca"
CA_PATH="/data/hlf/${NAMESPACE}/${CA_RELEASE}"
ORD_RELEASE="${HLF_ORG}-hlf-ord"
PEER_RELEASE="${HLF_ORG}-hlf-peer"


# # all
# _hostname="cluster-1"
# kubectl taint nodes --all node-role.kubernetes.io/master-
# kubectl get configmaps -n kube-system kubeadm-config -o yaml | sed "s/    clusterName: kubernetes/    clusterName: ${_hostname}/g" | kubectl replace -f - &&
# kubectl config set-context kubernetes-admin@kubernetes --cluster=${_hostname}
# kubectl config set-context kubernetes-admin@kubernetes --user=${_hostname}
# kubectl config rename-context kubernetes-admin@kubernetes ${_hostname}
# sed -i "s/  name: kubernetes/  name: ${_hostname}/g" ~/.kube/config
# sed -i "s/- name: kubernetes-admin/- name: ${_hostname}/g" ~/.kube/config
# kubectl get nodes --show-labels

# kubectl create serviceaccount ${_hostname} -n kube-system
# kubectl create clusterrolebinding ${_hostname} \
#   --clusterrole=cluster-admin \
#   --serviceaccount=kube-system:${_hostname}

CLUSTER_NAME="$(hostname)"
# APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
APISERVER=$(kubectl config view -o jsonpath="{.clusters[?(@.name==\"$CLUSTER_NAME\")].cluster.server}")
# TOKEN=$(kubectl get secrets -n kube-system -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='default')].data.token}"|base64 --decode)
TOKEN=$(kubectl get secrets -n kube-system -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='${CLUSTER_NAME}')].data.token}"|base64 --decode)
NAMESPACE_CHECK=$(curl -s -o /dev/null -w "%{http_code}" -X GET $APISERVER/api/v1/namespaces/${NAMESPACE} --header "Authorization: Bearer $TOKEN" --insecure)

if [[ $NAMESPACE_CHECK == *"404"* ]]; then
    echo "${RED}--namespace not exist--${NC}"
    # -f /data/hlf/${NAMESPACE}/${RELEASE} 
else
    echo "${RED}--namespace exist...--${NC}"
    
    read -r -p "USER EXIST RESET? (name is ${NAMESPACE}) : " input
    case $input in
        [yY][eE][sS]|[yY])
    		    echo "Yes"
        kubectl delete ns ${NAMESPACE} --force
        # kubectl delete pvc ${CA_RELEASE} -n ${NAMESPACE} --force
        kubectl delete pvc --namespace ${NAMESPACE} -l "name=${CA_RELEASE}" --force
        kubectl delete pv --namespace ${NAMESPACE} -l "name=${CA_RELEASE}" --force
        rm -rf /data/hlf/${NAMESPACE}
        echo "${GREEN} uninstall complete ${NC}"
        exit 1
		    ;;
        [nN][oO]|[nN])
		    echo "No"
       		    ;;
        *)
	    echo "Invalid input..."
	    exit 1
	    ;;
    esac
fi



cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
  namespace: ${NAMESPACE}
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
EOF

# kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml



# first - Peer Organization 1
# Hyperledger Fabric CA
echo "${GREEN} helm HLF-CA install ${NC}"

helm repo add owkin https://owkin.github.io/charts
helm repo update

helm install ${CA_RELEASE} owkin/hlf-ca --version 2.0.1 \
  --create-namespace \
  --namespace ${NAMESPACE} \
  --set image.repository="hyperledger/fabric-ca" \
  --set image.tag="1.5.2" \
  --set config.hlfToolsVersion="1.5.2" \
  --set caName=${CA_RELEASE} \
  --set adminUsername=ca-admin,adminPassword=innogrid \
  --set config.csr.names.c=KR \
  --set config.csr.names.st=Daejeon \
  --set config.csr.names.o=Etri \
  --set config.csr.names.ou=Blockchain \
  --set persistence.enabled=false \
  --set persistence.storageClass="local-storage"
#   --set config.mountTLS=true

CA_POD_NAME=$(kubectl get pods --namespace ${NAMESPACE} -l "app=hlf-ca,release=${CA_RELEASE}" -o jsonpath="{.items[0].metadata.name}")
CA_ADMIN=$(kubectl get secret --namespace ${NAMESPACE} ${CA_RELEASE}--ca -o jsonpath="{.data.CA_ADMIN}" | base64 --decode; echo)
CA_PASSWORD=$(kubectl get secret --namespace ${NAMESPACE} ${CA_RELEASE}--ca -o jsonpath="{.data.CA_PASSWORD}" | base64 --decode; echo)

kubectl logs -n ${NAMESPACE} ${CA_POD_NAME} | grep "Listening on"

echo -e "${GREEN} helm installed ${NC} \n"

echo -e "\n ${GREEN} Data Folder creating... ${NC}"
mkdir -p ${CA_PATH}
ls -al ${CA_PATH}
echo -e "${GREEN} Data Folder created ${NC} \n"

_UUID="$(uuidgen)"
echo "${GREEN} PersistentVolume creating... ${NC}"
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ${CA_RELEASE}-${_UUID}
  namespace: ${NAMESPACE}
  labels:
    name: ${CA_RELEASE}
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 5Gi
  claimRef:
    name: ${CA_RELEASE}
    namespace: ${NAMESPACE}
  hostPath:
    path: /data/hlf/${NAMESPACE}/${CA_RELEASE}
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  volumeMode: Filesystem
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: ${CA_RELEASE}-${_UUID}
  namespace: ${NAMESPACE}
  labels:
    name: ${CA_RELEASE}
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
EOF
echo -e "${GREEN} PersistentVolume created ${NC} \n"


while true ; do
    echo "${GREEN} HLF-CA Preparing... ${NC}"
    CA_RUNNING_CHECK=$(curl -s -X GET $APISERVER/api/v1/namespaces/${NAMESPACE}/pods/${CA_POD_NAME} --header "Authorization: Bearer $TOKEN" --insecure | jq '.status.phase')
    CA_PV_CHECK=$(curl -s -X GET $APISERVER/api/v1/persistentvolumes/${CA_RELEASE}-${_UUID} --header "Authorization: Bearer $TOKEN" --insecure | jq '.status.phase')
    CA_PVC_CHECK=$(curl -s -X GET $APISERVER/api/v1/namespaces/${NAMESPACE}/persistentvolumeclaims/${CA_RELEASE}-${_UUID} --header "Authorization: Bearer $TOKEN" --insecure | jq '.status.phase')
    echo " - CA_POD Status phase is : ${CA_RUNNING_CHECK}"
    echo " - CA_PV Status phase is : ${CA_PV_CHECK}"
    echo " - CA_PVC Status phase is : ${CA_PVC_CHECK}"
    if [[ $CA_RUNNING_CHECK == *"Running"* ]]; then
        echo -e "${GREEN} HLF-CA Installed Got it... ${NC} \n"
        break
    fi
    sleep 5s
done

SERVICE_DNS="0.0.0.0"

# 인증서 정보 가입을 위한 권한 취득
echo -e "\n${GREEN} ${CA_RELEASE} CA- admin Certificate Creating... ${NC}\n"
# kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c 'fabric-ca-client enroll -d -u http://$CA_ADMIN:$CA_PASSWORD@$SERVICE_DNS:7054 -M /var/hyperledger/fabric-ca/msp/ca-admincerts'
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c 'fabric-ca-client enroll -d -u http://$CA_ADMIN:$CA_PASSWORD@$SERVICE_DNS:7054'
echo -e "\n${GREEN} ${CA_RELEASE} admin Certificate Created... ${NC}\n"

echo "${GREEN} admincerts exporting... ${NC}"
kubectl cp ${NAMESPACE}/${CA_POD_NAME}:/var/hyperledger/fabric-ca/msp ${CA_PATH}/ordererOrganizations/orgorderer.com/msp
if [ -d ${CA_PATH}/ordererOrganizations/orgorderer.com/msp ]; then
    ls -al ${CA_PATH}/ordererOrganizations/orgorderer.com/msp
    echo -e "${GREEN} ca-admincerts export ok. ${NC} \n"

    mkdir -p ${CA_PATH}/ca-certs
    ORG_CERT=$(cat ${CA_PATH}/ordererOrganizations/orgorderer.com/msp/signcerts/cert.pem)
    ORG_KEY=$(cat ${CA_PATH}/ordererOrganizations/orgorderer.com/msp/keystore/*_sk)
    CA_CERT=$(cat ${CA_PATH}/ordererOrganizations/orgorderer.com/msp/cacerts/*.pem)
    kubectl create secret generic -n ${NAMESPACE} ${ORG_NAME}-admincert --from-literal=cert.pem="$ORG_CERT"
    kubectl create secret generic -n ${NAMESPACE} ${ORG_NAME}-adminkey --from-literal=key.pem="$ORG_KEY"
    kubectl create secret generic -n ${NAMESPACE} ${ORG_NAME}-ca-cert --from-literal=cacert.pem="$CA_CERT"
else 
    echo -e "${GREEN} ca-admincerts export failed. ${NC} \n"
fi


# # 인증서 정보 가입을 위한 권한 취득
# echo -e "\n${GREEN} ${CA_RELEASE} CA TLS Certificate Creating... ${NC}\n"
# # kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- bash -c 'fabric-ca-client enroll -d --enrollment.profile tls -u http://$CA_ADMIN:$CA_PASSWORD@$SERVICE_DNS:7054 -M /var/hyperledger/fabric-ca/msp/tlscerts --csr.hosts peer0.example.com''
# kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c 'fabric-ca-client enroll -d --enrollment.profile tls -u http://$CA_ADMIN:$CA_PASSWORD@$SERVICE_DNS:7054 -M /var/hyperledger/fabric-ca/msp/ca-tlscerts'
# echo -e "\n${GREEN} ${CA_RELEASE} CA TLS Certificate Created... ${NC}\n"


# echo "${GREEN} tlscerts exporting... ${NC}"
# kubectl cp ${NAMESPACE}/$CA_POD_NAME:/var/hyperledger/fabric-ca/msp/ca-tlscerts ${CA_PATH}/ca-tlscerts
# if [ -d ${CA_PATH}/ca-tlscerts ]; then
#     ls -al ${CA_PATH}/ca-tlscerts
#     echo -e "${GREEN} ca-tlscerts export ok. ${NC} \n"
#     # mkdir -p ${CA_PATH}/tlscerts/complete/
#     cp ${CA_PATH}/ca-tlscerts/signcerts/* ${CA_PATH}/ca-tlscerts/tls.crt
#     cp ${CA_PATH}/ca-tlscerts/keystore/* ${CA_PATH}/ca-tlscerts/tls.key
# else 
#     echo -e "${GREEN} tlscerts export failed. ${NC} \n"
# fi


# Orderer Organisation
ORD_ADMIN="ord-admin"
ORD_PASSWORD="OrdAdm1nPW"
kubectl exec -n ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client register --id.name ${ORD_ADMIN} --id.secret ${ORD_PASSWORD} --id.attrs 'admin=true:ecert'"


echo "\n${GREEN} ${CA_RELEASE} orderer MSP CA Certificate Creating... ${NC}\n"
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client getcacert -d -u http://${ORD_ADMIN}:${ORD_PASSWORD}@$SERVICE_DNS:7054 -M /tmp/orgs/orderer/msp"

echo "\n${GREEN} ${CA_RELEASE} orderer admin msp certificate MSP Certificate Creating... ${NC}\n"
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client enroll -d -u http://${ORD_ADMIN}:${ORD_PASSWORD}@$SERVICE_DNS:7054 -M /tmp/orgs/orderer/admin/msp"

ORDERER0_NAME="orderer0.orgorderer.com"
ORDERER0_PASSWORD="orderer0_pw"
ORDERER1_NAME="orderer1.orgorderer.com"
ORDERER1_PASSWORD="orderer1_pw"
ORDERER2_NAME="orderer2.orgorderer.com"
ORDERER2_PASSWORD="orderer2_pw"

echo "${GREEN} Orderer 인증서 정보 가입... ${NC}"
kubectl exec -n ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client register -d --id.name ${ORDERER0_NAME} --id.secret ${ORDERER0_PASSWORD} --id.type orderer"
kubectl exec -n ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client register -d --id.name ${ORDERER1_NAME} --id.secret ${ORDERER1_PASSWORD} --id.type orderer"
kubectl exec -n ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client register -d --id.name ${ORDERER2_NAME} --id.secret ${ORDERER2_PASSWORD} --id.type orderer"


echo "\n${GREEN} ${CA_RELEASE} Orderer ord0, ord1, ord2, ord3 msp certificate MSP Certificate Creating... ${NC}\n"
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client enroll -d -u http://${ORDERER0_NAME}:${ORDERER0_PASSWORD}@$SERVICE_DNS:7054 -M /tmp/orgs/orderer/orderer0/msp"
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client enroll -d -u http://${ORDERER1_NAME}:${ORDERER1_PASSWORD}@$SERVICE_DNS:7054 -M /tmp/orgs/orderer/orderer1/msp"
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client enroll -d -u http://${ORDERER2_NAME}:${ORDERER2_PASSWORD}@$SERVICE_DNS:7054 -M /tmp/orgs/orderer/orderer2/msp"


echo "\n${GREEN} ${CA_RELEASE} Orderer TLS Certificate Creating... ${NC}\n"
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client enroll -d --enrollment.profile tls -u http://${ORDERER0_NAME}:${ORDERER0_PASSWORD}@$SERVICE_DNS:7054 -M /tmp/orgs/orderer/orderer0/tls --csr.hosts ${ORDERER0_NAME}"
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client enroll -d --enrollment.profile tls -u http://${ORDERER1_NAME}:${ORDERER1_PASSWORD}@$SERVICE_DNS:7054 -M /tmp/orgs/orderer/orderer1/tls --csr.hosts ${ORDERER1_NAME}"
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client enroll -d --enrollment.profile tls -u http://${ORDERER2_NAME}:${ORDERER2_PASSWORD}@$SERVICE_DNS:7054 -M /tmp/orgs/orderer/orderer2/tls --csr.hosts ${ORDERER2_NAME}"


echo "${GREEN} Orderer admincerts exporting... ${NC}"
# kubectl cp ${NAMESPACE}/${CA_POD_NAME}:/var/hyperledger/fabric-ca/msp/ord-admincerts ${CA_PATH}/ord-admincerts

echo "${GREEN} Orderer0... ${NC}"
mkdir -p ${CA_PATH}/ordererOrganizations/orgorderer.com/orderer/${ORDERER0_NAME}/tls
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "cat /tmp/orgs/orderer/orderer0/tls/signcerts/*" > ${CA_PATH}/ordererOrganizations/orgorderer.com/orderer/${ORDERER0_NAME}/tls/server.crt
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "cat /tmp/orgs/orderer/orderer0/tls/keystore/*" > ${CA_PATH}/ordererOrganizations/orgorderer.com/orderer/${ORDERER0_NAME}/tls/server.key
echo "${GREEN} Orderer1... ${NC}"
mkdir -p ${CA_PATH}/ordererOrganizations/orgorderer.com/orderer/${ORDERER1_NAME}/tls
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "cat /tmp/orgs/orderer/orderer1/tls/signcerts/*" > ${CA_PATH}/ordererOrganizations/orgorderer.com/orderer/${ORDERER1_NAME}/tls/server.crt
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "cat /tmp/orgs/orderer/orderer1/tls/keystore/*" > ${CA_PATH}/ordererOrganizations/orgorderer.com/orderer/${ORDERER1_NAME}/tls/server.key
echo "${GREEN} Orderer2... ${NC}"
mkdir -p ${CA_PATH}/ordererOrganizations/orgorderer.com/orderer/${ORDERER2_NAME}/tls
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "cat /tmp/orgs/orderer/orderer2/tls/signcerts/*" > ${CA_PATH}/ordererOrganizations/orgorderer.com/orderer/${ORDERER2_NAME}/tls/server.crt
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "cat /tmp/orgs/orderer/orderer2/tls/keystore/*" > ${CA_PATH}/ordererOrganizations/orgorderer.com/orderer/${ORDERER2_NAME}/tls/server.key

echo "${GREEN} Orderer Organizations MSP Certificate... ${NC}"
kubectl cp ${NAMESPACE}/${CA_POD_NAME}:/tmp/orgs/orderer/orderer0/msp ${CA_PATH}/ordererOrganizations/orgorderer.com/msp

echo "${GREEN} Orderer Orderer0-3 MSP Certificate... ${NC}"
echo "${GREEN} Orderer0... ${NC}"
kubectl cp ${NAMESPACE}/${CA_POD_NAME}:/tmp/orgs/orderer/orderer0/msp ${CA_PATH}/ordererOrganizations/orgorderer.com/orderer/${ORDERER0_NAME}/msp
echo "${GREEN} Orderer1... ${NC}"
kubectl cp ${NAMESPACE}/${CA_POD_NAME}:/tmp/orgs/orderer/orderer1/msp ${CA_PATH}/ordererOrganizations/orgorderer.com/orderer/${ORDERER1_NAME}/msp
echo "${GREEN} Orderer2... ${NC}"
kubectl cp ${NAMESPACE}/${CA_POD_NAME}:/tmp/orgs/orderer/orderer2/msp ${CA_PATH}/ordererOrganizations/orgorderer.com/orderer/${ORDERER2_NAME}/msp

echo "${GREEN} Orderer admin msp Certificate... ${NC}"
mkdir -p ${CA_PATH}/ordererOrganizations/orgorderer.com/users/Admin@orgorderer.com
kubectl cp ${NAMESPACE}/${CA_POD_NAME}:/tmp/orgs/orderer/admin/msp ${CA_PATH}/ordererOrganizations/orgorderer.com/users/Admin@orgorderer.com/msp


# if [ -d ${CA_PATH}/ord-admincerts ]; then
#     ls -al ${CA_PATH}/ord-admincerts
#     echo -e "${GREEN} ord-admincerts export ok. ${NC} \n"
#     # ORG_CERT=$(ls ${CA_PATH}/ord-admincerts/signcerts/cert.pem)
#     # ORG_KEY=$(ls ${CA_PATH}/ord-admincerts/keystore/*_sk)
#     # CA_CERT=$(ls ${CA_PATH}/ord-admincerts/cacerts/*.pem)
#     ORG_CERT=$(ls ${CA_PATH}/ord-admincerts/signcerts/cert.pem)
#     ORG_KEY=$(ls ${CA_PATH}/ord-admincerts/keystore/*_sk)
#     CA_CERT=$(ls ${CA_PATH}/ord-admincerts/cacerts/*.pem)
#     kubectl create secret generic -n ${NAMESPACE} ${ORG_NAME}--ord-admincert --from-file=cert.pem="$ORG_CERT"
#     kubectl create secret generic -n ${NAMESPACE} ${ORG_NAME}--ord-adminkey --from-file=key.pem="$ORG_KEY"
#     kubectl create secret generic -n ${NAMESPACE} ${ORG_NAME}--ord-ca-cert --from-file=cacert.pem="$CA_CERT"    
# else 
#     echo -e "${GREEN} ord-admincerts export failed. ${NC} \n"
# fi

# # Peer Organisation
PEER_ADMIN="apeer-admin"
PEER_PASSWORD="PeerAdm1nPW"
echo "${GREEN} aPeer admin 피어 인증서 정보 가입... ${NC}"
kubectl exec -n ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client register --id.name ${PEER_ADMIN} --id.secret ${PEER_PASSWORD} --id.attrs 'hf.Registrar.Roles=client,hf.Registrar.Attributes=*,hf.Revoker=true,hf.GenCRL=true,admin=true:ecert,abac.init=true:ecert'"

echo "\n${GREEN} ${CA_RELEASE} aPeer MSP CA Certificate Creating... ${NC}\n"
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client getcacert -d -u http://${PEER_ADMIN}:${PEER_PASSWORD}@$SERVICE_DNS:7054 -M /tmp/orgs/apeer/msp"

echo "\n${GREEN} ${CA_RELEASE} aPeer admin msp certificate MSP Certificate Creating... ${NC}\n"
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client enroll -d -u http://${PEER_ADMIN}:${PEER_PASSWORD}@$SERVICE_DNS:7054 -M /tmp/orgs/apeer/admin/msp"

PEER0_NAME="peer0.orgapeer.com"
PEER0_PASSWORD="peer0_apeerpw"
PEER1_NAME="peer1.orgapeer.com"
PEER1_PASSWORD="peer1_apeerpw"

echo "${GREEN} aPeer 피어 인증서 정보 가입... ${NC}"
kubectl exec -n ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client register -d --id.name ${PEER0_NAME} --id.secret ${PEER0_PASSWORD} --id.type peer"
kubectl exec -n ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client register -d --id.name ${PEER1_NAME} --id.secret ${PEER1_PASSWORD} --id.type peer"

echo "\n${GREEN} ${CA_RELEASE} aPeer peer0, peer1 / admin msp certificate MSP Certificate Creating... ${NC}\n"
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client enroll -d -u http://${PEER0_NAME}:${PEER0_PASSWORD}@$SERVICE_DNS:7054 -M /tmp/orgs/apeer/peer0/msp"
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client enroll -d -u http://${PEER1_NAME}:${PEER1_PASSWORD}@$SERVICE_DNS:7054 -M /tmp/orgs/apeer/peer1/msp"


echo "\n${GREEN} ${CA_RELEASE} aPeer TLS Certificate Creating... ${NC}\n"
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client enroll -d --enrollment.profile tls -u http://${PEER0_NAME}:${PEER0_PASSWORD}@$SERVICE_DNS:7054 -M /tmp/orgs/apeer/peer0/tls --csr.hosts ${PEER0_NAME}"
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "fabric-ca-client enroll -d --enrollment.profile tls -u http://${PEER1_NAME}:${PEER1_PASSWORD}@$SERVICE_DNS:7054 -M /tmp/orgs/apeer/peer1/tls --csr.hosts ${PEER1_NAME}"


echo "${GREEN} Peer admincerts exporting... ${NC}"
# kubectl cp ${NAMESPACE}/${CA_POD_NAME}:/var/hyperledger/fabric-ca/msp/peer-admincerts ${CA_PATH}/peer-admincerts

echo "${GREEN} Peer0... ${NC}"
mkdir -p ${CA_PATH}/peerOrganizations/orgapeer.com/peers/${PEER0_NAME}/tls
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "cat /tmp/orgs/apeer/peer0/tls/signcerts/*" > ${CA_PATH}/peerOrganizations/orgapeer.com/peers/${PEER0_NAME}/tls/server.crt
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "cat /tmp/orgs/apeer/peer0/tls/keystore/*" > ${CA_PATH}/peerOrganizations/orgapeer.com/peers/${PEER0_NAME}/tls/server.key
echo "${GREEN} Peer1... ${NC}"
mkdir -p ${CA_PATH}/peerOrganizations/orgapeer.com/peers/${PEER1_NAME}/tls
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "cat /tmp/orgs/apeer/peer1/tls/signcerts/*" > ${CA_PATH}/peerOrganizations/orgapeer.com/peers/${PEER1_NAME}/tls/server.crt
kubectl exec --namespace ${NAMESPACE} ${CA_POD_NAME} -- sh -c "cat /tmp/orgs/apeer/peer1/tls/keystore/*" > ${CA_PATH}/peerOrganizations/orgapeer.com/peers/${PEER1_NAME}/tls/server.key

echo "${GREEN} apper Organizations MSP Certificate... ${NC}"
kubectl cp ${NAMESPACE}/${CA_POD_NAME}:/tmp/orgs/apeer/msp ${CA_PATH}/peerOrganizations/orgapeer.com/msp

echo "${GREEN} apper Peer 0-1 MSP Certicate... ${NC}"
echo "${GREEN} Peer0... ${NC}"
kubectl cp ${NAMESPACE}/${CA_POD_NAME}:/tmp/orgs/apeer/peer0/msp ${CA_PATH}/peerOrganizations/orgapeer.com/peers/${PEER0_NAME}/msp
echo "${GREEN} Peer1... ${NC}"
kubectl cp ${NAMESPACE}/${CA_POD_NAME}:/tmp/orgs/apeer/peer1/msp ${CA_PATH}/peerOrganizations/orgapeer.com/peers/${PEER1_NAME}/msp


echo "${GREEN} apper peer admin msp Certificate... ${NC}"
mkdir -p ${CA_PATH}/peerOrganizations/orgapeer.com/users/Admin@orgapeer.com
kubectl cp ${NAMESPACE}/${CA_POD_NAME}:/tmp/orgs/apeer/admin/msp ${CA_PATH}/peerOrganizations/orgapeer.com/users/Admin@orgapeer.com/msp



# if [ -d ${CA_PATH}/peer-admincerts ]; then
#     ls -al ${CA_PATH}/peer-admincerts
#     echo -e "${GREEN} peer-admincerts export ok. ${NC} \n"
#     ORG_CERT=$(ls ${CA_PATH}/peer-admincerts/signcerts/cert.pem)
#     ORG_KEY=$(ls ${CA_PATH}/peer-admincerts/keystore/*_sk)
#     CA_CERT=$(ls ${CA_PATH}/peer-admincerts/cacerts/*.pem)
#     kubectl create secret generic -n ${NAMESPACE} ${ORG_NAME}--peer-admincert --from-file=cert.pem=$ORG_CERT
#     kubectl create secret generic -n ${NAMESPACE} ${ORG_NAME}--peer-adminkey --from-file=key.pem=$ORG_KEY
#     kubectl create secret generic -n ${NAMESPACE} ${ORG_NAME}--peer-ca-cert --from-file=cacert.pem=$CA_CERT    
# else 
#     echo -e "${GREEN} peer-admincerts export failed. ${NC} \n"
# fi


########################################

# # second - Hyperledger Fabric Peer
# echo "${GREEN} helm HLF-PEER install ${NC}"

# MSP_ID="${HLF_ORG}-MSP"

# helm install ${CA_RELEASE} owkin/hlf-peer \
#   --create-namespace \
#   --namespace ${NAMESPACE} \
#   --peer.mspID=${MSP_ID} \
#   --set persistence.storageClass="local-storage" \
#   --set peer.databaseType="CouchDB" \
#   --set peer.couchdbSecret="cdb1-hlf-couchdb"


# CA_POD_NAME=$(kubectl get pods --namespace ${NAMESPACE} -l "app=hlf-ca,release=${CA_RELEASE}" -o jsonpath="{.items[0].metadata.name}")
# CA_ADMIN=$(kubectl get secret --namespace ${NAMESPACE} ${CA_RELEASE}--ca -o jsonpath="{.data.CA_ADMIN}" | base64 --decode; echo)
# CA_PASSWORD=$(kubectl get secret --namespace ${NAMESPACE} ${CA_RELEASE}--ca -o jsonpath="{.data.CA_PASSWORD}" | base64 --decode; echo)

# echo -e "${GREEN} helm installed ${NC} \n"

# echo -e "\n ${GREEN} Data Folder creating... ${NC}"
# mkdir -p ${CA_PATH}
# ls -al ${CA_PATH}
# echo -e "${GREEN} Data Folder created ${NC} \n"

# echo "${GREEN} PersistentVolume creating... ${NC}"
# cat <<EOF | kubectl apply -f -
# apiVersion: v1
# kind: PersistentVolume
# metadata:
#   name: ${CA_RELEASE}
#   namespace: ${NAMESPACE}
# spec:
#   accessModes:
#   - ReadWriteOnce
#   capacity:
#     storage: 5Gi
#   claimRef:
#     name: ${CA_RELEASE}
#     namespace: ${NAMESPACE}
#   hostPath:
#     path: /data/hlf/${NAMESPACE}/${CA_RELEASE}
#   persistentVolumeReclaimPolicy: Delete
#   storageClassName: local-storage
#   volumeMode: Filesystem
# EOF
# echo -e "${GREEN} PersistentVolume created ${NC} \n"

########################################

# third - Hyperledger Fabric Orderer
# echo "${GREEN} helm HLF-Orderer install ${NC}"

# MSP_ID="${HLF_ORG}-MSP"

# helm install ${CA_RELEASE} owkin/hlf-ord \
#   --create-namespace \
#   --namespace ${NAMESPACE} \
#   --set ord.type="solo" \
#   --set ord.mspID=${MSP_ID} \
#   --set caUsername=ord-admin,caPassword=innogrid \
#   --set persistence.storageClass="local-storage"
  


# CA_POD_NAME=$(kubectl get pods --namespace ${NAMESPACE} -l "app=hlf-ca,release=${CA_RELEASE}" -o jsonpath="{.items[0].metadata.name}")
# CA_ADMIN=$(kubectl get secret --namespace ${NAMESPACE} ${CA_RELEASE}--ca -o jsonpath="{.data.CA_ADMIN}" | base64 --decode; echo)
# CA_PASSWORD=$(kubectl get secret --namespace ${NAMESPACE} ${CA_RELEASE}--ca -o jsonpath="{.data.CA_PASSWORD}" | base64 --decode; echo)

# echo -e "${GREEN} helm installed ${NC} \n"

# echo -e "\n ${GREEN} Data Folder creating... ${NC}"
# mkdir -p ${CA_PATH}
# ls -al ${CA_PATH}
# echo -e "${GREEN} Data Folder created ${NC} \n"

# echo "${GREEN} PersistentVolume creating... ${NC}"
# cat <<EOF | kubectl apply -f -
# apiVersion: v1
# kind: PersistentVolume
# metadata:
#   name: ${CA_RELEASE}
#   namespace: ${NAMESPACE}
# spec:
#   accessModes:
#   - ReadWriteOnce
#   capacity:
#     storage: 5Gi
#   claimRef:
#     name: ${CA_RELEASE}
#     namespace: ${NAMESPACE}
#   hostPath:
#     path: /data/hlf/${NAMESPACE}/${CA_RELEASE}
#   persistentVolumeReclaimPolicy: Delete
#   storageClassName: local-storage
#   volumeMode: Filesystem
# EOF
# echo -e "${GREEN} PersistentVolume created ${NC} \n"
```

