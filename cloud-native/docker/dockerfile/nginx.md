
Nginx反向代理

以下是Harbor反向代理的简单配置，下面是将配置放在了/etc/nginx/nginx.conf里面
```
  user nginx;
  worker_processes 1;
  
  error_log /var/log/nginx/error.log warn;
  
  pid /var/run/nginx.pid;
  
  events {
      worker_connections 1024;
  }
  
  http {
    upstream hub {
      server 106.54.22.232:80;
    }
  
    access_log  /var/log/nginx/access.log;
  
    client_max_body_size 500m;
  
  
    server {
      listen 443 ssl;   #SSL协议访问端口号为443。此处如未添加ssl，可能会造成Nginx无法启动。
      server_name www.seepre.com;  #将localhost修改为您证书绑定的域名，例如：www.example.com。
      root html;
      index index.html index.htm;
      ssl_certificate cert/ssl.pem;   #将domain name.pem替换成您证书的文件名。
      ssl_certificate_key cert/ssl.key;   #将domain name.key替换成您证书的密钥文件名。
      ssl_session_timeout 5m;
      ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;  #使用此加密套件。
      ssl_protocols TLSv1 TLSv1.1 TLSv1.2;   #使用该协议进行配置。
      ssl_prefer_server_ciphers on;
  
      location / {
        proxy_pass http://hub;
        root html;
        index index.html index.htm;
      }
    }
  
    server {
      listen 80;
      server_name www.seepre.com;   #将localhost修改为您证书绑定的域名，例如：www.example.com。
      rewrite ^(.*)$ https://$host$1 permanent;   #将所有http请求通过rewrite重定向到https。
      location / {
        index index.html index.htm;
      }
    }
  }  
```

通过脚本的方式启动Nginx
``` 
#!/bin/bash
docker stop nginx
docker rm nginx
docker run -idt --net=host --name nginx -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf nginx:1.13.12
```


在harbor新建一个项目hub
然后在本地找一个镜像打一上tag
docker tag nginx:1.13.12 harbor.seepre.com/hub/nginx:1.13.12


遇到以下错误
error parsing HTTP 413 response body: invalid character '<' looking for beginning of value: "<html>\r\n<head><title>413 Request Entity Too Large</title></head>\r\n<body>\r\n<center><h1>413 Request Entity Too Large</h1></center>\r\n<hr><center>nginx/1.19.1</center>\r\n</body>\r\n</html>\r\n"

设置nginx配置client_max_body_size 200 M 