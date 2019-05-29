---
title: kubernetes
date: 2019-05-29 13:58:59
tags: 
- devops
- docker
- kubernetes
---
#  安装docker
1. 安装所需的包。
yum-utils提供了yum-config-manager
> yum install -y yum-utils device-mapper-persistent-data lvm2

2. 设置稳定存储库
> yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
3. 安装DOCKER CE
> yum install docker-ce
4. 启动Docker。
> systemctl start docker
## 卸载docker
卸载Docker包.主机上的Images, containers, volumes或自定义配置文件不会自动删除。要删除所有Images, containers, volumes
> yum remove docker* && rm -rf /var/lib/docker

# 安装kubernetes
1. 安装 flannel etcd
```shell
yum -y install flannel etcd
systemctl start etcd.service
systemctl enable etcd.service
systemctl status etcd.service
```
2. 启动
```shell
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
# 关闭Swap Kubernetes 1.8开始要求关闭系统的Swap，如果不关闭，默认配置下kubelet将无法启动。方法一,通过kubelet的启动参数–fail-swap-on=false更改这个限制。方法二,关闭系统的Swap。
swapoff -a 
sed -i 's/.*swap.*/#&/' /etc/fstab
# 禁用SELinux
setenforce  0 
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/sysconfig/selinux 
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config 
sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/sysconfig/selinux 
sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/selinux/config  
# 配置系统内核参数使流过网桥的流量也进入iptables/netfilter框架中，在/etc/sysctl.conf中添加以下配置：
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl -p /etc/sysctl.d/k8s.conf
```