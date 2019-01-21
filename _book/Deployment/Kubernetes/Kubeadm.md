# Install

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

VERSION=1.13.1-0
yum install -y kubelet-$VERSION kubeadm-$VERSION kubectl-$VERSION

systemctl start  kubelet
systemctl enable kubelet
```

# Requirements

初级运维即可完成

# Setting

#### 配置文件

```bash
# kubeadm.conf
mkdir -p /etc/kubernetes/
cat << EOF > /etc/kubernetes/kubeadm.conf
---
apiVersion: kubeadm.k8s.io/v1alpha3
kind: InitConfiguration
apiEndpoint:
  advertiseAddress: 172.11.51.200 # 这里替换内网的高可用IP地址

---
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta1
clusterName: kubernetes-prod # 这里替换可成自定义集群名
kubernetesVersion: v1.13.1
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
controlPlaneEndpoint: 172.11.51.200:6443
dns:
  type: CoreDNS
etcd:
  external:
    endpoints:
    - http://172.11.51.201:18000 # 这里替换可成自定义ETCD节点信息
    - http://172.11.51.202:18000 # 这里替换可成自定义ETCD节点信息
    - http://172.11.51.203:18000 # 这里替换可成自定义ETCD节点信息
networking:
  dnsDomain: idc.naturali.io # 这里替换可成自定义KuberenetesCluster内部的DnsDomain信息
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
apiServer:
  extraArgs:
    bind-address: 172.11.51.201 # 如果Master节点上有多个IP地址 可以指定
  certSANs:
  - 172.11.51.200 # 这里依次填写Master、VIP、hostname、域名等信息
  - 172.11.51.201 # 用于kubu-api等服务在生成证书时使用
  - 172.11.51.202
  - 172.11.51.203
  - api.k8s.fds.so
  - k8s-master01.idc
  - k8s-master02.idc
  - k8s-master03.idc
EOF
```

#### 开始安装

```bash
#在其中一台服务器上安装
kubeadm init --config /etc/kubernetes/kubeadm.conf

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl taint nodes --all node-role.kubernetes.io/master-

#安装flannel
mkdir ~/kubernetes; cd ~/kubernetes
curl -sSL https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml | sed 's/vxlan/host-gw/' > kube-flannel.yml
kubectl apply -f kube-flannel.yml
```

