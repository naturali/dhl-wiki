# 安装

安装最新的7版本即可\(不需要swap分区） 
数据盘位置 /data/apps

# 要求

初级运维即可完成

# 配置

#### 关闭selinux

```shell
setenforce 0
sed -i 's/=enforcing/=disabled/g' /etc/selinux/config
```

#### 关闭service

```bash
ServiceList="firewalld "
for ServiceName in $ServiceList; do
    systemctl stop    $ServiceName
    systemctl disable $ServiceName
done
```

#### 时间同步

```bash
yum install -y ntp ntpdate

ntpdate time.asia.apple.com

cat << EOF > /etc/ntp.conf
logfile    /var/log/ntp.log
pidfile    /var/run/ntpd.pid
driftfile  /var/lib/ntp/drift

restrict    127.0.0.1
restrict    default kod nomodify notrap nopeer noquery
restrict -6 default kod nomodify notrap nopeer noquery

server 127.127.1.0
fudge  127.127.1.0 stratum 10

server   ntp1.aliyun.com iburst minpoll 4 maxpoll 10
restrict ntp1.aliyun.com nomodify notrap nopeer noquery

server   ntp2.aliyun.com iburst minpoll 4 maxpoll 10
restrict ntp2.aliyun.com nomodify notrap nopeer noquery
EOF

systemctl enable  ntpd
systemctl restart ntpd
```

#### sysctl

```bash
cat << EOF > /etc/sysctl.d/80-init.conf
fs.file-max = 6553560

net.ipv4.tcp_fin_timeout     = 15
net.ipv4.ip_local_port_range = 1024 65535
EOF

sysctl -p /etc/sysctl.d/80-init.conf

```

#### ulimit

```bash
cat << EOF > /etc/security/limits.d/80-nofile.conf
*    soft nofile 655350
*    hard nofile 655350
EOF
```

#### 关闭交换分区

```bash
swapoff -a
sed -i 's/^[^#].*swap/#&/' /etc/fstab
```

#### 必要软件

```bash
yum install -y curl wget epel-release
```



