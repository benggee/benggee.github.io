linux下的文本处理

# 查找操作

## find

**find  [路径] [选项] [操作]**

### options:

**-name**     指定文件名(区分大小写)

**-iname**    指定文件名（不区分大小写）

**-user**       指定文件属主

**-group**     指定文件属组

**-type**       指定文件类型，取值如下

- **f**       文件 
- **d**      目录
- **c**      字符设备文件
- **b**      块设备文件
- **l**       链接文件
- **p**      管道文件

**-size**       指定大小，使用如下

**-n**    小于n的文件，例如：find  /etc  -size  -1M    表示在etc目录下找小于1M的文件

**+n**   大于n的文件

**-mtime**   指定修改时间（天为单位）

**-n**    表示n天以内的文件

**+n**   表示n天以外的文件

**n**     表示正好n天修改的文件

**-mmin**    指定修改时间 （分钟为单位）

**-n**    表示n分钟以内修改的文件

**+n**   表示n分钟以前修改的文件

**-mindepth**  **n**   表示从n组子目开始搜索

**-maxdepth** **n**   表示最多搜索n组子目录

### Ohter options

**-nouser**       查看没有属主文件

**-nogroup**     查找没有属组的文件

**-perm**          通过文件权限查找文件  例如：find  -perm  664

**-prune**        通常和-path一起使用，用于将特定的目录排除在外

**例1：查找当前目录下所有普通文件，但排除test目录**

```shell
find  .  -path  ./test  -prune -o -type f
```

**例2：查找当前目录下所有普通文件，排除etc和opt目录**

```shell
find  .  -path  ./etc -prune -o  -path  ./opt  -prune  -o  -type f
```

**例3：查找当前目录下所有普通文件，排除etc和opt目录，属主为hdfs**

```shell
find  .  -pth  ./etc  -prune  -o -path  ./opt  -prune  -o  -type  f -a  user  hdfs
```

**-newer**  **file1**   查找比file1新的文件

**操作**

**-print**       打印输出

**-exec**       对搜索到的文件执行特定的操作，格式为 -exec   'command'  {} \;    

**例1：搜索/etc下的文件（非目录）,文件名以conf结尾，且大于10K，然后将其删除**

```shell
find  ./etc/ -type f  -name ‘\*.conf’  -size  +10k  -exec  rm -f {} \;   # {}表示所有结果
```

**例2：将/var/log/目录下以log结尾的文件，且更改时间在7天以的的文件删除**

```shell
find  /var/log  -name '\*.log'  -mtime +7  -exec  rm -f {}\;
```

**例3：搜索条件和例1一样，只是不删除，而是将其复制到/root/conf目录下**

```shell
find  ./etc/  -size +10k  -type f  -name '\*.conf'  -exec cp {} /root/conf \;
```

**逻辑运算符**

**-a**        与

**-o**        或

**-not|!**   非

**例1：查找当前目录下，属主不是hdfs的所有文件**

`find  .  -not  -user hdfs |  find  . !  -user  hdfs`

**例2：查找当前目录下，属主属于hdfs，且大小大于300字节的文件**

`find  .  -type f -a  -user  hdfs  -a  -size +300c`

**例3：查找当前目录下的属主为hdfs或者以xml结尾的普通文件**

`find  .  type  f  -a  \( -user  hdfs -o -name '\*.xml' \)`

** locate 命令**

locate命令在数据库文件中查找

locate命令是模糊匹配

可以使用updatedb命令来更新数据库，数据位置：/var/lib/mlocate/mlocate.db, 使用的配置文件：/etc/updatedb.conf

\> updatedb    # 更新数据库

## whereis

**-b**      只返回二进制文件

**-m**     只返回帮助文档文件

**-s**      只返回源代码文件

## which

仅查找二进制文件

**-b**    只返回二进制文件

# grep的高频操作

## grep的语法格式

**第一种形式：grep  [option] [pattern]  [file1, file2...]**

**第二种形式：command  |  grep  [option] [pattern]**

## option说明

**-v**        显示不匹配的pattern的行

`grep  -vi  server  ./Application.Server.log`

**-i**         忽略大小写

`grep -i "server" ./Application.Server.log`

**-n**        显示行号

`grep  -n  "Server"  ./Application.Server.log`

**-E**       支持扩展正则表达式  等价 egrep

**grep  -E  "Client|connect"  ./Application.Server.log**   # 加上-E参数之后“|”正则式就会生效

**-F**       不支持正则表达式，按字符串的字符字面意思匹配

**grep  -F  "Client\*"  ./Application.Server.log**     #  加上-F参数会原样匹配Client*,把“*”当作普通内容

**-r**       递归搜索 

