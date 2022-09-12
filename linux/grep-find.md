# 文本查找与匹配



## 正则元字符

| 字符 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| .    | 匹配除换行符外的任意单个字符（注意：是单个字符）             |
| *    | 匹配任意多个跟在它前面的字符（注意：只匹配和前一个字符一样的字符） |
| []   | 匹配方括号中的字符类中任意一个                               |
| ^    | 以什么开头，例如以a开头 ^a                                   |
| $    | 匹配结尾                                                     |
| \    | 转义后面的特殊字符，在shell中，一般使用""将其包起来          |
| +    | 匹配前面的正则表达式至少出现一次                             |
| ？   | 匹配前面的正则表达式出现零次或一次                           |
| \|   | 匹配它前面或后面的正则表达式，将多个正则联合起来             |



## grep命令

**grep  option  file1[file2, file3]**

Option:

- -i 忽略大小写

  ```shell
  grep -i "server" ./Application.Server.log
  ```

- -r 递归读取

  ```shell
  grep -r "server" ./Application/
  ```

- -v 显示不匹配的pattern行

  ```shell
  grep -vi server ./Application.Server.log
  ```

- -n 显示行号

  ```shell
  grep -n "server" ./Application.Server.log
  ```

- -E 支持扩展正则，比如"|"

  ```shell
  grep -E "Client|connect" ./Application.Server.log
  ```

- -F 不支持正则，grep -F "aa\*" ... 会把\*当作普通字符

  ```shell
  grep -F "Client" ./Application.Server.log
  ```

- -c 只输出匹配行数，不显示内容

  ```shell
  grep -c "Client" ./Application.Server.log
  ```

- -w 匹配整词

  ```shell
  grep -w "Client" ./Application.Server.log
  ```

- -x 匹配整行

  ```shell
  grep -x "client" ./Application.Server.log
  ```

- -l 只列出文件名，不显示内容

  ```shell
  grep -l "client"
  ```



## cut命令 

显示第1列，以":"分隔

```shell
cut -d ":" -f1 /etc/passwd 
```

以":"分隔，统计每列中的每行出现的次数（注意：uniq只统计相邻的）

```shell
cut -d ":" -f1 /etc/passwd | uniq -c
```

以":"分隔，统计每列中的每行出现的次数，先排序再统计

```shell
cut -d ":" -f1 /etc/passwd | sort | uniq -c 
```

以":"分隔，统计每列中的每行出现的次数，先排序再统计，将统计结果再排序

```shell
cut -d ":" -f1 /etc/passwd | sort | uniq -c | sort
```

wc统计行数

```shell
cut -d ":" -f1 /etc/passwd | wc -l
```



## find命令

**find   path  option [操作]**

Option:

- -name  指定文件名
- -iname 指定文件名(不区分大小写)
- -user 指定文件属主
- -group 指定文件属组
- -type 指定文件类型
  - f  文件
  - d  目录
  - c 字符设备文件
  - b 块设备文件
  - l 链接文件
  - p 管道文件
- -size 指定大小
- -atime 访问时间 
- -mtime 修改时间
- -ctime  change时间，比如修改属主
- -mmin  指定修改时间(分钟单位)
- -mindepth n   表示从n组子目录开始搜索
- -maxdepth n  表示最多搜索n组子目录
- -nouser  查看没有属主文件
- -nogroup  查看没有属组文件
- -perm  通过文件权限查找文件 例如：find -perm 664
- -prune 通常和-path一起使用，用于将特定目录排除在外



**找某个文件**

```shell
find  /etc  -name  passwd
find  /etc  -regex .*wd
```



**根据时间查找**

```shell
find /var/log -name '\*.log' -mtime +7  // 修改时间7天以内
find /var/log -atime 8 -regex .*wd   // 8小时以前
find /var/log -ctime 8 -regex .*wd 
```

查看文件mtime/atime/ctime

```
$ stat kpi.txt
...
Access: 2022-09-11 18:37:19.859025232 +0800    atime
Modify: 2022-09-11 18:36:30.902725986 +0800	   mtime
Change: 2022-09-11 18:36:30.906726010 +0800    ctime
...
```

加上时区

```
$ LANG=C stat kpi.txt    // 英语
...
Access: 2022-09-11 18:37:19.859025232 +0800    atime
Modify: 2022-09-11 18:36:30.902725986 +0800	   mtime
Change: 2022-09-11 18:36:30.906726010 +0800    ctime
...
```



**排队某个目录 **

```
// 查找当前目录所有普通文件，排除test目录 
find ./ -path ./test -prune -o -type f

// 查找当前目录下所有普通文件，排队etc和opt目录
find ./ -path ./etc -prune -o -path ./opt -prune -o -type f

// 查找当前目录所有普通文件，排队etc和opt目录，属主为hdfs
find ./ -path ./etc -prune -o -path ./opt -prune -o -type f -user hdfs
```



**比某个文件更新的文件**

```shell
find  ./  -newer aaa.txt -type f
```



**进一步操作（删除、打印）**

```
// 搜索/etc下的文件，文件名以conf结尾，大于10K, 将其删除
find  ./etc -type f -name '\*.conf' -size +10k -exec rm -f {} \;  // {}表示所有结果

// 将/var/log目录下以log结尾的文件，且修改时间在7天内的文件删除
find /var/log -name '\*.log' -mtime +7 -exec rm -f {}\;

// 搜索/etc下的文件，文件名以conf结尾，大于10K, 将其复制到/root/conf目录下
find ./etc  -size +10k -type f -name '\*.conf' -exec cp {} /root/conf \;

// 交互式删除
find /tmp -regex '.*txt' -ok rm {} \;
```



批量新建文件

```
touch /tmp/{1..9}.txt
```



**逻辑操作**

- -a  与
- -o  或
- -not | ! 非

```shell
// 查找当前目录，属主不是hdfs的所有文件
find ./ -not -user hdfs | find ./ ! -user hdfs

// 查找当前目录，属主属于hdfs, 且大于300字节的文件
find ./ -type f -a -user hdfs -a -size +300c

// 查找当前目录属主为hdfs或者xml结尾的普通文件
find ./ -type f -a \(-user hdfs -o -name '\*.xml' \)
```



## Whereis

- -b 只返回二进制文件
- -m  只返回帮助文档
- -s 只返回源代码



## which

仅查找二进制文件