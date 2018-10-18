a


# 手工安装K8S v1.11.2多master集群



## 环境配置

#### 开启IPVS：

- 加载**IPVS**模块,如果重新开机，需要重新加载

``` modprobe ip_vs
modprobe ip_vs

modprobe ip_vs_rr

modprobe ip_vs_wrr

modprobe ip_vs_sh

modprobe nf_conntrack_ipv4
```

- 开启之后，确认生效

``` 
lsmod | grep ip_vs

ip_vs_sh              12688  0 
ip_vs_wrr              12697  0 
ip_vs_rr              12600  0 
ip_vs                141092  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
nf_conntrack          133387  7 ip_vs,nf_nat,nf_nat_ipv4,nf_nat_ipv6,xt_conntrack,nf_conntrack_ipv4,nf_conntrack_ipv6
libcrc32c              12644  4 xfs,ip_vs,nf_nat,nf_conntrack
```

#### 配置hosts(分别在每台机器上设置hostname)

```
hostnamectl set-hostname master01

hostnamectl set-hostname master02

hostnamectl set-hostname master03
```

#### 关闭selinux

```
setenforce 0

sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/sysconfig/selinux 

sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config 

sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/sysconfig/selinux 

sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/selinux/config
```

#### 关闭swap

```
swapoff -a

sed -i 's/.swap./#&/' /etc/fstab
```

#### 开启forward

- Docker从1.13版本开始调整了默认的防火墙规则，禁用了iptables filter表中FOWARD链，这样会引起Kubernetes集群中跨Node的Pod无法通信。

```
iptables -P FORWARD ACCEPT

systemctl stop firewalld

systemctl disable firewalld
```

#### 配置转发参数

```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness=0
EOF

sysctl --system                #生效配置
```

#### 配置hosts解析

- 172.16.5.90是设置的VIP虚拟IP

```
cat >>/etc/hosts<<EOF
172.16.5.70 master01
172.16.5.71 master02
172.16.5.72 master03
172.16.5.90 vip-k8s
EOF
```

#### 重启（重启之后要重新配置ipvs），可选

```
reboot
```

#### 配置互信(SSH)

- 在所有的机器上生成公钥，一路回车即可

```
[root@master01 ~]# ssh-keygen -t rsa 

[root@master02 ~]# ssh-keygen -t rsa

[root@master03 ~]# ssh-keygen -t rsa 

```

- 相互拷贝公钥(可以考虑one-for-all或者all-for-all，根据自己需求定)

```
ssh-copy-id -i  .ssh/id_rsa.pub root@master01
ssh-copy-id -i  .ssh/id_rsa.pub root@master02
ssh-copy-id -i  .ssh/id_rsa.pub root@master03
```

## 安装配置docker/k8s

#### 安装docker

- 卸载原有**docker**（根绝自己的情况而定）

```
yum remove docker docker-common docker-selinux docker-engine
```

- 安装**docker 17.03 **(11推荐docker 1.17)

```
yum install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm  -y

yum install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm  -y
```

- **docker** 配置阿里云加速器(修改**Execstart**启动选项)

``` 
vi /usr/lib/systemd/system/docker.service

ExecStart=/usr/bin/dockerd  -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock  --registry-mirror=https://ms3cfraz.mirror.aliyuncs.com
```

- 重新加载并启动**docker**

```
systemctl enable docker && systemctl restart docker
```

#### 安装kubelet

- 单独的一台下载即可，我这里选择是**master01**

```
export KUBE_URL=https://storage.googleapis.com/kubernetes-release/release/v1.11.2/bin/linux/amd64

wget ${KUBE_URL}/kubelet -O /usr/local/bin/kubelet
```

- 拷贝**kubelet**到其他的**nodes**节点

```
scp /usr/local/bin/kubelet root@master02:/usr/local/bin                                                    
scp /usr/local/bin/kubelet root@master03:/usr/local/bin
```

- 赋予可执行权限（所有的**master**节点）

```
chmod +x /usr/local/bin/kubelet
```

#### 安装kubectl

- 下载**kubectl**

```
wget ${KUBE_URL}/kubectl -O /usr/local/bin/kubectl
```

- 复制**kubectl**到其他的**nodes**

```
scp /usr/local/bin/kubectl root@master02:/usr/local/bin

scp /usr/local/bin/kubectl root@master03:/usr/local/bin
```

- 赋予可执行权限