**grep  -r  "Client"**     # 会在当前目录的所有子目录进行匹配

**-c**       只输出匹配行的数量，不显示具体内容

**grep  -c "Client"  ./Application.Server.log**    #  输出匹配到的行数

**-w**      匹配整词

**grep  -w   "Client"  ./Application.server.log**   # 如果Client两边没有空格则不会匹配

**-x**       匹配整行

**grep  -x  "Client"  ./Application.Server.log**  # 只有当Client占一行的时候才会匹配

**-l**        只列出匹配的文件名，不显示内容 

**grep -l  "Client"** 

# sed的使用

## sed的两种形式

\> stdout | sed  [option] "pattern command" > sed [option] "pattern command"  file

## option

**-n     只打印模式匹配行**

**-e     直接在命令行进行sed编辑, 默认选项, 可以显示-e 指定多个**

**-f      编辑动作保存在文件中，指定文件执行**

**-r      支持扩展正则表达式**

**-i      直接修改文件内容**

## pattern

**10command                       #匹配到第10行**

**10,20command                    #匹配从第10行开始，到第20行结束**

**10,+5command                    #匹配从第10行开始，到第16行结束**

**/pattern1/command               #匹配到pattern1的行**

**/pattern1/,/pattern2/command    #匹配pattern1开始的行到pattern2开始的行结束**

**10,/pattern1/command            #匹配从第10行开始到pattern1的行结束**

**/pattern1/,10command            #匹配到pattern1的行开始到第10行匹配结束**

**pattern示例：**

```shell
# 直接指定行号 
> sed -n "17p"  file      
# 打印file文件的第17行 # 指定起始行号和结束行号 
> sed -n "10,20p" file    
# 打印file文件的10到20行 # 指定起始行号，然后后面N行 
> sed -n "10,+5p" file    
# 打印file文件中从第10行开始，往后面5行的所有内容 # 正则表达式匹配的行 
> sed -n "/^root/p" file  
# 打印file文件中以root开头的行 # 匹配从pattern1的行到pattern2的行 
> sed -n "/^ftp/,/^mail/p" file   
# 打印file中ftp开头的行到mail开头的行 # 从指定行号开始匹配，直到匹配到pattern 
> sed -n "4,/^hdfs/p"  file       
# 打印file文件中从第4行开始匹配，直到以hdfs开头的行 
# 从pattern匹配的行开始，直到匹配到指定的行 
> sed -n "/root/,10p" file   
```

## sed中的编辑命令

### **查询** 

**p    打印**

```shell
# 1. 打印passwd中第20行的内容 
> sed  -n '20p' /etc/passwd 
# 2. 打印passwd中从第8行开始，到15行结束的内容 
> sed -n '8,15p' /etc/passwd 
# 3. 打印passwd中从第8行开始，然后+5行结束的内容 
> sed -n '8,+5p' /etc/passwd 
# 4. 打印passwd中开头匹配hdfs字符串的内容 
> sed -n '/^hdfs/p' /etc/passwd 
# 5. 打印passwd中开头为root的行开始，到开头为hdfs的行结束的内容 
> sed -n '/^root/,/^hdfs/p' /etc/passwd 
# 6. 打印passwd中第8行开始，到含有/sbin/nologin的内容的行结束的内容 
> sed -n '8,/\/sbin\/nologin/p' /etc/passwd 
# 7. 打印passwd中第一个包含/bin/bash内容的行开始，到第5行结束的内容 
> sed -n '/\/bin\/bash/,5p'  /etc/passwd 
```

### 增加

​    **a   行后追加**

​    **i    行前追加**

​    **r    外部文件读入，行后追加**

​    **w   匹配行写入外部文件** 

**（1） a    append** 

```shell
# 1. passwd文件第10行后面追加“Add Line Behind” 
> sed -i '10a Add Line Begind' passwd 
# 2. passwd文件第10行到20行，每一行后面都追加“Test Line Behind” 
> sed -i '10,20a Test Line Behind' passwd 
# 3. passwd文件匹配到/bin/bash的行后面追加“Insert Line For /bin/bash Behind” 
> sed -i "/\/bin\/bash/a Insert Line For /bin/bash Behind" passwd
```

**（2）i**  

```shell
# 1. passwd文件匹配到以yarn开头的行，在匹配行前面加上“Add Line Before” 
> sed -i '/^yarn/i Add Line Before' passwd 
# 2. passwd 文件每一行前面都追加“Insert Line Before Every Line” 
> sed -i 'i Insert Line Before Every Line' passwd
```

**（3）r**

