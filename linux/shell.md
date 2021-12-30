# Linux系统的启动过程

linux的启动过程分为以下几个步骤：

BIOS -> MBR -> BootLoader(grub) -> Kernel -> systemd -> init -> shell



#### 查看MBR引导记录

dd  if=/dev/sda  of=mbr.bin  bs=512 count=1

hexdump -C mbr.bin  | more



#### 查看BootLoader(grub)

/boo/grub2

grub2-editenv list    # 查看内核版本



#### 查看Systemd

在centos6使用的是/usr/sbin/init

/etc/rc.d 下有很多系统初始化的脚本 

在centos7下会到/etc/systemd/system  

然后会进入/usr/lib/systemd/system



### 常用命令

```shell
# 查看进程
$ ps aux | grep "procceser"
# 查看端口
$ netstat -anpl | grep "8080"
$ lsof -i -P | grep "8080"  # 可以在mac下使用

# 只要结果的某一列
$ docker ps -a | awk '{print $3}'
```



# SHELL脚本的规则

#! /bin/bash  

“#” 单行注释 

chmod  u+rx  ./test.sh   加可读和可执行权限

### 运行：

开一个子进程   用cd命令不会对当前目录有影响

base  test.sh  需要有执行权限 

./test.sh

在当前进程运行  用cd命令会进入到脚本里的目录 

source ./test.sh

. test.sh

# 管道和重定向

重定向符号：|

重定向

输入重定向 <

read var < /path/to/a/file

输出重定向符号  

```shell
> 清空文件，输出内容到文件
>> 追加
2> 错误输出
&> 无论正确错误全部输出到文件
```

# 变量的高级用法

## 变量替换