```
chmod +x /usr/local/bin/kubectl
```

#### 安装CNI(所有的master)

- 下载**CNI**

```
export CNI_URL=https://github.com/containernetworking/plugins/releases/download

mkdir -p /opt/cni/bin && cd /opt/cni/bin

wget ${CNI_URL}/v0.7.1/cni-plugins-amd64-v0.7.1.tgz
```

- 解压

```
tar -zxvf cni-plugins-amd64-v0.7.1.tgz
```

#### 安装cfssl工具

- 下载**cfssl**

```
export CFSSL_URL=https://pkg.cfssl.org/R1.2

wget ${CFSSL_URL}/cfssl_linux-amd64 -O /usr/local/bin/cfssl
```

- 下载**cfssljson**

```
wget ${CFSSL_URL}/cfssljson_linux-amd64 -O /usr/local/bin/cfssljson
```

- 赋予执行权限

```
chmod +x /usr/local/bin/cfssl /usr/local/bin/cfssljson
```

## 下载K8S手工配置文件

```
git clone https://github.com/kairen/k8s-manual-files.git 
```

## 配置CA证书

#### etcd证书配置

- 配置环境

```
cd ~/k8s-manual-files/pki

export DIR=/etc/etcd/ssl

mkdir -p ${DIR}
```

- 生成**CA**证书和凭证(IP为master的物理IP)

```
cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare ${DIR}/etcd-ca

cfssl gencert \
  -ca=${DIR}/etcd-ca.pem \
  -ca-key=${DIR}/etcd-ca-key.pem \
  -config=ca-config.json \
  -hostname=127.0.0.1,172.16.5.70,172.16.5.71,172.16.5.72 \
  -profile=kubernetes \
  etcd-csr.json | cfssljson -bare ${DIR}/etcd
```

- 删除没有必要的文件(可选)

```
rm -rf ${DIR}/*.csr

ls /etc/etcd/ssl
etcd-ca-key.pem  etcd-ca.pem  etcd-key.pem  etcd.pem
```

- 复制证书到其他的节点

```
for NODE in master01 master02 master03; do
    echo "--- $NODE ---"
    ssh ${NODE} " mkdir -p /etc/etcd/ssl"
    for FILE in etcd-ca-key.pem  etcd-ca.pem  etcd-key.pem  etcd.pem; do
      scp /etc/etcd/ssl/${FILE} ${NODE}:/etc/etcd/ssl/${FILE}
    done
  done
```

#### API证书配置

- 环境配置(**APISERVER**设置为**vip** )

```
export K8S_DIR=/etc/kubernetes

export PKI_DIR=${K8S_DIR}/pki

export KUBE_APISERVER=https://172.16.5.90:6443  

mkdir -p ${PKI_DIR}
```

- 创建CA和API的凭证

```
cfssl gencert -initca ca-csr.json | cfssljson -bare ${PKI_DIR}/ca

ls ${PKI_DIR}/ca*.pem
/etc/kubernetes/pki/ca-key.pem  /etc/kubernetes/pki/ca.pem

cfssl gencert \
  -ca=${PKI_DIR}/ca.pem \
  -ca-key=${PKI_DIR}/ca-key.pem \
  -config=ca-config.json \
  -
 hostname=10.96.0.1,172.16.5.90,127.0.0.1,kubernetes.default \
  -profile=kubernetes \
  apiserver-csr.json | cfssljson -bare ${PKI_DIR}/apiserver
  
ls ${PKI_DIR}/apiserver*.pem
/etc/kubernetes/pki/apiserver-key.pem  /etc/kubernetes/pki/apiserver.pem
```

#### Front Proxy Client证书配置

```
cfssl gencert -initca front-proxy-ca-csr.json | cfssljson -bare ${PKI_DIR}/front-proxy-ca

ls ${PKI_DIR}/front-proxy-ca*.pem
/etc/kubernetes/pki/front-proxy-ca-key.pem  /etc/kubernetes/pki/front-proxy-ca.pem

cfssl gencert \
-ca=${PKI_DIR}/front-proxy-ca.pem \
-ca-key=${PKI_DIR}/front-proxy-ca-key.pem \
-config=ca-config.json \
-profile=kubernetes \

front-proxy-client-csr.json | cfssljson -bare ${PKI_DIR}/front-proxy-client

ls ${PKI_DIR}/front-proxy-client*.pem
/etc/kubernetes/pki/front-proxy-client-key.pem  /etc/kubernetes/pki/front-proxy-client.pem
```

