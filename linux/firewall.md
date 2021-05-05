防火墙



iptables

包防火墙，底层是内核netfilter去过滤

规则表

iptables 能支持的功能

filter

nat   网络地址转换

mangle

raw 



规则链

INPUT  OUTPUT  FORWARD（转发）

PREROUTING（路由前置化）  POSTROUTING（路由后置化）  选路之前或者之后进行前置化操作



iptables -t filter 命令  规则链  规则

命令

-L

-A -l

-D -F -P

-N -X -E



规则 

-p

-s -d

-i -o

-j



示例:

查看filter规则链

```bash
[root@localhost ~]# iptables -t filter -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

添加一条规则链

```bash
[root@localhost ~]# iptables -t filter -A INPUT -s 192.168.1.225 -j DROP
```

查看规则链（更详细）

- -v 表示显示更详细的信息
- -n 表示不解析ip背后的域名

```bash
[root@localhost ~]# iptables -t filter -vnL 
Chain INPUT (policy ACCEPT 156 packets, 11604 bytes)
 pkts bytes target     prot opt in     out     source               destination
  153 12466 DROP       all  --  *      *       192.168.1.225        0.0.0.0/0

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 100 packets, 8800 bytes)
 pkts bytes target     prot opt in     out     source               destination
```

添加规则链到第一条

iptables的规则链是由上到下解析的，以最上面一条为准，所以顺序很重要，可以使用-I 参数来添加到第一条。

```bash
[root@localhost ~]# iptables -t filter -I INPUT -s 192.168.1.225 -j ACCEPT
```



阻止所有请求，开放一部分的规则配置

配置默认规则

```bash
[root@localhost ~]# iptables -P INPUT DROP 
[root@localhost ~]# iptables -vnL
Chain INPUT (policy DROP 91 packets, 7124 bytes)
 pkts bytes target     prot opt in     out     source               destination
   57  5688 ACCEPT     all  --  *      *       192.168.1.225        0.0.0.0/0

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 69 packets, 6280 bytes)
 pkts bytes target     prot opt in     out     source               destination
```

注意，如果是用ssh连接到主机上，这个操作会使当前链接立即断开

清除所有规则

```bash
[root@localhost ~]# iptables -F 
```

删除规则

```bash
[root@localhost ~]# iptables -D INPUT -s 192.168.1.225 -j ACCEPT
```

允许某一段IP

```bash
[root@localhost ~]# iptables -A INPUT -s 10.0.0.0/24  -j  ACCEPT
```

限制网卡、端口、协议

```bash
[root@localhost ~]# iptables -t filter -A INPUT -i eth0 -s 10.0.0.2 -p tcp --dport 80 -j ACCEPT 
```

设置所有规则

```bash
[root@localhost ~]# iptables -t filter -A INPUT -j ACCEPT # 或者DROP
```



iptables -t nat 命令  规则链   规则

PREROUTING  目的地址转换

```bash
[root@localhost ~]# iptables -t nat -A PREROUTING -i eth0 -d 114.115.116.117 -p tcp --dport 80 -j DNAT --to-destination 10.0.0.1
```

POSTROUTING  源地址转换

将本地ip伪装成外网ip

```bash
[root@localhost ~]# iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth1 -j SNAT --to-source 111.112.113.114
```



iptables 配置文件

/etc/sysconfig/iptables

centos6

service  iptables  save | start | stop | restart

centos7

yum install iptables-services

保存配置文件

```bash
[root@localhost ~]# iptables-save > /etc/sysconfig/iptables
```



Firewalld

```bash
[root@localhost ~]# firwall-cmd --state
running
[root@localhost ~]# firwall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources:
  services: dhcpv6-client ssh
  ports: 22/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

查看信息

```bash
[root@localhost ~]# firewall-cmd --zone=public --list-interfaces 
eth0
[root@localhost ~]# firewall-cmd --list-interfaces 
eth0
[root@localhost ~]# firewall-cmd --list-ports 
22/tcp
[root@localhost ~]# firewall-cmd --list-services 
dhcpv6-client ssh
[root@localhost ~]# firewall-cmd --get-zones 
block dmz drop external home internal public trusted work
[root@localhost ~]# firewall-cmd --get-default-zone
public
[root@localhost ~]# firewall-cmd --get-active-zone 
public
  interfaces: eth0
```

添加规则

```bash
[root@localhost ~]# firewall-cmd --add-service=https
success
[root@localhost ~]# firewall-cmd --add-port=81/tcp
success
```

持久化

```bash
[root@localhost ~]# firewall-cmd --add-prot=82/tcp --permanent
[root@localhost ~]# firewall-cmd --reload
```

删除规则

```bash
[root@localhost ~]# firewall-cmd --remove-source=10.0.0.1
```

