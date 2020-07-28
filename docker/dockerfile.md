
Dockerfile核心说明


下面是使用python部署一个web服务的Dockerfile的例子，能过这个例子来说明Dockerfile的编写

Dockerfile文件内容
```
# 使用官方提供的Python开发镜像作为基础镜像
FROM python:2.7-slim

# 将工作目录切换为/app
WORKDIR /app

# 将当前目录下的所有内容复制到/app下
ADD . /app

# 使用pip命令安装这个应用所需要的依赖
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# 允许外界访问容器的80端口
EXPOSE 80

# 设置环境变量
ENV NAME World

# 设置容器进程为：python app.py，即：这个Python应用的启动命令
CMD ["python", "app.py"]
```

上面的Dockerfile文件名在构建的时候不通过-f参数是可以自支匹配到的，如果自己定义了名字可以通过-f参数来手动的指定Dockerfile