#### controller证书配置

```
cfssl gencert \
  -ca=${PKI_DIR}/ca.pem \
  -ca-key=${PKI_DIR}/ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  manager-csr.json | cfssljson -bare ${PKI_DIR}/controller-manager

ls ${PKI_DIR}/controller-manager*.pem
/etc/kubernetes/pki/controller-manager-key.pem  /etc/kubernetes/pki/controller-manager.pem
```

#### 生成Controller Manager的kubeconfig

- set-cluster

```
kubectl config set-cluster kubernetes \
    --certificate-authority=${PKI_DIR}/ca.pem \
    --embed-certs=true \
    --server=${KUBE_APISERVER} \
    --kubeconfig=${K8S_DIR}/controller-manager.conf
```

- set-credentials 

```
kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=${PKI_DIR}/controller-manager.pem \
    --client-key=${PKI_DIR}/controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=${K8S_DIR}/controller-manager.conf
```

- set-context

```
kubectl config set-context system:kube-controller-manager@kubernetes \
    --cluster=kubernetes \
    --user=system:kube-controller-manager \
    --kubeconfig=${K8S_DIR}/controller-manager.conf
```

- use-context

```
kubectl config use-context system:kube-controller-manager@kubernetes \
    --kubeconfig=${K8S_DIR}/controller-manager.conf
```

#### scheduler证书配置

```
cfssl gencert \
  -ca=${PKI_DIR}/ca.pem \
  -ca-key=${PKI_DIR}/ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  scheduler-csr.json | cfssljson -bare ${PKI_DIR}/scheduler

ls ${PKI_DIR}/scheduler*.pem
/etc/kubernetes/pki/scheduler-key.pem  /etc/kubernetes/pki/scheduler.pem
```

#### 生成Scheduler的kubeconfig

- set-cluster

```
kubectl config set-cluster kubernetes \
    --certificate-authority=${PKI_DIR}/ca.pem \
    --embed-certs=true \
    --server=${KUBE_APISERVER} \
    --kubeconfig=${K8S_DIR}/scheduler.conf
```

- set-credentials

```
kubectl config set-credentials system:kube-scheduler \
    --client-certificate=${PKI_DIR}/scheduler.pem \
    --client-key=${PKI_DIR}/scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=${K8S_DIR}/scheduler.conf
```

- set-context 

```
kubectl config set-context system:kube-scheduler@kubernetes \
    --cluster=kubernetes \
    --user=system:kube-scheduler \
    --kubeconfig=${K8S_DIR}/scheduler.conf
```

- use-context 

```
kubectl config use-context system:kube-scheduler@kubernetes \
    --kubeconfig=${K8S_DIR}/scheduler.conf
```

#### admin证书配置

```
cfssl gencert \
  -ca=${PKI_DIR}/ca.pem \
  -ca-key=${PKI_DIR}/ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare ${PKI_DIR}/admin
```

#### 生成Admin的kubeconfig文件

- set-cluster 

```
kubectl config set-cluster kubernetes \
    --certificate-authority=${PKI_DIR}/ca.pem \
    --embed-certs=true \
    --server=${KUBE_APISERVER} \
    --kubeconfig=${K8S_DIR}/admin.conf
```

- set-credentials

```
kubectl config set-credentials kubernetes-admin \
    --client-certificate=${PKI_DIR}/admin.pem \
    --client-key=${PKI_DIR}/admin-key.pem \
    --embed-certs=true \
    --kubeconfig=${K8S_DIR}/admin.conf
```

- set-context

```
kubectl config set-context kubernetes-admin@kubernetes \
    --cluster=kubernetes \
    --user=kubernetes-admin \
    --kubeconfig=${K8S_DIR}/admin.conf
```

- use-context

```
kubectl config use-context kubernetes-admin@kubernetes \
    --kubeconfig=${K8S_DIR}/admin.conf
```

#### 生成所有节点的Kubelet的CA认证

```
for NODE in master01 master02 master03; do
    echo "--- $NODE ---"
    cp kubelet-csr.json kubelet-$NODE-csr.json;
    sed -i "s/$NODE/$NODE/g" kubelet-$NODE-csr.json;
    cfssl gencert \
      -ca=${PKI_DIR}/ca.pem \
      -ca-key=${PKI_DIR}/ca-key.pem \
      -config=ca-config.json \
      -hostname=$NODE \
      -profile=kubernetes \
      kubelet-$NODE-csr.json | cfssljson -bare ${PKI_DIR}/kubelet-$NODE;
    rm kubelet-$NODE-csr.json
  done
```

