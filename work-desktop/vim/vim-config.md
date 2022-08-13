# Vim配置

```
set enc=utf-8
set tabstop=4 
// 不兼容vi
set nocompatible 
// 加载vim默认配置
source $VIMRUNTIME/vimrc_example.vim

// 关闭备份
set nobackup
// 撤销文件自定义目录，防止在当前目录下产生不必要的文件
set undodir=~/.vim/undodir  
// 撤销文件目录不存在需要创建
if !isdirectory(&undodir) 
    call mkdir(&undodir, 'p', 0700)
endif

// 表示满足以下条件时响应鼠标事件
// 1.图形界面正在运行
// 2.终端是xterm兼容，并且不是Mac
if has('mouse') 
    if has('gui_running') || (&term =~ 'xterm' && !has('mac')) 
        set mouse=a 
    else 
        set mouse=nvi 
    endif
endif
```