```shell
# 1. 将fstab文件的内容追加到passwd文件的第20行后面 
> sed -i '20r /etc/fstab' passwd 
# 2. 将inittab文件内容追加到passwd文件匹配/bin/bash行的后面 
> sed -i '/\/bin\/bash/r /etc/inittab' passwd
```

**（4）w**

```shell
# 1. 将passwd文件匹配到/bin/bash的行追加到/tmp/sed.txt文件中 
> sed -i '/\/bin\/bash/w tmp/sed.txt' passwd 
# 2. 将passwd文件从第10行开始，到匹配到hdfs开头的所有行内容追加到/tmp/sed-1.txt 
> sed -i '10,/^hdfs/w /tmp/sed-1.txt' passwd
```

### 删除

 **d   删除**

**1d**

**5,10d**

**10,+10d**

**/pattern1/d**

**/pattern1/,/pattern2/d**

**/pattern1/,20d**

**15,/pattern1/d**

```shell
# 1. 删除passwd中的第15行 
> sed -i '15d' passswd 
# 2. 删除passwd中的第8行到14行的所有内容 
> sed -i '8,14d' passwd 
# 3. 删除passwd中的不能登陆的用户 
> sed -i '/\/sbin\/nologin/d' passwd 
# 4. 删除passwd中以mail开头的行，到以yarn开头的行的所有内容 
> sed -i '/^mail/,/^yarn/d' passwd 
# 5. 删除passwd中第一个不能登陆的用户，到13行的所有内容 
> sed -i '/\/sbin\/nologin/,13d' passwd 
# 6. 删除passwd中第5行到以ftp开头的所有行的内容 
> sed -i '5,/^ftp/d' passwd 
# 7. 删除passwd中以yarn形头的行到最后行的所有内容 
> sed -i '/^yarn/,$d' passwd 
# 删除配置文件中的所有注释行和空行 
> sed -i '/[:blank:]*#/d;/^$/d' nginx.conf 
# 在配置文件中所有不以#开头的行前面添加*号，注意：以#开头的行不添加 
> sed -i 's/^[^#]/\*&/g' nginx.conf
```

### 修改

​    **s/old/new      # 将行内第一个old替换为new**

​    **s/old/new/g        # 将行内全部的old替换为new** 

​    **s/old/new/2        # 替换前2个old为new**

​    **s/old/new/2g      # 从第2个开始替换**

​    **s/old/new/ig       # 将行内oid全部替换为new, 忽略大小写**

修改用户总结：

**（1）1s/old/new/**

**（2）5,10s/old/new/**

**（3）10,+10s/old/new/**

**（4）/pattern1/s/old/new**

**（5）/pattern1/,/pattern2/s/old/new/**

**（6）/pattern1/,20s/old/new**

**（7）15,/pattern1/s/old/new/**

```shell
# 1. 修改/passwd中第1行中第1个root为ROOT 
> sed -i '1s/root/ROOT/' passwd 
# 2. 修改passwd中第5行到第10行中所有的/sbin/nologin为/bin/bash 
> sed -i '5,10s/\/sgin\/nologin/\/bin\/bash/g' passwd 
# 3. 修改passwd中匹配到/sbin/nologin的行，将匹配到行中的login改为大写的LOGIN 
> sed -i '/\/sbin\/nologin/s/login/LOGIN/g' passwd 
# 4. 修改passwd中从匹配到以root开头的行，到匹配到行中包含mail的所有行。修改为将这些所有匹配到的行 
> sed -i '/^root/,/mail/s/gin/HADOOP/g' passwd 
# 5. 修改passwd中匹配到以root开头的行，到第15行中的所有行，修改内容为将这些行中的nologin修改为SPARK 
> sed -i '/^root/,15s/nologin/SPARK/g' passwd 
# 6. 修改passwd中从第15行开始，到匹配以yarn开头的所有行，修改内容为将这些行中的bin换成BIN 
> sed -i '15,/^yarn/s/bin/BIN/g' passwd
```

**显示行号：**

**=** 

```shell
> sed -n '/nologin/=' passwd 
```



### 反向引用

**&和\1**

&和\1都表示引用前面的pattern，使用\1表示匹配前面pattern括号里的内容

```shell
# 在file中搜索以1开头，然后跟两个任意字符，以e结尾的字符串修改为以r结尾的字符串 
> sed "s/1..e/&r/g"  file    
# 和上面实现同样的功能 
> sed "s/\(1..e\)/\1r/g" file 
```

**对脚本变量的支持**

**使用双引号**

```shell
> /bin/bash str1=root str2=nginx sed -i "s/$str1/$str2/g" test.txt 
```

**变量加单引号**

```shell
> /bin/bash str1=root str2=nginx sed -i 's/'$str1'/'$str2'/g' test.txt 
```

