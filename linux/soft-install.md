RPM安装

prm常用参数

-q 查询软件包

-i  安装软件包

-e  卸载软件包

获取rpm包一般有两种方式，一种是从网上下载，还有一种是通过系统光盘，系统光盘一般是在/dev/sr0

我们可以使用dd命令来制作iso文件，如下：

```shell
# dd if=/dev/sr0  of=/xxx/xxx.iso
```

找到光盘之后我们可以将光盘挂载到系统的某个目录下：

```shell
# mount /dev/sr0  /mnt
```

然后使用rpm进行操作

```shell
# rpm -qa  // 列出所有能过rpm安装的软件
# rpm -qa | more   // 分屏显示
# rpm -q  vi-common  // 查询vi-common是否安装
# rpm -i  vi-common-x.x.x-x.x.x.x86_64.rpm   
# rpm -e  vi-common  // 卸载

```

YUM安装

源地址：

CentOS 官方源：http://mirror.centos.org/centos/7/

国内阿里镜像：https://developer.aliyun.com/mirror/centos

修改为阿里镜像

备份

```shell
# mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

下载新的CentOS-Base.repo到/etc/yum.repos.d

CentOS 6 

```shell
# wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-6.repo
// 或者
# curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-6.repo
```

CentOS 7

```shell
# wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
// 或者
# curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

CentOS 8

```shell
# wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
// 或者
# curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
```

生成缓存

```shell
# yum makecache
```

非阿里云ECS用户出现Couldn't resolve host 'mirrors.cloud.aliyuncs.com' 

```shell
# sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
```

安装

```shell
# yum install xxx
# yum install -y xxx
# yum update 
# yum update xxx
# yum remove xxx
# yum list 
# yum grouplist 
```

源代码安装

```shell
# ./configure --prefix=/xx/xx
# make 
# make install
# make -j2   // 使用两个逻辑核心编译
```

gmake  跨平台编译