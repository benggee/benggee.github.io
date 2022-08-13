# Kubeadm安装k8s集群

## 1. 环境准备

#### 机器配置

- 操作系统：centos7.6
- CPU：2核
- 内存：2G

#### 机器信息

| 节点名   | IP地址        | Hostname |
| -------- | ------------- | -------- |
| Master01 | 192.168.0.220 | m1       |
| Master02 | 192.168.0.221 | m2       |
| Node01   | 192.168.0.222 | n1       |
| Node02   | 192.168.0.223 | n2       |



快捷键  cmmand + shift + i  在iterm中批量运行命令

### 1.1 设置hostname 

```shell
# 192.168.0.220
[root@master01 ~]# hostnamectl set-hostname m1
# 192.168.0.221
[root@master02 ~]# hostnamectl set-hostname m2
# 192.168.0.222
[root@node01 ~]# hostnamectl set-hostname n1
# 192.168.0.223
[root@node02 ~]# hostnamectl set-hostname n2
```



### 1.2 修改host

```shell
# 在4台机器上分别运行
[root@node02 ~]# vi /etc/hosts
# 加入如下内容
192.168.0.220 m1 
192.168.0.221 m2
192.168.0.222 n1
192.168.0.223 n2
```

### 1.3 安装依赖包

```shell
[root@master01 ~] yum update -y
[root@master01 ~] yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp
```

### 1.4 关闭防火墙、swap

```shell
# 关闭防火墙
$ systemctl stop firewalld && systemctl disable firewalld
# 重置iptables
$ iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat && iptables -P FORWARD ACCEPT
# 关闭swap
$ swapoff -a
$ sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab
# 关闭selinux
$ setenforce 0
$ sed -i 's/enforcing/disabled/g' /etc/selinux/config   
# 关闭dnsmasq(否则可能导致docker容器无法解析域名)
$ service dnsmasq stop && systemctl disable dnsmasq
```

### 1.5 系统参数设置

```shell
# 制作配置文件
$ cat > /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
vm.swappiness=0
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
EOF
# 生效文件
$ sysctl -p /etc/sysctl.d/kubernetes.conf
```



### 1.6 安装Docker

```shell
# 配置yum源
$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 清理原有版本
$ yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine \
                  container-selinux

# 查看版本列表
$ yum list docker-ce --showduplicates | sort -r

# 根据kubernetes对docker版本的兼容测试情况，我们选择18.09版本
$ yum install docker-ce-18.09.9 docker-ce-cli-18.09.9 containerd.io

# 开机启动
$ systemctl enable docker

# 设置参数
# 1.查看磁盘挂载
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        98G  2.8G   95G   3% /
devtmpfs         63G     0   63G   0% /dev
/dev/sda5      1015G  8.8G 1006G   1% /tol
/dev/sda1       197M  161M   37M  82% /boot

# 2.选择比较大的分区（我这里是/tol）如果根分区比较大就不用创建目录了
$ mkdir -p /tol/docker-data
$ cat <<EOF > /etc/docker/daemon.json
{
	"graph": "/tol/docker-data",   /*这个没有创建目录的情况下可以不配*/
	"exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

# 启动docker服务
$ systemctl daemon-reload
$ systemctl restart docker
```



### 1.7 安装工具

```shell
# 配置yum源（科学上网的同学可以把"mirrors.aliyun.com"替换为"packages.cloud.google.com"）
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 安装工具
# 找到要安装的版本号
$ yum list kubeadm --showduplicates | sort -r

# 安装指定版本（这里用的是1.14.0）
$ yum install kubelet-1.14.0-0 -y && yum install kubectl-1.14.0-0 -y && yum install kubeadm-1.14.0-0 -y

# 设置kubelet的cgroupdriver（kubelet的cgroupdriver默认为systemd，如果上面没有设置docker的exec-opts为systemd，这里就需要将kubelet的设置为cgroupfs）
# 由于各自的系统配置不同，配置位置和内容都不相同
# 1. /etc/systemd/system/kubelet.service.d/10-kubeadm.conf(如果此配置存在的情况执行下面命令：)
$ sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

# 2. 如果1中的配置不存在，则此配置应该存在(不需要做任何操)：/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

# 启动kubelet
$ systemctl enable kubelet && systemctl start kubelet
```



## 2. 集群部署

### 2.1 部署keepalived-apiserver高可用（阿里云不支持，可以跳过此步骤）

#### 2.1.1 安装keepalived

```bash
# 在两个主节点上安装keepalived（一主一备）
$ yum install -y keepalived
```

#### 2.1.2 创建keepalived配置文件

```bash
# 创建目录
$ ssh <user>@<master-ip> "mkdir -p /etc/keepalived"
$ ssh <user>@<backup-ip> "mkdir -p /etc/keepalived"

# 分发配置文件
$ scp target/configs/keepalived-master.conf <user>@<master-ip>:/etc/keepalived/keepalived.conf
$ scp target/configs/keepalived-backup.conf <user>@<backup-ip>:/etc/keepalived/keepalived.conf

# 分发监测脚本
$ scp target/scripts/check-apiserver.sh <user>@<master-ip>:/etc/keepalived/
$ scp target/scripts/check-apiserver.sh <user>@<backup-ip>:/etc/keepalived/
```

