Etcdctl使用指南

github地址：https://github.com/etcd-io/etcd/tree/main/etcdctl

指定Etcdctl版本：

 export ETCDCTL_API=3



## etcdct常用命令

查看所key

```shell
$ etcdctl --endpoints=http://localhost:2379 ls /
```

