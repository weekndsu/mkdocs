#  HA 故障master01节点重置恢复

#### 环境配置

- 开启IPVS
- 配置hostname
- 关闭selinux
- 关闭swap
- 开启forward
- 配置转发参数
- 配置hosts解析

#### 移除故障的etcd

```
ETCD=`docker ps|grep etcd|grep -v POD|awk '{print $1}'`

docker exec \
  -it ${ETCD} \
  etcdctl \
  --endpoints https://127.0.0.1:2379 \
  --ca-file /etc/kubernetes/pki/etcd/ca.crt \
  --cert-file /etc/kubernetes/pki/etcd/server.crt \
  --key-file /etc/kubernetes/pki/etcd/server.key \
  member remove 19c5f5e4748dc98b
```

#### 文件安装

- 安装docker

```
yum remove docker docker-common docker-selinux docker-engine

yum install 
https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm  -y

yum install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm  -y

vi /usr/lib/systemd/system/docker.service

ExecStart=/usr/bin/dockerd  -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock  --registry-mirror=https://ms3cfraz.mirror.aliyuncs.com

systemctl enable docker && systemctl restart docker
```

- 处理kubelet和kubectl

```
scp /usr/local/bin/kubelet root@master01:/usr/local/bin 

scp /usr/local/bin/kubectl root@master01:/usr/local/bin

chmod +x /usr/local/bin/kubelet
chmod +x /usr/local/bin/kubectl
```

- 处理CNI

```
export CNI_URL=https://github.com/containernetworking/plugins/releases/download

mkdir -p /opt/cni/bin && cd /opt/cni/bin

wget ${CNI_URL}/v0.7.1/cni-plugins-amd64-v0.7.1.tgz

tar -zxvf cni-plugins-amd64-v0.7.1.tgz
```

- 拷贝CA文件和凭证

```
for NODE in master01; do
    echo "--- $NODE ---"
    ssh ${NODE} " mkdir -p /etc/etcd/ssl"
    for FILE in etcd-ca-key.pem  etcd-ca.pem  etcd-key.pem  etcd.pem; do
      scp /etc/etcd/ssl/${FILE} ${NODE}:/etc/etcd/ssl/${FILE}
    done
  done

export K8S_DIR=/etc/kubernetes
export PKI_DIR=${K8S_DIR}/pki
export KUBE_APISERVER=https://172.16.5.90:6443  


for NODE in master01; do
    echo "--- $NODE ---"
    ssh ${NODE} "mkdir -p ${PKI_DIR}"
    scp ${PKI_DIR}/ca.pem ${NODE}:${PKI_DIR}/ca.pem
    scp ${PKI_DIR}/kubelet-$NODE-key.pem ${NODE}:${PKI_DIR}/kubelet-key.pem
    scp ${PKI_DIR}/kubelet-$NODE.pem ${NODE}:${PKI_DIR}/kubelet.pem
    rm ${PKI_DIR}/kubelet-$NODE-key.pem ${PKI_DIR}/kubelet-$NODE.pem
  done
  
for NODE in master01; do
    echo "--- $NODE ---"
    for FILE in $(ls ${PKI_DIR}); do
      scp ${PKI_DIR}/${FILE} ${NODE}:${PKI_DIR}/${FILE}
    done
  done
  
for NODE in master01; do
    echo "--- $NODE ---"
    for FILE in admin.conf controller-manager.conf scheduler.conf; do
      scp ${K8S_DIR}/${FILE} ${NODE}:${K8S_DIR}/${FILE}
    done
  done 
```

#### 拉取镜像

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

- 拷贝kubelet配置文件

```
for NODE in master01; do
    echo "--- $NODE ---"
    ssh ${NODE} "mkdir -p /var/lib/kubelet /var/log/kubernetes /var/lib/etcd /etc/systemd/system/kubelet.service.d"
    scp master/var/lib/kubelet/config.yml ${NODE}:/var/lib/kubelet/config.yml
    scp master/systemd/kubelet.service ${NODE}:/lib/systemd/system/kubelet.service
    scp master/systemd/10-kubelet.conf ${NODE}:/etc/systemd/system/kubelet.service.d/10-kubelet.conf
  done
```

- etcd新成员加入集群

```
ETCD=`docker ps|grep etcd|grep -v POD|awk '{print $1}'`
hostname=master01
ip=172.16.5.70
 
docker exec \
 -it ${ETCD} \
 etcdctl \
 --ca-file /etc/kubernetes/pki/etcd/ca.crt \
 --cert-file /etc/kubernetes/pki/etcd/peer.crt \
 --key-file /etc/kubernetes/pki/etcd/peer.key \
 --endpoints=https://127.0.0.1:2379 \
 member add $host https://$ip:2380
```

- 启动kubelet

```
for NODE in master01; do
    ssh ${NODE} "systemctl enable kubelet.service && systemctl start kubelet.service"
  done
```



