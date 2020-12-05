# Grub配置

列出当前默认引导的第几个内核版本

```shell
# grub2-editenv list
```

查找所有内核版本

```shell
# grep ^menu  /boot/grub2/grub.cfg
```

然后设置默认引导内核

```shell
# grub2-set-default 0  // 这里的0是查找到的内核版本列表的第几个
```



##  

重启系统，选择对应的引导内核版本，按e键进入编辑界面

然后找到linux16  /vmlinuzxxxx这一行最后添加一个设置项 single，如果是centos7应该是rd.break

然后Ctrl + x 

进入到内存虚拟的文件系统的根目录，要注意这个根目录并非是系统的根目录。此时，系统的目录在  /sysroot下面

进入到系统根目录

```shell
# mount -o remount,rw   // rw 表示读写 
# chroot /sysroot  // 将sysroot当作根目录使用
```

设置root用户密码

```shell
# echo new_password | passwd --stdin  root 
```

关闭SELinux 

```shell
# vi /etc/selinux/config
```

将SELINUX的值改为 disabled

重启

```shell
# exit 
# reboot
```

