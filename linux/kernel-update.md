# RPM格式内核

查看内核版本

```shell
# uname -r 
```

升级内核版本

```shell
# yum install kernel-3.10.0 
```

升级已安装的其它软件包和补丁

```shell
# yum update
```



# 源代码编译安装内核

### 安装依赖

```shell
# yum install gcc gcc-c++ make ncurses-devel openssl-devel elfutils-libelf-devel
```

小技巧

```shell
# yum install epel-release -y  // 扩展centos 的软件仓库
```



### 下载并解压内核

https://www.kernel.org

```shell
# tar xvf linux-5.1.10.tar.xz -C /usr/src/kernels
```

### 配置内核编译参数

```shell
# cd /usr/src/kernels/linux-5.1.10/
// menuconfig 是菜单式的配置
// allyesconfig 有的功能全都配置上
// allnoconfig 最小内核，这种是有可能启动不了的
# make menuconfig/allyesconfig/allnoconfig
```

除了自己make一个配置文件，还可以使用已有的配置文件

```shell
# cp /boot/config-kernelversion.platform /usr/src/kernels/linux-5.1.10/.config
```

查看cpu信息

```shell
# lscpu
// -j2 是根据lscpu里的信息来设置的
# make -j2 all  // all表示所有的都要安装
```

### 安装内核

```shell
# make modules_install
# make install
```



