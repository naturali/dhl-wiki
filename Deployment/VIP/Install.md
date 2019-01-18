# Install

```bash
BasePath=/data/apps/VIP/
Name=k8s-vip-01
ProjectPath=$BasePath/$Name/

RouterID=100
VRRP=ens33                              # 用于发送VRRP协议的接口
VIP="172.16.190.100/24 dev ens33"       # VIP配置信息

PORT=6443
NODE01=172.11.51.201:6443
NODE02=172.11.51.202:6443
NODE03=172.11.51.203:6443
```

# Requirements

初级运维即可完成

# Setting

#### 配置文件

```bash
cat << EOF > /etc/sysctl.d/81-vip.conf
net.ipv4.ip_nonlocal_bind = 1
EOF

sysctl -p /etc/sysctl.d/81-vip.conf


mkdir -p $ProjectPath; cd $ProjectPath
mkdir -p data/{data,conf,dockerfile}/
```

#### 健康检查脚本

```bash
cat << EOF > data/conf/healthcheck.sh
#!/usr/bin/env bash
curl -sk https://\$1:\$2/ 2>&1 > /dev/null
echo -n \$?
EOF
chmod +x data/conf/healthcheck.sh
```

#### 负载均衡配置

```bash
cat << EOF > data/conf/gobetween
[servers.$Name]
bind     = "172.16.190.100:$PORT" # ip替换成VIP
protocol = "tcp"
balance  = "roundrobin"

max_connections      = 10000
client_idle_timeout  = "60m"
backend_idle_timeout = "60m"
backend_connection_timeout = "2s"

[servers.$Name.discovery]
kind = "static"
static_list = [
    "$NODE01 weight=5",
    "$NODE02 weight=5",
    "$NODE03 weight=5"
]

[servers.$Name.healthcheck] 
kind         = "exec"
interval     = "2s"  
timeout      = "2s"  
exec_command = "/data/conf/healthcheck.sh"
exec_expected_positive_output = "0"
EOF
```

#### 高可用配置

```bash
cat << EOF > data/conf/keepalived.conf
global_defs {
    router_id $Name  
}

vrrp_instance $Name-$RouterID {
    state             BACKUP
    interface         $VRRP
    virtual_router_id $RouterID
    priority          100
    advert_int        1
    authentication {
        auth_type PASS
        auth_pass $Name-PW
    }
    virtual_ipaddress {
        $VIP
    }
}
EOF
```

#### Dockerfile

```bash
cat << EOF > data/dockerfile/Dockerfile
FROM alpine:3.8
ENV  PKGS bash curl keepalived tzdata
RUN  apk add --no-cache \$PKGS && \
     ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
     test -f /usr/local/bin/gobetween || \
     curl -sSL https://github.com/yyyar/gobetween/releases/download/0.6.1/gobetween_0.6.1_linux_amd64.tar.gz | \
     tar xzf - -C /usr/local/bin/ gobetween
EOF
```

#### Docker-compose

```bash
cat << EOF > docker-compose.yaml
version: '2.2'
services:
  gobetween:
    build:
      context: data/dockerfile/
    command: gobetween -c /data/conf/gobetween
    network_mode: "host"
    volumes:
    - ./data/:/data/
    restart: always
  keepalived:
    build:
      context: data/dockerfile/
    command: keepalived -D -n -l
    network_mode: "host"
    volumes:
    - ./data/conf/keepalived.conf:/etc/keepalived/keepalived.conf
    restart: always
    cap_add:
    - ALL
EOF
```

#### 启动

```bash
docker-compose up -d
```



