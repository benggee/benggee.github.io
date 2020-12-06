# 内存查看

```shell
# free
# free -m
# free -h

# top
```

# 磁盘查看

```shell
# fdisk
# df -h
# ls -lh  /xxx
# du /xxx    //实际占用空间
# dd if=/dev/zero  bs=4M count=10 of=afile   // 表示从dev/zero里面每次读4M一共读10次，写入afile
# dd if=/dev/zero  bs=4M count=10 seek=20 of=afile  // seek=20表示跳过20个块之后再开始写入
```

# 文件系统

ext4、xfs、NTFS

ext4文件系统基本结构

- 超级块

​	记录了文件系统、分区中文件总数，在创建文件的时候计算的

- 超级块副本

​	超级块的副本

- i节点（inode）

​	记录每个文件的信息，名称、大小、编号、权限，文件名记录在文件节点父节点的i节点

```shell
# ls -i  // 查看i节点信息
```

- 数据块（datablock）

​	实际数据存放的位置，如果一个数据块装不下，会创建多个数据块挂在i节点

在统计文件个数和大小的时候，ls查看的是i节点的信息，du统计的是数据块的实际信息，所以这两者有一定的差别

# i节点和数据块操作

以一个文件afile为例

```shell
# touch afile
# ls -li afile 
```

结果如下：

12053 -rw-r--r--. 1 root root 0 12月  6 02:32 afile

其中12053就是i结节编号

文件创建默认是4K， 如何创建小文件？？

#### 对于cp来讲

复制出来的文件的i节点和datablock都是新的

#### 对于mv来讲

只会修改父目录里记录的文件名，但如果mv是将文件移动到其它分区，会重建i节点和datablock

#### 对于vim来讲

修改文件会修改文件的i节点

#### 对于rm来讲

将文件名和i节点断开

#### ln增加i节点链接

**硬链**

硬链会将两个文件指向同一个i节点

```shell
# ln afile bfile  
```

**软链**

软链两个文件的i节点是不一样的，会创建一个新的文件

```shell
# ln -s afile aafile
```

文件权限管理facl

```shell
# getfacl afile
# setfacl -m u:user1:r afile  // -m表示赋予权限，-x表示收回权限   -u表示用户 -g表示组 
```

# 分区和挂载

fdisk

mkfs

parted     大于2T的硬盘

mount   

配置文件  /etc/fstab

```shell
# fdisk -l  // 查看设备
# fdisk /dev/sdc  // 对sdc分区
  > n  // 新建分区  
  > p  // 显示分区
  > w  // 保存

# mkfs.ext4 /dev/sdc1   // 格式化
# mkdir /mnt/sdc1
# mount -t ext4  /dev/sdc1  /mnt/sdc1
# mount -t auto  /dev/sdc1  /mnt/sdc1
# mount /dev/sdc1 /mnt/sdc1
# mount  // 查看挂载情况
```

修改/etc/fstab文件固化

UUID=2df301b1-8c07-4d01-b861-b6f447368046  / ext4    defaults,noatime 0 0
UUID=4BCA-2C96  /boot vfat    defaults,noatime 0 0
UUID=58dfd3b7-8eb2-41d6-b98f-e01e03127478  swap swap    defaults,noatime 0 0

增加如下配置

/dev/sdc1   /mnt/sdc1   ext4  defaults  0 0

# 用户磁盘配额

这里以xfs文件系统为例

```shell
# mkfs.xfs  /dev/sdb1
# mkdir /mnt/disk1
# mount -o uquota,gquota  /dev/sb1  /mnt/disk1
# chmod 1777  /mnt/disk1
# xfs_quota -x -c 'report -ugibh' /mnt/disk1
# xfs_quota -x -c 'limit -u isoft=5 ihard=10 user1' /mnt/disk1
```

# 交换分区查看与创建

使用分区方式

```shell
# fdisk /dev/sdd  // 建立分区
# mkswap /dev/sdd1 
# swapon /dev/sdd1
# free -m 
# swapoff /dev/sdd1
```

使用文件方式

需要准备一个对应大小的文件

```shell
# dd if=/dev/zero bs=4M count=1024 of=/swapfile
# chmod 600 /swapfile
# mkswap  /swapfile 
# swapon /swapfile
# free -m 
```

固化/etc/fstab

/swapfile   swap   swap  defaults  0 0

# 软件RAID的使用

RAID常见级别

RAID 0 striping 条带方式，提高单盘吞吐率

RAID 1 mirroring  镜像方式，提高可靠性

RAID 5 有奇偶校验

RAID 10 是RAID 1与RAID 0的结合

```shell
# yum install mdadm
...  创建三个大小一样的分区 sdb1  sda1  sdc1
// -a 表示同意创建分区
// -l 级别
// -n 几块硬盘是活动的
# mdadm -C /dev/md0 -a yes -l1 -n2 /dev/sd[a,b]1 
# mdadm -D /dev/md0   // 查看信息
// 增加配置项
# echo DEVICE /dev/sd[a,b]1 
# echo DEVICE /dev/sd[a,b]1  >> /etc/mdadm.conf
# mdadm -Evs >> /etc/mdadm.conf
// 格式化
# mkfs.xfs  /dev/md0
# mkdir /mnt/md0
# mount /dev/md0 /mnt/md0
# mdadm --stop /dev/md0
# dd if=/dev/zero of=/dev/sdb1 bs=1M count=1  // 破坏分区
```

​	此时就可以往/mnt/md0写数据了

# 逻辑卷LVM的使用

1. 添加硬盘

   这里假如我们有sda1、sdb1、sdc1

2. 创建逻辑卷

   ```shell
   # pvcreate /dev/sd[a,b,c]1
   # vgcreate vg1 /dev/sdb1 /dev/sdc1   // 将sdb1和sdc1加入到逻辑卷组vg1
   # pvs  // 查看卷组信息
   # vgs  // 查看有哪些卷组
   # lvcreate -L 100M -n lv1  vg1   // 创建逻辑卷 -n表示名字 vg1表示从vg1逻辑卷组创建
   # mkdir /mnt/lv1
   # mkfs.xfs /dev/vg1/lv1
   # mount /dev/vg1/lv1 /mnt/lv1
   ```

   如果要固化，编辑/etc/fstab

​       在上面其实还可以加一层RAID

3. 扩充root目录LV

   ```shell
   # mount | grep root
   # lvs 
   # vgextend centos  /dev/sdd1扩充LV
   ```

4. 扩充LV

   ```shell
   # lvextend -L +50G /dev/centos/root
   # lvs
   ```

5. 文件系统扩充

   ```shell
   # df -h
   # xfs_growfs /dev/centos/root
   # df -h
   ```



## 实例1：解决树莓派空间缩水的问题

```shell
# df -h 
# cat /sys/block/mmcblk0/mmcblk0p2/start  // 查看第二分区的起始地址，后面分区的时候的起始位置 
# fdisk /dev/mmcblk0
	> d 删除分区
	> n 创建分区
	> p 创建主分区
	> 输入第一次得到的起始扇区
	> w
# reboot
# resize2fs /dev/mmcblk0p2
```

