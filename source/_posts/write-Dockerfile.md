---
title: Dockerfile 编写
categories: "docker教程" 
tags: 
     - docker
     - 教程
---

   开发者经常需要配置一些环境，用来模拟、测试、学习新系统。这时候使用docker，实现这些需求，是十分方便的。编写Dockerfile来完成一个完整的环境，维护、使用起来很是顺手。下面整理了编写一个Dockerfile的实例。

#### 配置nginx的web服务
##### 简介：
   >先使用python pip 包管理工具安装 supervisor, 使用supervisor管理nginx进程。

##### 准备材料:
   - python pip，python包管理工具，安装需要： get-pip.py 文件。
   下载地址：[get-pip.py](https://bootstrap.pypa.io/get-pip.py)
   详细安装请查看：[install pip](https://pip.pypa.io/en/stable/installing/)
   
   - supervisor,是在 UNIX-like 系统下管理进程的工具。
   需要配置文件：supervisord.conf
   详细文档请查看：[supervisord](http://supervisord.org)
   
   - nginx，提供web服务。
   下载地址：[nginx download](http://nginx.org/download/nginx-1.12.0.tar.gz)
    
##### 目录文件:
    app
    Dockerfile
    get-pip.py
    supervisord.conf
    nginx-1.12.0.tar.gz

##### Dockerfile

``` bash
FROM centos:7
WORKDIR /app
ADD . /app
RUN python get-pip.py
RUN pip install supervisor
COPY supervisord.conf /etc/supervisor/supervisord.conf
RUN tar -xzvf nginx-1.12.0.tar.gz
RUN yum -y install gcc
RUN yum -y install make 
RUN yum -y install pcre-devel openssl openssl-devel
WORKDIR /app/nginx-1.12.0
RUN ./configure --prefix=/usr/local/nginx
RUN make
RUN make install
EXPOSE 80  9001
CMD ["/usr/bin/supervisord"]
```

##### supervisord.conf

``` bash
[supervisord]
nodaemon=true
logfile=/app/logs/supervisord.log
pidfile=/app/logs/supervisord.pid
[inet_http_server]         ; supervisord的tcp服务配置
port=0.0.0.0:9001          ; tcp端口
username=demo              ; tcp登陆用户
password=demopwd           ; tcp登陆密码

[program:nginx]
command=/usr/local/nginx/sbin/nginx
autorestart=unexpected
autostart=true
``` 

##### 创建image命令

``` bash
$bash docker build -t nginx-1.12.0 ./
```

##### 启动容器
``` bash
$bash docker run --name nginx -id -p 8888:80 -p 9001:9001 nginx-1.12.0
```

##### 进入容器
``` bash
$bash docker exec -it nginx bash
``` 

##### 查看服务
``` bash
nginx:
    http://loclahost:8080/
supervisor:
    http://localhost:9001/
```

【完】下面是常用命令


-----

#### Docker 常用命令：

获取 [docker hub](https://hub.docker.com/) 上的 image:
``` 
docker pull centos:7
```
查看本机 image 列表
``` 
docker images
``` 

从image创建container:
``` 
(-i 交互式，-t新开启tty，-d daemon运行)
docker run -i -d centos:7 /bin/bash
``` 

查看运行的 container:
``` 
docker ps 
``` 

查看所有container （stopped starting）
``` 
docker ps -a 
``` 

进入运行的container:
``` 
docker exec -it [container id] /bin/bash
``` 

提交新的image:
``` 
docker commit -m “注释，修改了什么” -a “作者，修改人”  [container id] myimage/centos7:0.1
``` 
停止container
``` 
docker stop container id
``` 

删除container
``` 
docker rm container id
``` 

删除image
``` 
docker rmi image id
``` 

