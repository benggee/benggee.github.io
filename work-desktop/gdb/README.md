## 1. 调试器的原理

1. 编译程序的时候带上调试信息
```
// 对于gcc来讲
gcc -g
```

2. 调试信息包含：指令地址、对应源代码及行号

3. 指令完成后，回调调试器来获取CPU寄存器的信息

使用方式

```
> gdb testsoft
```


## 2. GDB基础命令

| Command | Simplify | Action | Description |
| --- | --- | --- | --- |
| run | r | 支行程序 | 调试开始，注意在run之前要先设置断点 |
| break | b | 设置断点 | b 函数名 |
| info | i | 查看信息 | 查看断点i b，等后面详细内容 |
| delete | d | 删除断点 | d 断点编号 |
| disable | disable | 禁用断点 | disable 断点编号 |
| backtrace | bt,where | 查看栈帧 | bt N显示开头N个栈帧, bt -N最后N个栈帧 |
| print | p | 打印信息 | p argc打印变量 |
| x | x | 显示内存 | x 0xffffffff |
| set | set | 设置变量的值 | set variable <变量>=<表达式>;比如 set var test=3 |
| next | n | 单步执行 | 执行到下一步 |
| step | s | 单步执行/进入函数 | 如果下一行是函数，则进入到函数内 |
| continue | c | 执行到最后 | c 可以设置执行次数，默认表示1次 |
| finish | finish |  | 从当前函数跳出，会执行完当前函数所有逻辑 |
| until | until |  | 执行完代码块 |

### 2.1. 打印变量值

| Format | Description | 
| --- | --- |
| x | p/x var 显示16进制 | 
| d | p/d var 显示10进制 | 
| u | p/u var 显示无符号10进制 | 
| o | p/o var 显示8进制 | 
| t | p/t var 显示2进制 | 
| a | p/a var 显示地址 | 
| c | p/c var 显示字符 | 
| f | p/f var 符点数 | 
| s | p/s var 显示字符串 | 

### 2.2. 查看内存

```
(gdb) x &a 
0xfffffffffdddd: 0x000000001

(gdb) x 0xfffffffffdddd
0xfffffffffdddd: 0x000000001
```

### 2.3. 自动换行
```
(gdb) set hight 0
```

### 2.4. 打印所有线程堆栈

```
(gdb) thread apply all bt
```

### 2.5. 查看某个地址详细信息
这个似乎只针对函数有效

```
(gdb) info line 0xfffffffddddd
```

### 2.6. 查看结构体定义

```
(gdb) ptype pTimeVal
```

### 2.7. 美化格式

```
(gdb) set print pretty on
```
### 2.8. 打印数组

```
(gdb) p *pStTmp->pst@4
```

### 2.9. 显示信息

```
// display
(gdb) display var  // 显示变量信息

// info
// info args 查看当前函数参数
// info line 查看源代码在内存中地址，可以跟行号、函数名
// info locals 显示当前函数的局部变量
// info symbol 显示全局变量信息
// info function 显示所有函数名
// info thread 查看线程信息
// info registers 列举寄存器的值
```

### 2.10. 指定动态库位置

```
(gdb) set solib-search-pathc ./libso/
(gdb) set solib-absolute-prefix ./libso/
```

### 2.11. 打印当前进程map信息

```
(gdb) i proc map
```

## 3. GDB进阶

### 3.1. 多纯程调试

```
(gdb) set scheduler-locking on
(gdb) set scheduler-locking off

// 锁定某个线程
(gdb) b getTimeOfDay thread 23  // 23表示线程号
```

### 3.2. 反汇编

```
(gdb) disassemble /m
```

汇编单步调试

```
(gdb) nexti
(gdb) stepi
```

汇编指令偏移

```
(gdb) b * main + 4
```
