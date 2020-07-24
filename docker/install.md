### CentOS7及以上版本的安装

#### Dcoker安装

安装依赖   
```shell
[root@docker ~]# yum install vim bash-completion net-tools gcc -y
[root@docker ~]# yum install -y yum-utils device-mapper-persistent-data lvm2
```

添加阿里云的源   
```shell 
[root@docker ~]# yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

安装docker    
```shell
`[root@docker ~]# yum -y install docker-ce`

如果出现错误    
```shell
[root@docker ~]# Error:
                   Problem: package docker-ce-3:19.03.8-3.el7.x86_64 requires containerd.io >= 1.2.2-3, but none of the providers can be installed
```

解决方法    
```shell
[root@docker ~]# wget https://download.docker.com/linux/centos/7/x86_64/edge/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
[root@docker ~]# yum install containerd.io-1.2.6-3.3.el7.x86_64.rpm
```

然后再运行yum -y install docker-ce就可以了

配置阿里云加速器    
```shell
[root@docker ~]# mkdir -p /etc/docker

[root@docker ~]# tee /etc/docker/daemon.json <<-'EOF'
                  {
                    "registry-mirrors": ["https://fl791z1h.mirror.aliyuncs.com"]
                  }
                  EOF      
[root@docker ~]# systemctl daemon-reload
[root@docker ~]# systemctl restart docker
```

#### Docker-compose安装
```shell
[root@docker ~]# sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
[root@docker ~]# sudo chmod +x /usr/local/bin/docker-compose
```