#### 将kubelet生成的CA证书复制到所有的节点

```
for NODE in master01 master02 master03; do
    echo "--- $NODE ---"
    ssh ${NODE} "mkdir -p ${PKI_DIR}"
    scp ${PKI_DIR}/ca.pem ${NODE}:${PKI_DIR}/ca.pem
    scp ${PKI_DIR}/kubelet-$NODE-key.pem ${NODE}:${PKI_DIR}/kubelet-key.pem
    scp ${PKI_DIR}/kubelet-$NODE.pem ${NODE}:${PKI_DIR}/kubelet.pem
    rm ${PKI_DIR}/kubelet-$NODE-key.pem ${PKI_DIR}/kubelet-$NODE.pem
  done
```

#### 所有的节点生成config文件

```
for NODE in master01 master02 master03; do
    echo "--- $NODE ---"
    ssh ${NODE} "cd ${PKI_DIR} && \
      kubectl config set-cluster kubernetes \
        --certificate-authority=${PKI_DIR}/ca.pem \
        --embed-certs=true \
        --server=${KUBE_APISERVER} \
        --kubeconfig=${K8S_DIR}/kubelet.conf && \
      kubectl config set-credentials system:node:${NODE} \
        --client-certificate=${PKI_DIR}/kubelet.pem \
        --client-key=${PKI_DIR}/kubelet-key.pem \
        --embed-certs=true \
        --kubeconfig=${K8S_DIR}/kubelet.conf && \
      kubectl config set-context system:node:${NODE}@kubernetes \
        --cluster=kubernetes \
        --user=system:node:${NODE} \
        --kubeconfig=${K8S_DIR}/kubelet.conf && \
      kubectl config use-context system:node:${NODE}@kubernetes \
        --kubeconfig=${K8S_DIR}/kubelet.conf"
  done
```

#### Service Account证书配置(使用ssl认证，当然也可以使用cfssl)

```
openssl genrsa -out ${PKI_DIR}/sa.key 2048

openssl rsa -in ${PKI_DIR}/sa.key -pubout -out ${PKI_DIR}/sa.pub
```

#### 删除多余文件（可选）

```
rm -rf ${PKI_DIR}/*.csr \
    ${PKI_DIR}/scheduler*.pem \
    ${PKI_DIR}/controller-manager*.pem \
    ${PKI_DIR}/admin*.pem \
    ${PKI_DIR}/kubelet*.pem
```

#### 凭证将复制到其他master节点

```
for NODE in master01 master02 master03; do
    echo "--- $NODE ---"
    for FILE in $(ls ${PKI_DIR}); do
      scp ${PKI_DIR}/${FILE} ${NODE}:${PKI_DIR}/${FILE}
    done
  done
```

#### 复制kubeconfig文件至其他master节点

```
for NODE in master01 master02 master03; do
    echo "--- $NODE ---"
    for FILE in admin.conf controller-manager.conf scheduler.conf; do
      scp ${K8S_DIR}/${FILE} ${NODE}:${K8S_DIR}/${FILE}
    done
  done
```

## 拉取镜像（所有的master）

```
docker pull docker.io/osixia/keepalived:1.4.5

docker pull docker.io/haproxy:1.7-alpine

docker pull quay.io/coreos/etcd:v3.3.8

docker pull quay.io/calico/typha:v0.7.4

docker pull quay.io/calico/node:v3.1.3

docker pull quay.io/calico/cni:v3.1.3

docker pull quay.io/calico/ctl:v3.1.3

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver-amd64:v1.11.2

docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver-amd64:v1.11.2 k8s.gcr.io/kube-apiserver-amd64:v1.11.2

docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver-amd64:v1.11.2

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager-amd64:v1.11.2

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler-amd64:v1.11.2

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1

docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler-amd64:v1.11.2 k8s.gcr.io/kube-scheduler-amd64:v1.11.2

docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler-amd64:v1.11.2

docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager-amd64:v1.11.2 k8s.gcr.io/kube-controller-manager-amd64:v1.11.2

docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager-amd64:v1.11.2

docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1 k8s.gcr.io/pause:3.1

docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy-amd64:v1.11.2

docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy-amd64:v1.11.2 k8s.gcr.io/kube-proxy-amd64:v1.11.2

docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy-amd64:v1.11.2

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.1.3

docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.1.3 k8s.gcr.io/coredns:1.1.3

docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.1.3
```

