# Redis集群部署

## 机器准备

操作系统：CentOS 7.8  amd64 

内存：2G 

CPU：2core

| HOST  | IP            | 说明  |
| ----- | ------------- | ----- |
| node1 | 192.168.1.221 | 节点1 |
| node2 | 192.168.1.222 | 节点2 |
| node3 | 192.168.1.223 | 节点3 |



### 环境初始化

```bash
[root@master ~]# yum install -y gcc g++ tcl jemalloc 
```



### 下载redis源码并安装

```bash
[root@master ~]# wget https://download.redis.io/releases/redis-6.2.2.tar.gz
[root@master ~]# tar -zxvf redis-6.2.2.tar.gz
[root@master ~]# cd redis-6.2.2 
[root@master redis-6.2.2]# make MALLOC=libc
[root@master redis-6.2.2]# make test
[root@master redis-6.2.2]# make install
```



### 构建redis的base目录 

```bash
[root@master ~]# mkdir -p /usr/local/redis-6.2/bin
[root@master ~]# cp redis-6.2.2/src/redis-server /usr/local/redis-6.2/bin
[root@master ~]# cp redis-6.2.2/src/redis-cli /usr/local/redis-6.2/bin
[root@master ~]# cp redis-6.2.2/src/redis-benchmark /usr/local/redis-6.2/bin
```



### 加入到环境变量

```bash
[root@master ~]# vi /etc/profile.d/path.sh
export PATH=/usr/local/redis-6.2/bin:$PATH
[root@master ~]# source /etc/profile
```



## Cluster部署

我们在每台机器上模拟出两个实例（通过不同的端口号），但生产环境为了容灾和可用性，应该使用物理机

下面在每台机器上分别进行下面的操作：

```bash
[root@master ~]# cd /usr/local/redis-6.2/
[root@master redis-6.2]# mkdir cluster/6379  cluster/7000
```

然后在6379和7000文件夹下创建配置文件redis.conf，内容如下：

```
port 6379
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

注意，port要改为文件夹对应的端口号

然后分别进入6379和7000文件夹，进行如下操作：

```bash
[root@master 6379]# redis-server ./redis.conf &
```

最后，我们配置cluster，如下：

```bash
[root@master ~]# redis-cli --cluster create 192.168.1.221:6379 192.168.1.221:7000 \
 192.168.1.222:6379 192.168.1.222:7000 192.168.1.223:6379 192.168.1.223:7000 \
 --cluster-replicas 1

>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.1.221:6379 to 192.168.1.221:7000
Adding replica 192.168.1.222:6379 to 192.168.1.222:7000
Adding replica 192.168.1.223:6379 to 192.168.1.223:7000
...
Can I set the above configuration? (type 'yes' to accept): yes
...
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
[root@iZbp1ei15txqpd9cc8ri50Z 7001]# 24515:S 30 Apr 2021 15:52:06.306 * AOF rewrite child asks to stop sending diffs.
24529:C 30 Apr 2021 15:52:06.306 * Parent agreed to stop sending diffs. Finalizing AOF...
24529:C 30 Apr 2021 15:52:06.306 * Concatenating 0.00 MB of AOF diff received from parent.
24529:C 30 Apr 2021 15:52:06.306 * SYNC append only file rewrite performed
24529:C 30 Apr 2021 15:52:06.307 * AOF rewrite: 0 MB of memory used by copy-on-write
24507:M 30 Apr 2021 15:52:06.315 * Background saving terminated with success
24507:M 30 Apr 2021 15:52:06.315 * Synchronization with replica 121.43.234.180:7001 succeeded
24515:S 30 Apr 2021 15:52:06.385 * Background AOF rewrite terminated with success
24515:S 30 Apr 2021 15:52:06.385 * Residual parent diff successfully flushed to the rewritten AOF (0.00 MB)
24515:S 30 Apr 2021 15:52:06.385 * Background AOF rewrite finished successfully
24507:M 30 Apr 2021 15:52:10.050 # Cluster state changed: ok
```

注意，中间有一步是确认配置信息，要输入yes确认

到这里，cluster就部署完成了，下面我们来验证一下：

在任意node上

```bash
[root@master ~]# redis-cli -p 6379 -c 
[root@master ~]# set foo bar
```

然后我们在刚刚这个节点上连接上另一个实例

```bash
[root@master ~]# redis-cli -p 7000 -c 
[root@master ~]# get foo
-> Redirected to slot [10439] located at 192.168.1.221:6379
"bar"
```

设置密码

登陆每个节点，执行以下操作：

```shell
./redis-cli -c -p 7000 
config set masterauth passwd123 
config set requirepass passwd123 
config rewrite 
```

注意，-c表示的是cluster模式
