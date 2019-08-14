---
title: Docker部署mysql
date: 2019-08-07 15:42:52
tags: docker
categories: 容器及虚拟化技术
---

#### 查看mysql的镜像

```shell
docker search mysql  //查看mysql官方发行版
```
   ![](https://dashuaishuaishuai.github.io/image/docker_search.png)
   
可以到dockerhub上查看mysql的镜像 官网地址 https://hub.docker.com/_/mysql
官网上提供了一些实例，展示如何在docker上部署并启动mysql


 
#### 拉取mysql镜像

```shell
docker pull mysql:tag  //tag是mysql的版本号 eg: docker pull mysql:5.7
```

   ![](https://dashuaishuaishuai.github.io/image/docker-mysql.png)
   
#### 启动mysql

将docker中的mysql文件系统映射到宿主机

```shell
    mkdir -p /opt/docker-mysql/conf.d
```
在刚才创建的目录下增加配置文件mysqld.cnf 内容如下

```shell
    [mysqld]
    pid-file        = /var/run/mysqld/mysqld.pid
    socket          = /var/run/mysqld/mysqld.sock
    datadir         = /var/lib/mysql
    #log-error      = /var/log/mysql/error.log
    # By default we only accept connections from localhost
    #bind-address   = 127.0.0.1
    # Disabling symbolic-links is recommended to prevent assorted security risks
    symbolic-links=0
```
增加数据文件夹

 ```shell 
    mkdir -p /opt/docker-mysql/var/lib/mysql
```
启动命令
```shell
    docker run --name mysql-docker \
            --restart=always \   
            -p 3306:3306 \  
            -v /opt/docker-mysql/conf.d:/etc/mysql/mysql.conf.d \
            -v /opt/docker-mysql/var/lib/mysql:/var/lib/mysql \
            -e MYSQL_ROOT_PASSWORD=12345678 \
            -d mysql:5.7
```

        
查看mysql 启动日志

```shell
    docker logs -f mysql
```

    
进入容器
```shell
  docker exec -it mysql-docker bash // mysql-docker是容器名称    
```
   
    
![](https://dashuaishuaishuai.github.io/image/docker-mysql-exec.png)