## 配置k8s

#### 生成配置文件

- 部署与设定

```
cd ~/k8s-manual-files

export NODES="master01 master02 master03"

# 执行前请修改脚本(修改为master节点的hostname)
: ${NODES:="master01 master02 master03"}

替换成etcd流量转换的网卡（IP地址修改为你选择走流量的网卡）
ip route get 172.16.5.0|awk '{print $NF; exit}' 
```

- 利用./hack/gen-configs.sh脚本在每台master节点产生组态文件

```
./hack/gen-configs.sh
```

- 生成etcd和proxy的config文件

```
# 检查IP信息是否正常

more /etc/etcd/config.yml

initial-advertise-peer-urls: 'https://172.16.5.70:2380'
advertise-client-urls: 'https://172.16.5.70:2379'
discovery:
discovery-fallback: 'proxy'
discovery-proxy:
discovery-srv:
# example: hostname=https://ip:2380, ...,
initial-cluster: 'master01=https://172.16.5.70:2380,master02=https://172.16.5.71:2380,master03=https://172.16.5.72:2380'
initial-cluster-token: 'k8s-etcd-cluster'
```

```
more /etc/haproxy/haproxy.cfg
# 检查IP信息是否正常
frontend kube-apiserver-https
  mode tcp
  bind :6443
  default_backend kube-apiserver-backend
backend kube-apiserver-backend
    mode tcp
    server master01-api 172.16.5.70:5443 check
    server master02-api 172.16.5.71:5443 check
    server master03-api 172.16.5.72:5443 check
```

- 利用./hack/gen-manifests.sh生成k8s的yml文件

```
export NODES="master01 master02 master03"

# 执行前请修改脚本
# 修改为所有master节点的hostname
: ${NODES:="master01 master02 master03"}      
# ADVERTISE_VIP修改为自己的VIP
: ${ADVERTISE_VIP:="172.16.5.90"} 
# 获取走流量网卡ip
HOST_START=$(ip route get 172.16.5.0 | awk '{print $NF; exit}')    
# 获取所有master节点的ip
IP=$(ssh ${NODE} "ip route get 172.16.5.0" | awk '{print $NF; exit}') 
# 获取NIC（走流量的网卡名）
NIC=$(ssh ${NODE} "ip route get 172.16.5.0" | awk '{print $4; exit}') 

# 根据自己的需求修改manifests下yml文件里所需要k8s版本号
cd /root/k8s-manual-files/master/manifests

more kube-apiserver.yml 
image: k8s.gcr.io/kube-apiserver-amd64:v1.11.2

# 运行脚本生成配置文件
 ./hack/gen-manifests.sh
 
# 检查以下目录配置文件是否正确
/etc/kubernetes/manifests
/etc/kubernetes/encryption
/etc/kubernetes/audit

# 修改api端口为8080
vi /etc/kubernetes/manifests/kube-apiserver.yml 
–insecure-port=8080

# 默认配置为0，会无法启动服务，报错The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

- 拷贝kubelet配置文件到其他的节点

```
for NODE in master01 master02 master03; do
    echo "--- $NODE ---"
    ssh ${NODE} "mkdir -p /var/lib/kubelet /var/log/kubernetes /var/lib/etcd /etc/systemd/system/kubelet.service.d"
    scp master/var/lib/kubelet/config.yml ${NODE}:/var/lib/kubelet/config.yml
    scp master/systemd/kubelet.service ${NODE}:/lib/systemd/system/kubelet.service
    scp master/systemd/10-kubelet.conf ${NODE}:/etc/systemd/system/kubelet.service.d/10-kubelet.conf
  done
```

- 启动kubelet

```
for NODE in master01 master02 master03; do
    ssh ${NODE} "systemctl enable kubelet.service && systemctl start kubelet.service"
  done
```

#### 建立 TLS Bootstrapping(在其中一台操作master01)

```
export TOKEN_ID=$(openssl rand 3 -hex)

export TOKEN_SECRET=$(openssl rand 8 -hex)

export BOOTSTRAP_TOKEN=${TOKEN_ID}.${TOKEN_SECRET}