| 替换规则                   | 说明                                         |
| -------------------------- | -------------------------------------------- |
| ${变量#匹配规则}           | 从头开始匹配，最短删除                       |
| ${变量##匹配规则}          | 从头开始匹配，最长删除                       |
| ${变量%匹配规则}           | 从尾开始匹配，最短删除                       |
| ${变量%%匹配规则}          | 从尾开始匹配，最长删除                       |
| ${变量/旧字符串/新字符串}  | 替换变量内的旧字符串为新字符串，只替换第一个 |
| ${变量//旧字符串/新字符串} | 替换变量内的旧字符串为新字符串，全部替换     |

### 举例

```shell
variable_1 = "I love you, Do you love me"
var1=${variable_1#*ov}
var2=${variable_1##*ov}
var3=${variable_1%ov*}
var4=${variable_1%%ov*}
var5=${PATH/bin/BIN}
var6=${PATH//bin/BIN}
```

### 变量测试

| 变量配置方式     | str没有配置 | str为空字符串 | str已配置且非空 |
| ---------------- | ----------- | ------------- | --------------- |
| var=${str-expr}  | var=expr    | var=          | var=$str        |
| var=${str:-expr} | var=expr    | var=expr      | var=$str        |
| var=${str+expr}  | var=        | var=expr      | var=expr        |
| var=${str:+expr} | var=        | var=          | var=expr        |
| var=${str=expr}  | var=expr    | var=          | var=$str        |
| var={str:=expr}  | var=expr    | var=expr      | var=$str        |

### 字符串处理

|       | 语法                  | 说明                         |
| ----- | --------------------- | ---------------------------- |
| 方法1 | ${#string}            | 无                           |
| 方法2 | expr length "$string" | string有空格，则必须加双引号 |

```shell
> str1="therr are many companies connected with the computer"
# str_len就是str2的长度
> str_len=${#str1}  
# 或者
> str_len=`expr length "$str1"`
```

### 获取子串在字符串中的索引位置

expr index $string $substring 

```shell
idx_str1=`expr index "$str1" "many"`
```

### 计算子串长度

expr  match  $string  substr

```shell
> sub_str_len=`expr match "$str1" "there"`
```

注意：expre match 的子串必须是字符串第一个出现的

### 抽取子串

|        | 语法                                  | 说明                                |
| ------ | ------------------------------------- | ----------------------------------- |
| 方法一 | ${string:position}                    | 从string中的position开始            |
| 方法二 | ${string:position:length}             | 从position开始，匹配长度为length    |
| 方法三 | ${string: -position}                  | 从右边开始匹配，-position前面有空格 |
| 方法四 | ${string:(position)}                  | 从左边开始匹配                      |
| 方法五 | Expr substr $string $position $length | 从position开始，匹配长度为length    |

```
> sub_str=${str1:20}
> sub_str=${str1:10:10}
# -10前面是有空格的
> sub_str=${str1: -10}  
> sub_str=${str1:(10)}
> sub_str=expr substr $str1 10 20
```

### 命令替换

方法1：`command`

方法2：$(command)

#### cut命令

```
# 以“:”分隔并取第一列
> cat /etc/passwd | cut -d ":" -f 1 
```

在使用data+%j这类操作的时候，会自动在前面补零，导致在做运算的时候出现异常：

Value too great for base (error token is "089")

解决办法：将data + %j改为data + %-j就可以了

### 类型变量

| 参数 | 说明                               |
| ---- | ---------------------------------- |
| -r   | 将变量设为只读                     |
| -i   | 将变量设为整数                     |
| -a   | 将变量定义为数组                   |
| -f   | 显示此脚本前定义过的所有函数及内容 |
| -F   | 仅显示此脚本定义过的函数名         |
| -x   | 将变量声明为环境变量               |

```shell
> num1=10
> num2=num1+20
> echo $num2   # 结果为10+20是一个字符串
> expr $num1+10  # 结果为20
> declare -i num3
> num=$num1+90   # 结果为100
```

# SHELL语法 

## 变量

#### 变量定义

```shell

# 变量“=”两侧不能出现空格
a=123
let a=10+20
a=ls # 命令名赋值给a
let c=$(ls -l /etc)  # 将命令执行结果赋值c
let c=`ls -l /etc`  # 和()的效果一样

# 字符串赋值需要使用单引号或者双引号，不然出现空格的时候会认为是两个表达式
string1="hello bash"
string2='hello "bash"'
```

### 变量的引用

```shell
a=100
$a 
${a}
echo $a
echo ${a} # 100
echo ${a}2345 # 1002345
```

### 变量的作用域

变量默认只对当前bash生效，假设我们有如下一个脚本var.sh，脚本内容如下：

```shell
#!/bin/bash
echo $var1
```

然后我们回到终端定义一个变量var1，来通过几种运行方式测试一下

```shell
# var1="hello bash"
# base ./var.sh    // 没有输出，因为bash会开一个子进程，此时var.sh里的var1是另一个子进程的变量
# ./var.sh  // 和使用bash是一样的
# . var.sh  // 输出 hello bash  这是因为在当前进行运行
# source ./var.sh  // 输出hello bash 这是因为在当前进程运行的
```

如果我们要使变量在父进程也可见，我们可以使用export

```shell
# export var1="hello bash"
# bash ./var.sh // hello bash
# ./var.sh  // hello bash
```

## 数组

### 定义数组

```shell
IPTS=( 10.0.0.1  10.0.0.2  10.0.0.3 )
```

### 显示所有元素

```shell
echo ${IPTS[@]}
```

### 显示元素个数

```shell
echo ${#IPTS[@]}
```

### 显示数组第一个元素

```shell
echo ${IPTS[0]}
```

### 输出第一个元素的长度

```shell
echo ${#array[0]} # 输出第一个元素的长度
```



## 转义和引用

### 特殊字符

```
# ； \ " '
```

### 转义符号

```
\n \r \t
\$ \" \\ 
```

### 引用 

```
" 双引号  不完全引用  如果里面有变量会被解析
' 单引号  完全引用 原样输出
` 反引号  执行命令
```



## 运算符

```
# 赋值
=
# 运算
expr 4 + 5   # 符号和数字之前有空格
+ - * / **(乘方) %

num1=`expr 10 + 20`
num2=`expr a + b`
```

### expr 只支持整形

```
let "变量名=变量值"
变量名使用0开头为八进制
变量值使用0x开头为十六进制

let的简化：
((a=10))
((a=10+5))
((a++))
echo $((10+20))
```



## TEST判断与比较

在我们自己的脚本里面我们可以使用exit来约定一个退出的编号判断是否成功与失败，比如：

```shell
#！/bin/bash
# 1.sh
pwd
exit 0
```

```shell
./1.sh  
echo $?  # 0
```

在bash中，test内置了很多的判断，具体的内容我们可以使用man  test来查看

```shell
man test
```

test的使用方式有三种

```shell
test 1 -gt 2
echo $?
```

```shell
[ 1 -gt 2 ]   # 注意空格
echo $?   # 1
```

```shell
[[ 1 < 2 ]]   # 注意空格
echo $?   # 0
```



## IF语句

```shell
if [ express ] # 返回值==0
then  
   # dome thne code
elif [ express ]
then 
   # elif code
else  
   # some else code
fi 
```

```shell
if [ $UID=0 ]
then 
	echo "hello"
if 

# hello
```

```shell
if pwd 
then 
	echo "this is mine"
fi
```

## CASE语句

```
case "$变量"  in

	"case1" )

		命令..  ;;

    "case2" )

		命令.. ;;
		* )
		命令.. ;;
esac
 
```

### 示例程序：

```shell
#!/bin/bash
case "$1" in 
	"start"|"START")   # start 或者 START
			echo $0 start...
	;;
	"stop")
			echo $0 stop...
	;;
	"restart"|"reload")
			echo $0 restart...
	;;
	*)
			echo "Usage: $0 {start|stop|restart|reload}"
	;;
