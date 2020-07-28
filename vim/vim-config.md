
```
// 关闭备份
set nobackupset 
// 撤销文件自定义目录，防止在当前目录下产生不必要的文件
undodir=~/.vim/undodir  
// 撤销文件目录不存在需要创建
if !isdirectory(&undodir) 
    call mkdir(&undodir, 'p', 0700)
endif


if has('mouse') 
    if has('gui_running') || (&term =~ 'xterm' && !has('mac')) 
        set mouse=a 
    else 
        set mouse=nvi 
    endif
endif
```