export KUBE_APISERVER="https://172.16.5.90:6443"
```

- set-cluster

```
kubectl config set-cluster kubernetes \
    --certificate-authority=/etc/kubernetes/pki/ca.pem \
    --embed-certs=true \
    --server=${KUBE_APISERVER} \
    --kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf
```

- set-credentials

```
kubectl config set-credentials tls-bootstrap-token-user \
    --token=${BOOTSTRAP_TOKEN} \
    --kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf
```

- set set-context

```
kubectl config set-context tls-bootstrap-token-user@kubernetes \
    --cluster=kubernetes \
    --user=tls-bootstrap-token-user \
    --kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf
```

- use-context

```
kubectl config use-context tls-bootstrap-token-user@kubernetes \
    --kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf
```

- 建立token

```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-${TOKEN_ID}
  namespace: kube-system
type: bootstrap.kubernetes.io/token
stringData:
  token-id: "${TOKEN_ID}"
  token-secret: "${TOKEN_SECRET}"
  usage-bootstrap-authentication: "true"
  usage-bootstrap-signing: "true"
  auth-extra-groups: system:bootstrappers:default-node-token
EOF

secret/bootstrap-token-33f792 created
```

- 创建TLS Bootstrap Autoapprove RBAC来提供自动受理CSR

```
kubectl apply -f master/resources/kubelet-bootstrap-rbac.yml

clusterrolebinding.rbac.authorization.k8s.io/kubelet-bootstrap created
clusterrolebinding.rbac.authorization.k8s.io/node-autoapprove-bootstrap created
clusterrolebinding.rbac.authorization.k8s.io/node-autoapprove-certificate-rotation created
```

#### 验证Master节点

- 复制Admin kubeconfig文件(所有的master)

```
cp /etc/kubernetes/admin.conf ~/.kube/config
```

- etcd查看状态

```
kubectl get cs

NAME                STATUS    MESSAGE            ERROR
scheduler            Healthy  ok                  
controller-manager  Healthy  ok                  
etcd-2              Healthy  {"health":"true"}  
etcd-0              Healthy  {"health":"true"}  
etcd-1              Healthy  {"health":"true"}  
```

- 查看pods状态

```
kubectl -n kube-system get po -o wide

NAME                              READY    STATUS    RESTARTS  AGE      IP            NODE      NOMINATED NODE
etcd-master01                      1/1      Running  0          30m      172.16.5.70  master01  <none>
etcd-master02                      1/1      Running  0          30m      172.16.5.71  master02  <none>
etcd-master03                      1/1      Running  0          30m      172.16.5.72  master03  <none>
kube-apiserver-master01            1/1      Running  0          7m        172.16.5.70  master01  <none>
kube-apiserver-master02            1/1      Running  0          7m        172.16.5.71  master02  <none>
kube-apiserver-master03            1/1      Running  0          6m        172.16.5.72  master03  <none>
kube-controller-manager-master01  1/1      Running  0          30m      172.16.5.70  master01  <none>
kube-controller-manager-master02  1/1      Running  0          30m      172.16.5.71  master02  <none>
kube-controller-manager-master03  1/1      Running  0          30m      172.16.5.72  master03  <none>
kube-haproxy-master01              1/1      Running  0          30m      172.16.5.70  master01  <none>
kube-haproxy-master02              1/1      Running  0          30m      172.16.5.71  master02  <none>
kube-haproxy-master03              1/1      Running  0          30m      172.16.5.72  master03  <none>
kube-keepalived-master01          1/1      Running  0          30m      172.16.5.70  master01  <none>
kube-keepalived-master02          1/1      Running  0          30m      172.16.5.71  master02  <none>
kube-keepalived-master03          1/1      Running  0          30m      172.16.5.72  master03  <none>
kube-scheduler-master01            1/1      Running  0          30m      172.16.5.70  master01  <none>
kube-scheduler-master02            1/1      Running  0          30m      172.16.5.71  master02  <none>
kube-scheduler-master03            1/1      Running  0          30m      172.16.5.72  master03  <none>
```

- 查看masters

```
- kubectl get node

NAME      STATUS    ROLES    AGE      VERSION
master01  NotReady  master    31m      v1.11.2
master02  NotReady  master    31m      v1.11.2
master03  NotReady  master    31m      v1.11.2
```

- 查看API的log

```
kubectl -n kube-system logs -f kube-apiserver-master01 

Error from server (Forbidden): Forbidden (user=kube-apiserver, verb=get, resource=nodes, subresource=proxy) ( pods/log kube-apiserver-master01)

