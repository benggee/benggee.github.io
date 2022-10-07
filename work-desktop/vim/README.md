# vim完全搞定指南

## 各个系统的快捷键配置

**将将CapsLock配置成Esc键**

Mac下 

```
系统偏好 > 键盘 > 修饰键 > Capslock或者中/英文：Escape/Control
```

Windows下

将下面代码保存为 capslock2esc.reg：

```
Windows Registry Editor Version 5.00
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout]
"Scancode Map"=hex:00,00,00,00,00,00,00,00,02,00,00,00,01,00,3a,00,00,00,00,00
```

注：上面不是互换，如果要互换，则用下面代码：

```
Windows Registry Editor Version 5.00
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout]
"Scancode Map"=hex:00,00,00,00,00,00,00,00,03,00,00,00,3a,00,01,00,01,00,3a,00,00,00,00,00
```



## 1. vim模式

![image.gif](/_media/vim-master.gif)

![image.gif](/_media/vim-edit.gif)

![image.gif](/_media/vim-operator.gif)

![image.gif](/_media/vim-yank.gif)

![image.gif](/_media/vim-searching.gif)

![image.gif](/_media/vim-marks.gif)

![image.gif](/_media/vim-various-motions.gif)

![image.gif](/_media/vim-various-commands.gif)



## 2. 移动

| 指令               | 作用                                                         |
| ------------------ | ------------------------------------------------------------ |
| h / j / k /        | 左 / 下 / 上 / 右                                            |
| Shift+$            | 行尾                                                         |
| Shift+0  / Shift+^ | 行首，^移动到行首第一个非空字符                              |
| b / w  、 B / W    | 向后 / 向前移动一个单词，小写认为单词由数字、字母、下划线组成，大写认为空格相格的都是单词 |
| f / t  、 F / T    | 找到命令下一个输入字符，f包含这个字符，t不包含。大写是反向查找 |
| ( / )              | 上一句 / 下一句                                              |
| { / }              | 上一段  /  下一段                                            |
| gg  /  G           | 行首 /  最后一行                                             |



## 3.  修改

| 指令   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| dd     | 删除一行，光标跳到下一行开头                                 |
| D      | 删除一行，光标位置不变                                       |
| cc / C | 删除一行，进入插入模式，cc删除光标所在行，C删除光标所在行光标以后的所有内容 |
| s      | 替换一个字符，删除光标后一个字符，进入插入模式               |
| S      | 替换一整行，相当于cc                                         |
| i      | 在当前字符前进入插入模式                                     |
| I      | 大写i，光标移动到第一个非空字符前，进入插入模式              |
| a      | 在当前字符后进入插入模式                                     |
| A      | 移动到行尾进入插入模式                                       |
| o      | 在当前行下插入新行                                           |
| O      | 在当前行上插入新行                                           |
| r      | 替换光标下的字符                                             |
| R      | 进入替换模式，每按一下字符替换一个                           |
| u      | 撤销最近一次修改                                             |
| U      | 撤销当前行上的所有修改                                       |



## 4. 选择/删除

| 指令   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| dw     | 删除一个词，从当前位置往后                                   |
| diw    | 删除光标所在词                                               |
| daw    | 删除光标所在词和后面的空格                                   |
| diW    | 删除光标所在词，及相邻符号                                   |
| daW    | 删除光标所在词， 及相邻符号和后面的空格                      |
| di"    | 删除" "内的所有内容，还可以搭配 '' / () / {} / [] / <> / `` / t (xml/html标签) / s（句子） / p（段落） |
| da"    | 删除" "内的所有内容及双引号，还可以搭配 '' / () / {} / [] / <> / `` / t (xml/html标签) / s（句子） / p（段落） |
| 数字+G | 跳转到对应行                                                 |
| yy     | 复制一行                                                     |
| 4yy    | 复制4行                                                      |
| p      | 粘贴复制                                                     |
| yw     | 复制一个单词(包括后面的空白字符)                             |
| ye     | 复制一个单词(不包括单词后面的空白字符)                       |
| yl     | 复制当前光标下的字符                                         |
| yh     | 复制光标前面的一个字符                                       |
| 4yl    | 复制当前光标下的字符、以及后面三个字符                       |
| 4yh    | 复制当前光标前的四个字符(不包含当前光标所在字符)             |



