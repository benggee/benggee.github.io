# Kubeadm安装k8s集群

## 1. 环境说明

#### 机器配置

- 操作系统：centos7
- CPU：2核
- 内存：4G

#### 机器信息

| 节点名 | IP地址        | Hostname |
| ------ | ------------- | -------- |
| Master | 192.168.0.220 | master   |
| Node01 | 192.168.0.221 | node01   |
| Node02 | 192.168.0.222 | node02   |



## 2. 系统准备

### 2.1. 设置hostname

Master 192.168.0.220 

```shell
> hostnamectl set-hostname master
```

Node01 192.168.0.221

```shell
> hostnamectl set-hostname node01
```

Node02 192.168.0.222

```shell
> hostnamectl set-hostname node02
```



### 2.2. 设置置/etc/hosts

```shell
> vi /etc/hosts
```

```shell
...
192.168.0.220  master
192.168.0.221  node01
192.168.0.222  node02
```



## 3. 安装kubelet、kubeadm、kubectl

初始化环境

```shell
> setenforce 0  
> yum install vim bash-completion net-tools gcc -y
> yum install -y yum-utils device-mapper-persistent-data lvm2
```
或者
```shell
> vi /etc/sysconfig/selinux  
```

设置SELINUX=disabled（永久关闭）

设置阿里去镜像

```shell
> cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

安装并设置开机启动
```shell 
> yum install -y kubelet kubeadm kubectl
> systemctl enable kubelet && systemctl start kubelet
```

关闭交换分区
``` shell 
> swapoff -a   
```
或者永久关闭
``` shell
> sed -ri 's/.*swap.*/#&/' /etc/fstab
```

设置k8s内核参数
``` shell
> cat > /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

## 4. 安装docker

``` shell
# 设置阿里云源
> yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装docker 
> yum -y install docker-ce
# 修改
> mkdir -p /etc/docker
> tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://fl791z1h.mirror.aliyuncs.com"]
}
EOF

> systemctl daemon-reload
> systemctl restart docker
```

初始化kerbunetes集群
``` shell
> kubeadm init --kubernetes-version=1.18.0  \
--apiserver-advertise-address=192.168.0.220   \
--image-repository registry.aliyuncs.com/google_containers  \
--service-cidr=10.10.0.0/16 --pod-network-cidr=10.0.0.0/16
```