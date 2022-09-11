# AWK原理与使用



### AWK执行流程

- 主循环前置操作BEGIN{}
- 主输入循环{}
- 所有文件完成执行END{}



BEGIN{} 可以设置在主循环{}之前设置环境变量等操作，比如设置分隔符：

```
// 将分隔符设置成":"
$ awk 'BEGIN{FS=":"}{print $1}' /etc/passwd
```

主循环{}从文件或者管道循环一行一行读取内容，{}内的指令是针对这一行处理，比如输出每行第二列：

```
$ docker images | awk '{print $3}'
IMAGE
ff3b5098b416
5d0da3dc9764
```

END{}在主循环{}处理完之后执行



### AWK核心概念

- 每一行称作AWK的记录
- 使用空格、制表符分隔开的单词称作字段
- 可以自己指定分隔的字段

指定自己的分隔字段，比如：

```
$ head -5 /etc/passwd | awk -F ":" '{print $1}'
$ awk -F "'" '/^menu/{ print x++,$2 }' /boot/grub2/grub.cfg  // 每行menu开头，'分隔，x每行行号累加
```

其中，-F表示重置分隔符为":"



### 字段引用

- awk '{print $1,$2,$3,$n}' filename
- awk -F ":" '{print $1, $2, $3}' filename



### 赋值操作符

```
var1 = "name"
var2 = "hello" "world"   // 最终 helloworld
```

++/--/+=/-=/*=//=/%=/^=



#### 算术操作符

- +、-、*、/、%、^



#### 关系操作符

- <、 >、 <=、 >=、 ==、 !=、 ~、 !~



#### 系统变量

- FS、OFS，FS表示输入分隔符，OFS表示输出分隔符

  ```
  $ head -5 /etc/passwd | awk 'BEGIN{FS=":";OFS="-"} {print $1, $2}' 
  ```

- RS记录分隔符，指定行分隔符

  ```
  $ head -5 /etc/passwd | awk 'BEGIN{RS=":"}{print $0}'
  ```

- NR和FNR，表示行数

  ```
  $ head -5 /etc/passwd | awk '{print NR, $0}'  // NR显示行号
  $ awk '{print FNR, $1}' /etc/hosts  /etc/hosts  // 多个文件FNR每个文件从头记数
  ```

- NF字段数量，最后一个字段内容可以用$NF取出

  ```
  $ head -i kpi.txt | awk '{for(c2;c<=NF;c++) sum = sum+$c }' // 累加 NF表示每行字段数量
  $ head -5 /etc/passwd | awk 'BEGIN{FS=":"}{print NF}'    // NF表示字段数量
  $ head -5 /etc/passwd | awk 'BEGIN{FS=":"}{print $NF}'   // 最后一个字段内容
  ```



### 关系操作符

示例文件：kpi.txt

```
user1 70 72 74 76 74 72
user2 80 82 84 82 80 78
user3 60 61 63 64 65 62
user4 90 89 88 87 86 85 
user5 45 60 63 62 61 50
```

示例：

```
$ awk '{if($2>=80) print $1}' kpi.txt // 如果第行第二个字段大于等于80，输出第一个字段
$ awk '{if($2>=80) { print $1; print $2} }'  kpi.txt // 条件语句中，多个语句用"{}"括起来
```



#### 循环  

while循环

```
while(表达式)
	awk语句
```

do循环

```
do {
	awk 语句
} while(表达式)
```

For循环

```
for (i=0;i<NF;i++)
	awk 语句
```

循环控制

- break
- Continue



for循环示例

```
$ head -1 kpi.txt | awk '{for(c2;c<=NF;c++) print $c}'    // c表示第几列，NF表示字段数，$c表示输出第c列字段的值
$ head -i kpi.txt | awk '{for(c2;c<=NF;c++) sum = sum+$c }' // 累加
$ head -i kpi.txt | awk '{for(c2;c<=NF;c++) { sum = sum+$c; print sum } }' // 累加
$ head -i kpi.txt | awk '{sum=0; for(c2;c<=NF;c++) { sum = sum+$c; print sum/(NF-1) } }' // 求平均值
```



#### 数组

- 数组名[下标] = 值

- 遍历 

  ```
  for (变量 in 数组) 
  ```

- 删除元素

  ```
  delete 数组[下标]
  ```

  

```
$ awk '{sum=0; for(c=2;c<=NF;c++) sum+=$c; average[$1]=sum/(NF-1)} END{for(user in average) print user, average[user]}'  kpi.txt

$ awk '{sum=0; for(c=2;c<=NF;c++) sum+=$c; average[$1] = sum/(NF-1)} END {for(user in average) sum2+=average[user]; print sum2/NR}' kpi.txt  // 求所有人平均值的平均值  
```

##### 使用文件保存awk脚本

vi avg.awk

awk -f avg.awk kpi.txt



#### 参数ARGC/ARGV

vi  argv.awk  

```
BEGIN {
	for (i=0;i<ARGC;i++)
		print ARGV[i]

	print ARGC

}
```



- ARGC 表示参数个数

- ARGV 表示参数数组



示例

```
{
	sum = 0
	for (c = 2; c <= NF; c++)
		sum += $c

	average[$1] = sum / (NF-1)
	
	if (average[$1] >= 80)
		letter = "S"
	else if (average[$1] >= 70)
		letter = "A"
	else if (average[$1] >= 60)
		letter = "B"
	else 
		letter = "C"


	letter_all[letter]++

}
END {
	for (user in average)
		sum_all += average[user]

	avg_all = sum_all / NR
	
	for (user in average) 
		if (average[user] > avg_all)
			above ++
		else 
			below++
	
	print "above", above
	print "below", below
	
	print "S:", letter_all["S"]
	print "A:", letter_all["A"]
	print "B:", letter_all["B"]
	print "C:", letter_all["C"]

}
```



#### 函数

- sin() 
- cos()
- int()
- rand() 0-1 伪随机
- srand()  
- awk 'BEGIN{pi=3.14; print int(pi)}'

- gsub(r, s, t)
- index(s, t)
- length(s)
- match(s, r)
- split(s, a, sep)
- sub(r, s, t)
- substr(s, p, n)

#### 自定义函数

function 函数名 （参数） {
	awk 语句
	return awk变量
}

```
awk 'function a(){return 0} BEGIN{ print a() }'   // 函数要放在主输入循环前面
```


