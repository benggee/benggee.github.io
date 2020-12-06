# 系统综合状态

### 使用sar命令

```shell
# sar -u 1 10  // cpu每隔1秒采集一次，采集10次
# sar -r 1 10  // 内存读写情况
# sar -b 1 10  // io
# sar -d 1 10  // 磁盘
# sar -q 1 10  // 进程的使用
```



### 使用第三方命令

```shell
# yum install epel-release 
# yum install iftop
# iftop  -P  // 网络情况，默认监听eth0
```

