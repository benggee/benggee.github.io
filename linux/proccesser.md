# 进程的管理

#### 查看进程

ps、pstree、top

```shell
# ps 
# ps -e  // 查看所有进程
# ps -el // 显示UID等更多其它信息
# ps -elf // 显示线程
```

其它更多用法可以使用 man  ps   

```shell
# pstree 
```



```shell
# top
```

top 按1  可以显示所有逻辑cpu的信息

top 按s  可以修改刷新时间间隔



# 进程的控制

**进程优先级调整**

nice、renice

```shell
# nice -n 10 ./a.sh    // 这种方式会启动一个新的a.sh进程
# renice -n 10 1982    // 这种方式会将已经启动的进程1982的优先级调整为10
```

进程前后台运行、切换

```shell
# ./a.sh  &    // 后台运行 
# jobs    // 查看所有后台进程
# fg  1   // 将1号后台进程调入到前台
```

然后使用Ctrl + z 可以将进程调入后台



**进程间通信与信号**

```shell
# kill -l   // 查看所有信号
```

然后我们可以使用kill  + 信号 来发送信息到进程，比如kill -9 



# 守护进程

nohup可以实现和守护进程差不多的效果，使用nohup与& 符号运行一个命令，nohup命令使进程忽略hangup(挂起)信号

```shell
# nohup tail -f /var/log/messages  & // 关掉终端还继续运行
```

忽略输入并把输出追加到当前目录的“nohup.out”文件

此时，我们可以去/proc目录查看进程信息，proc目录下是将进程的内存信息以文件的形式保存起来的

假设上面的tail进程的进程号是2810

```shell
# cd /proc/2810   // 进入到进程2810的目录
# ls -l cwd   // 当前在哪个位置，这个取决于运行这个程序的终端使用的是哪个用户，如果是root，那对应的就是/root
# ls -l fd    // 输入输出信息  0 标准输入 1、2 标准输出 
```

可以查看一个daemon的例子

```shell
# ps -ef | grep sshd  // 查看sshd的进程ID
# cd /proc/ssh进程ID
# ls -l cwd   // 当前所在的位置是/目录
```

通过上面nohup和daemon的对比，我们可以看到最大区别在于所以目录的不一样，nohup取决于当前用户，并且挂载的目录是不能卸载的。deamin始终都是在根目录，只有当系统重启或关闭的时候才有机会卸载。总结下来就是nohup会占用目录导致不能卸载。

#### 使用Screen命令

Screen命令可以在终端退出的时候进程不退出，同时可以实时显示执行结果。

```shell
# screen   // 进入终端执行以下命令
// ctrl + a  然后按 d  退出 
# screen  -ls // 查看有哪些screen会话
# screen -r sessionid 恢复会话
```

#### 系统日志

/var/log/messages     常规日志

/var/log/dmesg    内核相关信息

/var/log/secure    安全相关

/var/log/cron    计划任务相关

