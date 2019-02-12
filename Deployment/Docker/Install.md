# 安装

```shell
export Version=18.06.1.ce-3.el7

yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y docker-ce-$Version

wget http://mirrors.aliyun.com/docker-toolbox/linux/compose/1.21.2/docker-compose-Linux-x86_64 -O /bin/docker-compose && \
chmod +x /bin/docker-compose
```

# 要求

初级运维即可完成

# 配置

#### 配置文件

```bash
mkdir -p /etc/docker/
cat << EOF > /etc/docker/daemon.json
{
    "graph": "/data/apps/docker/",
    "log-opts": {"max-size": "1024m", "max-file": "10"},
    "live-restore": true,
    "registry-mirrors": ["https://h3vtnoaa.mirror.aliyuncs.com"]
}
EOF

cat << EOF > /usr/lib/sysctl.d/81-docker.conf
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl -p /usr/lib/sysctl.d/81-docker.conf

```

#### 启动

```bash
systemctl start  docker
systemctl enable docker
```