esac
```



## 循环语句

```
for  参数  in  列表

do  // some code

done
```

### 示例1

```shell
#!/bin/bash
for i in {1..9}
do
	echo i
done
```

### 示例2

```shell
#!/bin/bash
for filename in `ls *.mp3`
do
	# basename filename  ext  表示获取文件不带扩展名的文件名
	mv $filename $(basename $filename .mp3).mp4  
done
```

### 示例3

```shell
#!/bin/bash
for sc_name in /etc/profile.d/*.sh
do 
  # 判断是否有执行权限
	if [ -x $sc_name ]; then
	  # 执行sh脚本
		. $sc_name
	fi
done
```

### C语言风格的for

```
for ((变量初始化;循环判断;变量变化))
do
	// some code
done
```

### while循环

```
while  [test condition]  // 条件成立
do 
	// some command
done
```

### 示例1

```shell
#!/bin/bash
# 死循环
while :   
do
done 
```

### untils循环 

```shell
untils [test condition]   # 条件不成功
do
	# some command
done 
```

## Break&continue

```shell
#!/bin/bash
for num in {1..9}
do 
	if [ $num -eq 5 ]; then
		break
	fi
	echo num
done
```

```shell
#!/bin/bash
for num in {1..9}
do 
	if [ $num -eq 5 ]; then
		continue
	fi
	echo num
done
```

## 循环读取位置参数(命令行参数)

$0   脚本名

$*和$@代表所有位置参数

$#代表位置参数的数量

```shell
#!/bin/bash
for pos in $*
do 
	if [ "$pos" = "help" ]; then
		echo $pos $pos  # 显示两次
	fi
done
```

```shell
#!/bin/bash
# 参数个数大于等于1的时候循环
while [ $# -ge 1 ]
do 
	echo "do somethine"
	# shift 表示将参数从第一个开始调一次删除一个
	shift
done
```

## 函数

## 自定义函数

### 定义

```
function fname(){

}
```

### 函数参数

```shell
#!/bin/bash
# function 可以省略
cdls(){
	# 表示取第一个参数
	cd $1
	ls 
}
```

### 局部变量

```shell
#!/bin/bash
checkpid(){
	# 局部变量
	local i
	for i in $*; do 
	  # 如果存在对应进程号的目录返回0
		[ -d "proc/$i" ] && return 0
	done
	return 1
}
```

函数封装在shell脚本文件里的时候，我们要使用. Filename.sh 或者source ./filename.sh的方法加载

```shell
# source ./checkpid.sh
# checkpid 848
# echo $?
```

### 执行

```
fname
fname param1 param2
```



### 系统函数

/etc/init.d/functions

系统函数的使用

```shell
# source /etc/init.d/functions
# echo_success 
```



## 数学运算之BC（浮点数运算）

shell本身并不支持浮点数的运算，但可以借助bc工具来使用

```shell
> bc > scale=2  # 表示保留两位小数 
> 3/4   
```

可以通过命令行的方式一次性搞定

```shell
> echo "scale=4;23.3+35" | bc
```

还可以接收返回值

```shell
> num3=`echo "scale=4;23.3/3" | bc
```



## 脚本资源控制

ulimit -a  当前终端资源限制查看

### fork炸弹

```shell
# .(){.|.&};.
# func() { func | func& } ; func
```

### 捕获信息

```shell
#!/bin/bash
# 捕获15号信号
# catch.sh
trap "echo sig 15" 15
# 当前进程号
echo $$
while :
do
	:
done
```

iterm1

```shell
# ./catch.sh   // 假设进程号为 1024
```

item2

```shell
# kill -15 1024
```

此时iterm1的catch.sh就会捕获到15号信息

# 计划任务

### 一次性计划任务

```
at  18:31  

echo   hello  > /emp/hello.txt

<EOT>
```

```
atq  查看一次任务
```

### 周期计划任务

crontab -e 

crontab -l

分钟  小时  日期  月份  星期  执行的命令

/var/log/cron   运行日志

/etc/cron.d/0hourly    # 如果在一小时内执行过就不再执行

```shell
# Run the hourly jobs
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
01 * * * * root run-parts /etc/cron.hourly
```

/etc/anacrontab

```shell
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22

#period in days   delay in minutes   job-identifier   command
1       5       cron.daily              nice run-parts /etc/cron.daily
7       25      cron.weekly             nice run-parts /etc/cron.weekly
@monthly 45     cron.monthly            nice run-parts /etc/cron.monthly
```

/etc/cron.daily/logrotate     # 日志拆分

### 计划任务锁

flock  -xn  "/tmp/f.lock"  -c "/root/a.sh"

```shell
# flock -xn "/tmp/f.lock" -c "/root/a.sh"
```

此时，如果另外的进程使用flock去运行的时候shell就没有机会执行



# 特殊字符&关键字

## 引号

```
'  完全引用，内容原样输出
"  不完全引用，会解析变量
`  执行命令
```

## 括号

```
() (()) $() 
   单独使用圆括号会产生一个子shell(xyz=123)
   数组初始化IPS=(ip1  ip2 ip3)
   
[] [[]]
	 单独使用是测试或获取数组元素 [ a -gt 2 ] 或者 ${a[0]}
	 两个方括号表示测试表达式 if [[ a < 1 ]]
	
<> 
   重定向符号
  
{} 
	 输出范围   for num in {1..9}
	 文件复制 cp /etc/passwd{,.bak}

+ - * / % 
	 算术运算

><= 
	 比较运算符
	 
&& || ！ 
	 逻辑运算符
	 
# 注释
; 命令分隔
  case 语句的分隔符要转义;;
. 和source命令相同
~ 家目录 
, 分隔目录
* 通配符
？条件测试 或 通配符
| 管道符
& 后台运行
_ 空格
```

# 环境变量

```shell
$path
$user
$uid
$?  // 是一次命令是否正确执行 0 成功  1 失败
$$  // 当前进程PID
$0  // 当前运行程序名
```

### Shell脚本的参数

```shell
$1 $2 $3 $4 $5 $6 $7 $8 $9 // 分别代表第1~9个参数  
${10} // 表示第10个参数
// 给参数加默认值
${1}a  // 如果第一个参数没传则默认为a,如果传了参数比如：-l那最终就是-la
// 给参数加默认值，不追加到已传参数的后面
${1-a} // 如果参数传了-l最终是-l,如果没传最终是a
```

### 环境变量配置文件

/etc/profile    

/etc/profile.d/

/etc/bashrc

~/.bash_profile

~/.bashrc



su  - root   # login shell 会加载全部环境变量配置文件，顺序如下：

/etc/profile

~/.bash_profile

~/.bashrc

/etc/bashrc

su  root  // no login shell  加载的环境变量配置文件如下

 ~/.bashrc

/etc/bashrc

# 实例

## Nginx守护进程

```shell
#/bin/bash 
# 获取当前进程PID，如果脚本名包含了nginx关键字会被greg出来，这里获取PID以便过滤 
this_pid=$$ 
while true  
do    
	# 这里只需要命令执行成功与否，不需要输出    
	ps -ef | grep nginx | grep -v grep | grep -v $this_pid &> /dev/null    
	if [ $? -eq 0 ];then        
		echo "Nginx status is well"        
		sleep 2   
else        
	/usr/local/nginx/sbin/nginx       
	echo "Nginx is down,Start is..."  
done
```

**后台运行脚本：**

```shell
> nohup sh demo_nginx.sh  & # 可以通过tail 查看日志  > tail -f nohup.log
```

## 将文本数据导入MySQL

```shell
#!/bin/bash

user="dbuser"
password="123456"
host="127.0.0.1"

mysql_conn="mysql -u"$user" -p"$password" -h"$host""

cat data.txt | while read id name birth sex
do
	if [ $id -gt 1014 ];then
		$mysql_conn -e "INSERT INTO school.student1 values('$id','$name','$birth','$sex')"
	fi
done
```

文本格式如下：

```
1010	jerry	1991-12-13	male
1011	mike	1991-12-13	female
1012	tracy	1991-12-13	male
1013	kobe	1991-12-13	male
1014	allen	1991-12-13	female
1015	curry	1991-12-13	male
1016	tom	1991-12-13	female
```

## MySQL备份

```shell
#!/bin/bash

db_user="dbuser"
db_password="123456"
db_host="192.168.184.132"

ftp_user="ftp_user"
ftp_password="redhat"
ftp_host="192.168.184.3"

dst_dir="/data/backup"
time_date="`date +%Y%m%d%H%M%S`"
file_name="school_score_${time_date}.sql"

function auto_ftp
{
	ftp -niv << EOF
		open $ftp_host
		user $ftp_user $ftp_password

		cd $dst_dir
		put $1
		bye
EOF
}

mysqldump -u"$db_user" -p"$db_password" -h"$db_host" school score > ./$file_name && auto_ftp ./$file_name
```

## 获取指定应该状态信息

app_status.sh

```shell
#!/bin/bash
#
# Func: Get Porcess Status In process.cfg

# Define Variables
HOME_DIR="/root/"
CONFIG_FILE="process.cfg"
this_pid=$$


function get_all_group
{
	G_LIST=`sed -n '/\[GROUP_LIST\]/,/\[.*\]/p' $HOME_DIR/$CONFIG_FILE | egrep -v "(^$|\[.*\])"`
	echo "$G_LIST"
}

function get_all_process
{
	for g in `get_all_group`
	do
		P_LIST=`sed -n "/\[$g\]/,/\[.*\]/p" $HOME_DIR/$CONFIG_FILE | egrep -v "(^$|\[.*\])"`
		echo "$P_LIST"
	done
}

function get_process_pid_by_name
{
	if [ $# -ne 1 ];then
		return 1
	else
		pids=`ps -ef | grep $1 | grep -v grep  | grep -v $0 | awk '{print $2}'`
		echo $pids
	fi
}

function get_process_info_by_pid
{
	if [ `ps -ef | awk -v pid=$1 '$2==pid{print }' | wc -l` -eq 1 ];then
		pro_status="RUNNING"
	else
		pro_status="STOPED"
	fi
	pro_cpu=`ps aux | awk -v pid=$1 '$2==pid{print $3}'`
	pro_mem=`ps aux | awk -v pid=$1 '$2==pid{print $4}'`
	pro_start_time=`ps -p $1 -o lstart | grep -v STARTED`
}

function is_group_in_config
{
	for gn in `get_all_group`;do
		if [ $gn == $1 ];then
			return
		fi
	done
	echo "Group $1 is not in process.cfg"
	return 1
}

function is_process_in_config
{
	for pn in `get_all_process`;do
		if [ $pn == $1 ];then
			return
		fi
	done
	echo "Process $1 is not in process.cfg"
	return 1
}

function get_all_process_by_group
{
	is_group_in_config $1
	if [ $? -eq 0 ];then
		p_list=`sed -n "/\[$1\]/,/\[.*\]/p" $HOME_DIR/$CONFIG_FILE | egrep -v "(^$|^#|\[.*\])"`
		echo $p_list
	else
		 echo "GroupName $1 is not in process.cfg"
	fi	
}	

function get_group_by_process_name
{
	for gn in `get_all_group`;do
		for pn in `get_all_process_by_group $gn`;do
			if [ $pn == $1 ];then
				echo "$gn"
			fi
		done
	done
}

function format_print
{
	ps -ef | grep $1 | grep -v grep | grep -v $this_pid &> /dev/null
	if [ $? -eq 0 ];then
		pids=`get_process_pid_by_name $1`
		for pid in $pids;do
			get_process_info_by_pid $pid
			awk -v p_name=$1 \
				-v g_name=$2 \
				-v p_status=$pro_status \
				-v p_pid=$pid	\
				-v p_cpu=$pro_cpu \
				-v p_mem=$pro_mem \
				-v p_start_time="$pro_start_time" \
				'BEGIN{printf "%-20s%-12s%-10s%-6s%-7s%-10s%-20s\n",p_name,g_name,p_status,p_pid,p_cpu,p_mem,p_start_time}'
		done
	else
		awk -v p_name=$1 -v g_name=$2 'BEGIN{printf "%-20s%-12s%-10s%-6s%-7s%-10s%-20s\n",p_name,g_name,"STOPPED","NULL","NULL","NULL","NULL"}'
	fi
}

awk 'BEGIN{printf "%-20s%-10s%-10s%-6s%-7s%-10s%-20s\n","ProcessName---------","GroupName---","Status----","PID---","CPU----","MEMORY----","StartTime---"}'

if [ $# -gt 0 ];then
	if [ "$1" == "-g" ];then
		shift
		for gn in $@;do
			is_group_in_config $gn || continue
			for pn in `get_all_process_by_group $gn`;do
				is_process_in_config $pn && format_print $pn $gn
			done
		done 
	else
		for pn in $@;do
			gn=`get_group_by_process_name $pn`
			is_process_in_config $pn && format_print $pn $gn
		done
	fi
else
	for pn in `get_all_process`;do
		gn=`get_group_by_process_name $pn`
                is_process_in_config $pn && format_print $pn $gn
	done
fi 

```

process.cfg

```
[GROUP_LIST]
WEB
DB
HADOOP
YARN

[WEB]
nginx
httpd

[DB]
mysql
postgresql
oracle

[HADOOP]
datanode
namenode
journalnode

[YARN]
resourcemanager
nodemanager

[nginx]
description="Web Server 1"
program_name=tail
parameter=-f /root/lesson/9.1/tmp/web-nginx.conf



[httpd]
description="Web Server 2"
program_name=tail
parameter=-f /root/lesson/9.1/tmp/web-httpd.conf



[mysql]
description="High Performance DataBase"
program_name=tail
parameter=-f /root/lesson/9.1/tmp/mysql.conf 



[postgresql]
description="PG Server"
program_name=tail
parameter=-f /root/lesson/9.1/tmp/postgresql.conf



[oracle]
description="The Best DB Server In The World"
program_name=tail
parameter=-f /root/lesson/9.1/tmp/oracle.conf



[datanode]
description="NODE: Storage Data For HDFS"
program_name=tail
parameter=-f /root/lesson/9.1/tmp/hdfs-datanode.xml



[namenode]
description="NODE: Storage MetaData For HDFS"
program_name=tail
parameter=-f /root/lesson/9.1/tmp/hdfs-namenode.xml



[journalnode]
description="Data synchronization For NameNode"
program_name=tail
parameter=-f /root/lesson/9.1/tmp/hdfs-journalnode.xml



[resourcemanager]
description="Resource allocation, task scheduling"
program_name=tail
parameter=-f /root/lesson/9.1/tmp/yarn-resourcemanager.xml


[nodemanager]
description="Compute nodes to perform tasks assigned by RM"
program_name=tail
parameter=-f /root/lesson/9.1/tmp/yarn-nodemanager.xml
```

# 常用命令集合

```shell
wc -l  // 统计接下来输入的行数
wc -l < test.txt   // 统计test.txt文件的行数

清除一个变量
unset var1

du -h  // 查看当前目录占用空间

$PS1  // 修改终端样式

# 以“:”分隔并取第一列
> cat /etc/passwd | cut -d ":" -f 1 
```

