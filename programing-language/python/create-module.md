**一、创建一个文件夹**

/test

**二、将自己的python文件复制到文件夹**

/test/list.py

**三、创建setup.py文件加入以下内容**

```python
from  distutils.core  import  setup
setup(
	name               = 'test',
	version            = '1.0.0',
	py_modules         = ['test'],                
	author             = 'heads',
	author_email       = 'seepre@seepre.com',
	url                = 'www.seepre.com',
	description        = 'test  list',
)
```

**四、构建发布文件**

```shell
>  python3  setup.py  sdist
```

**五、将发布包安装在本地Python副本中**

```shell
>  python3  setup.py  install
```

**六、导入并使用**

import  test 

... ...

**注意：**

一、这里的test是一个命名空间在使用test中的函数的时候应该使用 test.fun()来限定

二、如果想直接使用，可以在导入的时候使用  from test  import fun   但是如果当前命名空间有一个名叫fun的函数会被覆盖

最后，可以将代码发布到PyPI网站

http://pypi.python.org   注册一个帐号

然后  

```shell
> python3  setup.py  register
```

输入用户名密码

发布

```shell
> python3  setup.py  sdist  upload
```

如果要更新版本 则修改setup.py文件里的version  然后  python3 setup.py sdist upload