#### 2.1.3 启动keepalived

```bash
# 分别在master和backup上启动服务
$ systemctl enable keepalived && service keepalived start

# 检查状态
$ service keepalived status

# 查看日志
$ journalctl -f -u keepalived

# 查看虚拟ip
$ ip a
```



### 2.2 部署主节点

```shell
$ kubeadm init --apiserver-advertise-address=172.16.236.80 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.23.4 --service-cidr=10.1.0.0/16 --pod-network-cidr=10.244.0.0/16

# copy kubectl配置（上一步会有提示）
$ mkdir -p ~/.kube
$ cp -i /etc/kubernetes/admin.conf ~/.kube/config

# 测试一下kubectl
$ kubectl get pods --all-namespaces

# **备份init打印的join命令**
```

### 2.3 部署网络插件flannel

我们使用calico官方的安装方式来部署。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

# 参考：https://github.com/flannel-io/flannel
```

### 2.4 加入其它master节点

```shell
# 使用之前保存的join命令加入集群
$ kubeadm join ...

# 耐心等待一会，并观察日志
$ journalctl -f

# 查看集群状态
# 1.查看节点
$ kubectl get nodes
# 2.查看pods
$ kubectl get pods --all-namespaces
```

### 2.5 加入Node节点

```shell
# 使用之前保存的join命令加入集群
$ kubeadm join ...

# 耐心等待一会，并观察日志
$ journalctl -f

# 查看节点
$ kubectl get nodes
```

### 2.6 检查健康

```
$ kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused  
controller-manager   Healthy     ok                                                                                           
etcd-0               Healthy     {"health":"true","reason":""}
```

如果出现上面的情况，解决办法如下：

cd /etc/kubernetes/manifests

vi kube-scheduler.yaml

将 --port=0 这一行删除

systemctl restart kubelet

## 3. 集群测试

3.1 集群可用性测试

创建nginx ds (nginx-ds.yml)

```shell
# 写入配置
$ cd ~
$ cat > nginx-ds.yml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: nginx-ds
  labels:
    app: nginx-ds
spec:
  type: NodePort
  selector:
    app: nginx-ds
  ports:
  - name: http
    port: 80
    targetPort: 80
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: nginx-ds
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  template:
    metadata:
      labels:
        app: nginx-ds
    spec:
      containers:
      - name: my-nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
EOF

# 创建ds
$ kubectl create -f nginx-ds.yml
```

检查IP连通性

```shell
# 检查各 Node 上的 Pod IP 连通性
$ kubectl get pods  -o wide

# 在每个节点上ping pod ip
$ ping <pod-ip>

# 检查service可达性
$ kubectl get svc

# 在每个节点上访问服务
$ curl <service-ip>:<port>

# 在每个节点检查node-port可用性
$ curl <node-ip>:<port>
```

检查dns可用性

```shell
# 创建一个nginx pod
$ cat > pod-nginx.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
EOF

# 创建pod
$ kubectl create -f pod-nginx.yaml

# 进入pod，查看dns
$ kubectl exec  nginx -i -t -- /bin/bash

# 查看dns配置
root@nginx:/# cat /etc/resolv.conf

# 查看名字是否可以正确解析
root@nginx:/# ping nginx-ds
```



## 4. 部署Dashboard

创建pod

```shell
# 上传dashboard配置
$ scp target/addons/dashboard-all.yaml <user>@<node-ip>:/etc/kubernetes/addons/

# 创建服务
$ kubectl apply -f /etc/kubernetes/addons/dashboard-all.yaml

# 查看服务运行情况
$ kubectl get deployment kubernetes-dashboard -n kube-system
$ kubectl --namespace kube-system get pods -o wide
$ kubectl get services kubernetes-dashboard -n kube-system
$ netstat -ntlp|grep 30005
```

服务https证书配置

```
为了集群安全，从 1.7 开始，dashboard 只允许通过 https 访问，我们使用nodeport的方式暴露服务，可以使用 https://NodeIP:NodePort 地址访问
关于自定义证书
默认dashboard的证书是自动生成的，肯定是非安全的证书，如果大家有域名和对应的安全证书可以自己替换掉。使用安全的域名方式访问dashboard。
在dashboard-all.yaml中增加dashboard启动参数，可以指定证书文件，其中证书文件是通过secret注进来的。

- –tls-cert-file
- dashboard.cer
- –tls-key-file
- dashboard.key
```

生成登陆token

Dashboard 默认只支持 token 认证，所以如果使用 KubeConfig 文件，需要在该文件中指定 token，我们这里使用token的方式登录

```bash
# 创建service account
$ kubectl create sa dashboard-admin -n kube-system

# 创建角色绑定关系
$ kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin

# 查看dashboard-admin的secret名字
$ ADMIN_SECRET=$(kubectl get secrets -n kube-system | grep dashboard-admin | awk '{print $1}')

# 打印secret的token
$ kubectl describe secret -n kube-system ${ADMIN_SECRET} | grep -E '^token' | awk '{print $2}'
```

我安装在192.168.1.220机器上，输入以下地址访问Dashboard

https://192.168.1.220:30005/#!/service?namespace=default