# 这边会发现出现 403 Forbidden 问题，这是因为 kube-apiserver user 并没有nodes的资源访问权限，属于正常。
```

- RBAC Role 来获取访问权限(单节点执行)

```
# 查看文件
more master/resources/apiserver-to-kubelet-rbac.yml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kube-apiserver
    
# 创建服务
kubectl apply -f master/resources/apiserver-to-kubelet-rbac.yml

clusterrole.rbac.authorization.k8s.io/system:kube-apiserver-to-kubelet created
clusterrolebinding.rbac.authorization.k8s.io/system:kube-apiserver created
```

- 再次访问即可

####  设定 Taints and Tolerations

```
$ kubectl taint nodes node-role.kubernetes.io/master="":NoSchedule --all

node "k8s-m1" tainted
node "k8s-m2" tainted
node "k8s-m3" tainted

# 让一些特定 Pod 能够排程到所有master节点上
```

## Core Addons部署

- 当完成master与node节点的部署，并组合成一个可运作集群后，就可以开始通过 kubectl 部署 Addons，Kubernetes 官方提供了多种 Addons 来加强 Kubernetes 的各种功能，如集群 DNS 解析的kube-dns(or CoreDNS)、外部存取服务的kube-proxy与 Web-based 管理接口的dashboard等等。而其中有些 Addons 是被 Kubernetes 认定为必要的，因此本节将说明如何部署这些 Addons。

#### Kubernetes Proxy

- kube-proxy 是实现 Kubernetes Service 资源功能的关键组件，这个组件会通过 DaemonSet 在每台节点上执行，然后监听 API Server 的 Service 与 Endpoint 资源对象的事件，并依据资源预期状态通过 iptables 或 ipvs 来实现网络转发，这里采用 ipvs（ipvs比起iptables效率更高，更加稳定）
- 部署proxy

```
$ cd ~/k8s-manual-files

export KUBE_APISERVER=https://172.16.5.90:6443

#修改proxy文件（设置对应的版本和ip）

sed -i 's/${KUBE_APISERVER}/${KUBE_APISERVER}/g' addons/kube-proxy/kube-proxy-cm.yml

more addons/kube-proxy/kube-proxy-cm.yml
server: https://172.16.5.90:6443
more addons/kube-proxy/kube-proxy.yml 
image: k8s.gcr.io/kube-proxy-amd64:v1.11.2
```

- 启动proxy pods

```
kubectl create -f addons/kube-proxy/

kubectl -n kube-system get po -l k8s-app=kube-proxy

NAME              READY    STATUS    RESTARTS  AGE
kube-proxy-6p86n  1/1      Running  0          26s
kube-proxy-csr2b  1/1      Running  0          26s
kube-proxy-r6kdn  1/1      Running  0          26s
```

- 检查proxy是否使用 ipvs

```
kubectl -n kube-system logs -f kube-proxy-6p86n

I0914 09:00:02.133589      1 server_others.go:183] Using ipvs Proxier.
I0914 09:00:02.170594      1 server_others.go:210] Tearing down inactive rules.
```

- 安装ipvsadm查看ipvs的规则

```
yum install ipvsadm

ipvsadm -ln

IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port          Forward Weight ActiveConn InActConn
TCP  10.96.0.1:443 rr
  -> 172.16.5.90:5443            Masq    1      0          0 
```

#### CoreDNS

- 通过 CoreDNS 取代 Kube DNS 作为集群服务发现组件，由于 Kubernetes 需要让 Pod 与 Pod 之间能够互相沟通，然而要能够沟通需要知道彼此的 IP 才行，而这种做法通常是通过 Kubernetes API 来取得达到，但是 Pod IP 会因为生命周期变化而改变，因此这种做法无法弹性使用，且还会增加 API Server 负担，基于此问题 Kubernetes 提供了 DNS 服务来作为查询，让 Pod 能够以 Service 名称作为域名来查询 IP 地址，因此用户就再不需要关切实际 Pod IP，而 DNS 也会根据 Pod 变化更新资源纪录(Record resources)。
- 创建coreDNS服务

```
kubectl create -f addons/coredns/

kubectl -n kube-system get po -l k8s-app=kube-dns

NAME                      READY    STATUS    RESTARTS  AGE
coredns-859d584858-swpnh  0/1      Pending  0          11s
coredns-859d584858-zhwjf  0/1      Pending  0          11s

