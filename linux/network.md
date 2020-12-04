# 网络查看工具

## net-tools工具集

net-tools工具集是CentOS 7.0之前网络工具集，工具集有：ifconfig 、route、netstat

```shell
// 网卡的禁用与启用
# ifconfig eth0 up 
# ifconfig eth0 down 
# ifup eth0
# ifdown eth0
// 修改ip地址
# ifconfig eth0 192.168.0.1 
# ifconfig eth0 192.168.0.1 netmask 255.255.255.0
// 路由管理
# route add default gw 192.168.0.1
# route add -host 192.168.0.22 gw 192.168.0.1  // 表示到0.22的机器走0.1网关
# route add -net 192.168.0.0 netmask 255.255.255.0 gw 192.168.0.1 // 指定网段
```

## iproute2 工具集

iproute2是CentOS 7.0以后的版本新的一套网络工具集，包括：ip 、ss

```shell
# ip addr ls 
# ip link set dev eth0 up   // 相当于ifup eth0
# ip addr add 10.0.0.1/24 dev eth1 // 相当于ifconfig eth1 10.0.0.1 netmask 255.255.255.0
# ip route add 10.0.0.1/24 via 192.168.0.1 // 相当于route add -net 10.0.0.0 netmask 255.255.255.0 gw 192.168.0.1
```



# 网络接口命名修改

网卡命名规则可以通过修改biosdevname和net.ifnames参数来修改

编辑/etc/default/grub，增加biosdevname=0  net.ifnames=0

然后更新grub

```shell
# grub2-mkconfig -o /boot/grub2/grub.cfg
```

最后重启生效



# 网络状况查看

查看网卡物理连接情况

```shell
# mii-tool  eth0
```

查看网关

```shell
# route -n  // -n 表示不解析主机名
```



# 网络故障排查

ping、traceroute、mtr、nslookup、telnet、tcpdump、netstat、ss



# 网络服务管理

网络服务管理程序分为两种，分别是SysV和systemd

```shell
// SysV
# service network start|stop|restart
# chkconfig --list network   // 查看对应等级是否打开
// systemd
# systemctl list-unit-files NetworkManager.service // 查看NetworkManager是否开启
# systemctl start|stop|restart NetworkManager
# systemctl enable|disable NetworkManager   // 关闭或开启NetworkManager

```

从CentOS 7.0开始提供了两种服务管理方式，但一般我们只需要其中一个就可以了，我们可以使用对应的命令选择关闭或打开服务即可

# 网络配置文件

网络配置文件一般在/etc/sysconfig/network-script/ifcfg-xxx

Host文件一般在/etc/hosts

# Hostname管理

```shell
# hostname // 查看主机名
# hostnamectl  set-hostname centos7
```

注意，一般我们修改了主机名之后需要在/etc/hosts里面加一条记录