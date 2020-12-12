规则

#! /bin/bash  

“#” 单行注释 

chmod  u+rx  ./test.sh   加可读和可执行权限

运行：

开一个子进程   用cd命令不会对当前目录有影响

base  test.sh  需要有执行权限 

./test.sh

在当前进程运行  用cd命令会进入到脚本里的目录 

source ./test.sh

. test.sh



管道和重定向

重定向符号：|

重定向

输入重定向 <

read var < /path/to/a/file

输出重定向符号  

```
> 清空文件，输出内容到文件
>> 追加
2> 错误输出
&> 无论正确错误全部输出到文件
```



SHELL语法 

变量

变量定义

```

# 变量“=”两侧不能出现空格
a=123
let a=10+20
a=ls // 命令名赋值给a
let c=$(ls -l /etc)  // 将命令执行结果赋值c
let c=`ls -l /etc`  // 和()的效果一样

# 字符串赋值需要使用单引号或者双引号，不然出现空格的时候会认为是两个表达式
string1="hello bash"
string2='hello "bash"'
```

变量的引用

```
a=100
$a 
${a}
echo $a
echo ${a} // 100
echo ${a}2345 // 1002345
```

变量的作用域

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



数组

定义数组

```
IPTS=( 10.0.0.1  10.0.0.2  10.0.0.3 )
```

显示所有元素

```
echo ${IPTS[@]}
```

显示元素个数

```
echo ${#IPTS[@]}
```

显示数组第一个元素

```
echo ${IPTS[0]}
```



转义和引用

特殊字符

```
# ； \ " '
```

转义符号

```
\n \r \t
\$ \" \\ 
```

引用 

```
" 双引号  不完全引用  如果里面有变量会被解析
' 单引号  完全引用 原样输出
` 反引号  执行命令
```



运算符

```
// 赋值
=
// 运算
expr 4 + 5   // 符号和数字之前有空格
+ - * / **(乘方) %

num1=`expr 10 + 20`
num2=`expr a + b`
```

expr 只支持整形

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

特殊符号



TEST判断与比较

在我们自己的脚本里面我们可以使用exit来约定一个退出的编号判断是否成功与失败，比如：

```
#！/bin/bash
# 1.sh
pwd
exit 0
```

```
./1.sh  
echo $?  // 0
```

在bash中，test内置了很多的判断，具体的内容我们可以使用man  test来查看

```
man test
```

test的使用方式有三种

```
test 1 -gt 2
echo $?
```

```
[ 1 -gt 2 ]   // 注意空格
echo $?   // 1
```

```
[[ 1 < 2 ]]   // 注意空格
echo $?   // 0
```



IF语句

```
if [ express ] // 返回值==0
then  
   // dome thne code
elif [ express ]
then 
   // elif code
else  
   // some else code
fi 
```



```shell
if [ $UID=0 ]
then 
	echo "hello"
if 

// hello
```



```shell
if pwd 
then 
	echo "this is mine"
fi
```

CASE语句

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

示例程序：

```
#!/bin/bash
case "$1" in 
	"start"|"START")   // start 或者 START
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



循环语句

```
for  参数  in  列表

do  // some code

done
```

示例1

```shell
#!/bin/bash
for i in {1..9}
do
	echo i
done
```

示例2

```shell
#!/bin/bash
for filename in `ls *.mp3`
do
	# basename filename  ext  表示获取文件不带扩展名的文件名
	mv $filename $(basename $filename .mp3).mp4  
done
```

示例3

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

C语言风格的for

```
for ((变量初始化;循环判断;变量变化))
do
	// some code
done
```

while循环

```
while  [test condition]  // 条件成立
do 
	// some command
done
```

示例1

```shell
#!/bin/bash
# 死循环
while :   
do
done 
```

untils循环 

```
untils [test condition]   // 条件不成功
do
	// some command
done 
```

Break&continue

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

循环读取位置参数(命令行参数)

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

函数

自定义函数

定义

```
function fname(){

}
```

函数参数

```shell
#!/bin/bash
# function 可以省略
cdls(){
	# 表示取第一个参数
	cd $1
	ls 
}
```

局部变量

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

执行

```
fname
fname param1 param2
```



系统函数

/etc/init.d/functions

系统函数的使用

```shell
# source /etc/init.d/functions
# echo_success 
```



脚本资源控制

ulimit -a  当前终端资源限制查看

fork炸弹

```shell
# .(){.|.&};.
# func() { func | func& } ; func
```



捕获信息

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

Iterm1

```shell
# ./catch.sh   // 假设进程号为 1024
```

Item2

```shell
# kill -15 1024
```

此时iterm1的catch.sh就会捕获到15号信息



计划任务

一次性计划任务

```
at  18:31  

echo   hello  > /emp/hello.txt

<EOT>
```

```
atq  查看一次任务
```





环境变量

```
$path
$user
$uid
$?  // 是一次命令是否正确执行 0 成功  1 失败
$$  // 当前进程PID
$0  // 当前运行程序名
```

Shell脚本的参数

```
$1 $2 $3 $4 $5 $6 $7 $8 $9 // 分别代表第1~9个参数  
${10} // 表示第10个参数
// 给参数加默认值
${1}a  // 如果第一个参数没传则默认为a,如果传了参数比如：-l那最终就是-la
// 给参数加默认值，不追加到已传参数的后面
${1-a} // 如果参数传了-l最终是-l,如果没传最终是a
```

环境变量配置文件

/etc/profile    

/etc/profile.d/

/etc/bashrc

~/.bash_profile

~/.bashrc



su  - root   // login shell 会加载全部环境变量配置文件，顺序如下：

/etc/profile

~/.bash_profile

~/.bashrc

/etc/bashrc



su  root  // no login shell  加载的环境变量配置文件如下

 ~/.bashrc

/etc/bashrc







命令集合

```
wc -l  // 统计接下来输入的行数
wc -l < test.txt   // 统计test.txt文件的行数

清除一个变量
unset var1

du -h  // 查看当前目录占用空间

$PS1  // 修改终端样式
```