#这边会发现 Pod 处于Pending状态，这是由于 Kubernetes 的集群网络没有建立，因此所有节点会处于NotReady状态,而这也导致 Kubernetes Scheduler 无法为 Pod 找到适合节点，因此coredns处于Pending状态，若 Pod 是被 DaemonSet 管理的话，则不会 Pending。
```

#### 配置Kubernetes集群网路

- Kubernetes 在默认情况下与 Docker 的网络有所不同。在 Kubernetes 中有四个问题是需要被解决的，分别为：

1. 高耦合的容器到容器沟通：通过 Pods 与 Localhost 的沟通来解决。
2. Pod 到 Pod 的沟通：通过实现网络模型来解决。
3. Pod 到 Service 沟通：由 Services object 结合 kube-proxy 解决。
4. 外部到 Service 沟通：一样由 Services object 结合 kube-proxy 解决。

- Kubernetes 对于任何网络的实现都需要满足以下基本要求

1. 所有容器能够在没有 NAT 的情况下与其他容器沟通。
2. 所有节点能够在没有 NAT 情况下与所有容器沟通(反之亦然)。
3. 容器看到的 IP 与其他人看到的 IP 是一样的。

- Kubernetes 中的网络插件有以下两种形式：

1. CNI plugins：以 appc/CNI 标准规范所实现的网络，详细可以阅读 CNI Specification。
2. Kubenet plugin：使用 CNI plugins 的 bridge 与 host-local 来实现基本的 cbr0。这通常被用在公有云服务上的 Kubernetes 集群网络。

#### 网路部署与设定

- 选择了 Calico 作为集群网络的使用。Calico 是一款纯 Layer 3 的网络，其好处是它整合了各种云原生平台(Docker、Mesos 与 OpenStack 等)，且 Calico 不采用 vSwitch，而是在每个 Kubernetes 节点使用 vRouter 功能，并通过 Linux Kernel 既有的 L3 forwarding 功能，而当数据中心复杂度增加时，Calico 也可以利用 BGP route reflector 来达成。
- Calico 提供了 Kubernetes resources YAML 文件来快速以容器方式部署网络插件至所有节点上

```
cd ~/k8s-manual-files

# 将CALICO_IPV4POOL_CIDR的网络修改 Cluster IP CIDR
sed -i 's/192.168.0.0\/16/10.244.0.0\/16/g' cni/calico/v3.1/calico.yml

kubectl create -f cni/calico/v3.1/

# 当节点过多产生压力，可以使用 Calico 的 Typha 模式来减少通过 Kubernetes datastore 造成 API Server 的负担。Typha模式具体请查看Calico官方文档
```

- 检查是否有启动

```
kubectl -n kube-system get po -l k8s-app=calico-node

NAME                READY    STATUS    RESTARTS  AGE
calico-node-dwtcn  2/2      Running    0          2m
calico-node-lmb4d  0/2      OutOfcpu  0          2m
calico-node-pkrnh  2/2      Running    0          2m

# 这里是因为虚拟机的CPU资源不足，超出了request的总和。由于本身硬件资源问题，这里暂时搁置
```

- 检查calicoctl pod 功能是否正常

```
kubectl exec -ti -n kube-system calicoctl -- calicoctl get profiles -o wide
NAME              LABELS
kns.default      map[]
kns.kube-public  map[]
kns.kube-system  map[]

kubectl exec -ti -n kube-system calicoctl -- calicoctl get node -o wide

NAME    ASN        IPV4              IPV6
master01  (unknown)  172.22.132.10/24
master02  (unknown)  172.22.132.11/24
master03  (unknown)  172.22.132.12/24

# 若没问题，就可以将kube-system下的calicoctl pod删除。
```

- 检查节点是否不再是NotReady

```
kubectl get no

NAME      STATUS    ROLES    AGE      VERSION
master01    Ready    master    35m      v1.11.2
master02    Ready    master    35m      v1.11.2
master03    Ready    master    35m      v1.11.2

```

- 检查Pod是否不再处于Pending

```
kubectl -n kube-system get po -l k8s-app=kube-dns -o wide

NAME                      READY    STATUS    RESTARTS  AGE      IP          NODE
coredns-589dd74cb6-5mv5c  1/1      Running  0          10m      10.244.4.2  mster02
coredns-589dd74cb6-d42ft  1/1      Running  0          10m      10.244.3.2  master01
```

## Everything is done！