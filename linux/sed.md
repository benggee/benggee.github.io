# sed快速上手

**sed的基本工作模式：**

- 将文件以行为单位读取到内存(模式空间)
- 使用sed的每个脚本对该行进行操作
- 处理完之后输出该行



**sed的两种形式**

- stdout | sed  [option] "pattern command" 

- sed [option] "pattern command"  file



**option**

- -n     只打印模式匹配行

- -e     直接在命令行进行sed编辑, 默认选项, 可以显示-e 指定多个

- -f      编辑动作保存在文件中，指定文件执行

- -r      支持扩展正则表达式

- -i      直接修改文件内容

**pattern**

- 10command                       #匹配到第10行

- 10,20command                    #匹配从第10行开始，到第20行结束

- 10,+5command                    #匹配从第10行开始，到第16行结束

- /pattern1/command               #匹配到pattern1的行

- /pattern1/,/pattern2/command    #匹配pattern1开始的行到pattern2开始的行结束

- 10,/pattern1/command            #匹配从第10行开始到pattern1的行结束

- /pattern1/,10command            #匹配到pattern1的行开始到第10行匹配结束


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

**p 打印**

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

**追加**

- ​    a   行后追加

- ​    i    行前追加

- ​    r    外部文件读入，行后追加

- ​    w   匹配行写入外部文件 


**a  - append** 

```shell
# 1. passwd文件第10行后面追加“Add Line Behind” 
> sed -i '10a Add Line Begind' passwd 
# 2. passwd文件第10行到20行，每一行后面都追加“Test Line Behind” 
> sed -i '10,20a Test Line Behind' passwd 
# 3. passwd文件匹配到/bin/bash的行后面追加“Insert Line For /bin/bash Behind” 
> sed -i "/\/bin\/bash/a Insert Line For /bin/bash Behind" passwd
```

**i   - insert**  

```shell
# 1. passwd文件匹配到以yarn开头的行，在匹配行前面加上“Add Line Before” 
> sed -i '/^yarn/i Add Line Before' passwd 
# 2. passwd 文件每一行前面都追加“Insert Line Before Every Line” 
> sed -i 'i Insert Line Before Every Line' passwd
```

**r**

```shell
# 1. 将fstab文件的内容追加到passwd文件的第20行后面 
> sed -i '20r /etc/fstab' passwd 
# 2. 将inittab文件内容追加到passwd文件匹配/bin/bash行的后面 
> sed -i '/\/bin\/bash/r /etc/inittab' passwd
```

**w**

```shell
# 1. 将passwd文件匹配到/bin/bash的行追加到/tmp/sed.txt文件中 
> sed -i '/\/bin\/bash/w tmp/sed.txt' passwd 
# 2. 将passwd文件从第10行开始，到匹配到hdfs开头的所有行内容追加到/tmp/sed-1.txt 
> sed -i '10,/^hdfs/w /tmp/sed-1.txt' passwd
```

**删除**

-  d   删除

  - 1d

  - 5,10d

  - 10,+10d
  - /pattern1/d

  - /pattern1/,/pattern2/d

  - /pattern1/,20d

  - 15,/pattern1/d

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



**替换**

- ​    s/old/new           # 将行内第一个old替换为new

- ​    s/old/new/g        # 将行内全部的old替换为new（全局替换）

- ​    s/old/new/2        # 替换前2个old为new

- ​    s/old/new/2g      # 从第2个开始替换

- ​    s/old/new/ig       # 将行内oid全部替换为new, 忽略大小写**


修改用法总结：

1. 1s/old/new/
2. 5,10s/old/new/
3. 10,+10s/old/new/
4. /pattern1/s/old/new

5. /pattern1/,/pattern2/s/old/new/

6. /pattern1/,20s/old/new

7. 15,/pattern1/s/old/new/

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



**正则引用分组**

```
// cfile 内容：axyzb
sed -r 's/(a.*b)/\1/' cfile      // 结果 axyzb   \1表示引用第一个分组

sed -r 's/(a.*b)/\1\1/' cfile    // 结果 axyzbaxyzb

sed -r 's/(a.*b)/\1:\1/' cfile   // 结果 axyzb:axyzb
```



**显示行号：**

**=** 

```shell
> sed -n '/nologin/=' passwd 
```



**反向引用**

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



**其它命令**

退出命令

```
sed  10q filename        // 只打印10行
sed  -n 1,10p filename   // 会匹配所有行，只是不打印
```



## 分组

**寻址可以匹配多条命令**

```
/regular/{s/old/new/;s/old/new/}
```



## 脚本文件

```
sed -f sedscript filename
```



## 多行模式

**多行模式场景**

例如XML/JSON格式的配置文件

a.txt内容如下：

```
hel
lo
```

多行示例

```
sed 'N;s/hel\nlo/!!!/' a.txt    
sed 'N;s/hel.lo/!!!/' a.txt
```



**多行模式命令**

- N  将下一行加入到模式空间
- D   删除模式空间中的第一个字符到第一个换行符
- P   打印模式空间中的第一个字符到第一个换行符



## 保持空间

**什么是保持空间**

保持空间是一种多行操作模式，它会在模式空间之外新创建一块内存空间，命令如下：

- h和H 将模式空间内容存放到保持空间
- g 和G 将保持空间内容取出到模式空间
- x 交换模式空间和保持空间内容

```
// 第一行放到保持空间，除了第一行之外从保持空间读入到模式空间，X交换，最后一行不交换，p输出
cat -n /etc/passwd | head -6 | sed -n '1h;1!G;$1x;$p'

cat -n /etc/passwd | head -6 | sed -n "G;h;$p"

cat -n /etc/passwd | head -6 | sed -n "1!G;h;$p"

cat -n /etc/passwd | head -6 | sed '1!G;h;$!d'
```



### TIPS

生成多行文本

```
seq 1 10000 > lines.txt
```

