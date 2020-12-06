# SELinux

SELinux的原理是比对各个服务、用户、文件的target，如果对得上说明是安全的

### 查看SELinux

```shell
# getenforce
# /usr/sbin/sestatus
// 下面是查看target方式
# ps -Z 
# ls -Z 
# id -Z
```



### 关闭SELinux

临时关闭

```shell
# setenforce 0
```

永久关闭

/etc/selinux/sysconfig

