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
[root@master ~]# hostnamectl set-hostname master
```

Node01 192.168.0.221

```shell
[root@node01 ~]# hostnamectl set-hostname node01
```

Node02 192.168.0.222

```shell
[root@node02 ~]# hostnamectl set-hostname node02
```



### 2.2. 设置置/etc/hosts

```shell
[root@master ~]# vi /etc/hosts
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
[root@master ~]# setenforce 0  
[root@master ~]# yum install vim bash-completion net-tools gcc -y
[root@master ~]# yum install -y yum-utils device-mapper-persistent-data lvm2
```
或者
```shell
[root@master ~]# vi /etc/sysconfig/selinux  
```

设置SELINUX=disabled（永久关闭）

设置阿里去镜像

```shell
[root@master ~]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
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
[root@master ~]# yum install -y kubelet kubeadm kubectl
[root@master ~]# systemctl enable kubelet && systemctl start kubelet
```

关闭交换分区
``` shell 
[root@master ~]# swapoff -a   
```
或者永久关闭
``` shell
[root@master ~]# sed -ri 's/.*swap.*/#&/' /etc/fstab
```

设置k8s内核参数
``` shell
[root@master ~]# cat > /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

## 4. 安装docker

Dcoker的安装可以查看 [CentOS下的Docker安装](../../docker/install.md)


## 5. 初始化kerbunetes集群
``` shell
[root@master ~]# kubeadm init --kubernetes-version=1.18.0  \
                              --apiserver-advertise-address=192.168.0.220   \
                              --image-repository registry.aliyuncs.com/google_containers  \
                              --service-cidr=10.10.0.0/16 --pod-network-cidr=10.0.0.0/16
```

看到下面的输出说明初始化成功了

```shell
[root@master ~]# 
....
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.220:6443 --token 4qmsl0.vzecfcke47164729 \
    --discovery-token-ca-cert-hash sha256:c600b7a3562d486e60decb346a12d43182a8c08caf75b24fc630f5893a04970f

[root@master ~]# 
```    

然后按照提示操作

```shell
[root@master ~]# mkdir -p $HOME/.kube
[root@master ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@master ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 6. 安装calico网络

```shell
[root@master ~]# kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

## 7. 将从节点加入到集群中

这里依赖上一步master节点初始化之后的token   

```shell 
[root@master ~]# kubeadm join 192.168.0.220:6443 --token 4qmsl0.vzecfcke47164729 \
    --discovery-token-ca-cert-hash sha256:c600b7a3562d486e60decb346a12d43182a8c08caf75b24fc630f5893a04970f

```

## 8. 安装Dashboard

```shell
[root@master ~]# kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc6/aio/deploy/recommended.yaml
```