## 5. 屏幕移动

| 指令         | 说明                                 |
| ------------ | ------------------------------------ |
| H            | 移动到屏幕顶部                       |
| M            | 移动到屏幕中部                       |
| L            | 移动到屏幕底部                       |
| zt / zz / zb | 将当前行移动到屏幕顶部 / 中部 / 底部 |



## 6. 重复操作

| 指令 | 说明                              |
| ---- | --------------------------------- |
| ;    | 重复最近的字符查找                |
| ,    | 重复最近的字符查找，反向          |
| n    | 重复最近的字符查找( / 和 ?)       |
| N    | 重复最近的字符查找(/ 和 ?) , 反向 |
| .    | 重复最近的修改操作                |



## 7. 块操作

**块选择**

1. 按v键
2. 使用y键复制、x剪切、d删除

**纵向选择**

1. Ctrl + v
2. h / j / k / l  选择
3. Shift + i 插入 或者del删除
4. Esc 
5. :w    

## 8. 多文件



## 9. 调试/跳转



## 10. 编程语言支持



## 11. 命令行快捷操作

| 指令     | 说明                       |
| -------- | -------------------------- |
| Ctrl + l | 清屏                       |
| Ctrl + a | 移动光标到行首             |
| Ctrl + e | 移动光标到行尾             |
| Ctrl + h | 往后删一个字符             |
| Ctrl + d | 往前删除一个字符           |
| Ctrl + b | 往后移动一个字符           |
| Ctrl + f | 往前移动一个字符           |
| Ctrl + w | 剪切前一个单词（空格间隔） |
| Ctrl + u | 剪切到行首                 |
| Ctrl + k | 剪切到行尾                 |
| Ctrl + y | 粘贴剪切                   |
| Ctrl + r | 查找历史执行命令           |
| Ctrl + p | 前一条指令                 |
| Ctrl + n | 后一条指令                 |



## 12. 配置

```shell
set nocompatible
set backspace=2
syntax on
set tabstop=4
set autoindent
set nu!
set nobackup
set scrolloff=1
set undodir=~/.vim/undodir

if !isdirectory(&undodir)
	call mkdir(&undodir, 'p', 0700)
endif


let g:go_version_warning=0

if has('mouse')
	if has('gui_running') || (&term =~ 'xterm' && !has('mac'))
		set mouse=a
	else
		set mouse=nvi
	endif
endif

if exists('g:loaded_minpac')
  " Minpac is loaded.
    call minpac#init()
	call minpac#add('k-takata/minpac', {'type': 'opt'})
	
	" Other plugins
	call minpac#add('tpope/vim-eunuch')
	call minpac#add('yegappan/mru')
	call minpac#add('preservim/nerdtree')
endif

if has('eval')
	" Minpac commands
	command! PackUpdate packadd minpac | source $MYVIMRC | call minpac#update()
	command! PackClean  packadd minpac | source $MYVIMRC | call minpac#clean()
	command! PackStatus packadd minpac | source $MYVIMRC | call minpac#status()
endif


if !has('gui_running')
  " 设置文本菜单
    if has('wildmenu')
	    set wildmenu
		set cpoptions-=<
		set wildcharm=<C-Z>
		nnoremap <F10>      :emenu <C-Z>
		inoremap <F10> <C-O>:emenu <C-Z>
	endif
endif

if !has('patch-8.0.210')
	" 进入插入模式时启用括号粘贴模式
	let &t_SI .= "\<Esc>[?2004h"
	" 退出插入模式时停用括号粘贴模式
	let &t_EI .= "\<Esc>[?2004l"
	" 见到<Esc>[200~就调用XTermPasteBegin
	inoremap <special> <expr> <Esc>[200~ XTermPasteBegin()

	function! XTermPasteBegin()
		" 设置使用<Esc>[201~ 关闭粘贴模式
		set pastetoggle=<Esc>[201~
		" 开启粘贴模式
		set paste
		return ""
	endfunction
endif
```
