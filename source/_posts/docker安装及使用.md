---
title: docker安装及使用
date: 2019-08-05 16:44:05
tags: docker
categories: docker
---


#### docker安装

docker的安装主要有三种方法
    
    1.先安装docker源 然后基于这个源来安装。这是比较推荐的做法，这样便于安装和升级。
    2.下载 docker 的RPM包，然后手动安装，并且完全手动管理和升级，这种方式适合安装的主机无法接入互联网的情况
    3.在测试开发环境中，也可以用自动化脚本进行简易安装
    

在线安装，如果以前安装过得话 先卸载

```shell
yum remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-selinux \
docker-engine-selinux \
docker-engine
```

安装最新版本的docker ce

```shell
yum install docker-ce
```
如果报No package docker available则需要先安装

安装docker源

安装需要的yum功能包 yum-utils用于支持yum-config-manager（yum配置管理器）功能，device-mapper-persistent-data和lvm2用户支持devicemapper存储驱动。
```shell
yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
``` 
增加稳定的docker源，尽管edge或者test的源也可以，但是还是有稳定的比较好。

```shell
yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
```

docker源安装完成之后就可以执行 yum install docker-ce 进行安装了。
如果还报No package docker available yum没有找到docker-ce包，而docker-ce是第三方软件，更新epel第三方软件库，
运行命令：yum install epel-release
EPEL (Extra Packages for Enterprise Linux)是基于Fedora的一个项目，为“红帽系”的操作系统提供额外的软件包，适用于RHEL、CentOS和Scientific Linux.
再次执行：yum install docker-ce



#### docker安装

启动： service docker start

状态： service docker status

重启： service docker restart

停止： service docker stop


拉取镜像：docker pull gitlab/gitlab-ce:latest

删除镜像：

    删除images，通过image的id来指定删除谁
    docker rmi <image id>
    想要删除untagged images，也就是那些id为<None>的image的话可以用
    docker rmi $(docker images | grep "^<none>" | awk "{print $3}")
    要删除全部image的话
    docker rmi $(docker images -q)
    
查看docker启动的容器列表
    docker ps
查看docker创建的所有容器
    docker ps -a

容器启动停止

    docker start 容器名 或者 容器id
    docker stop 容器名 或者 容器id
    docker restart 容器名 或者 容器id
查看容器日志	

    docker logs -f 容器名 或者 容器id

删除某个容器，若正在运行，需要先停止

    docker stop 容器名 或者 容器id
    docker rm 容器名 或者 容器id
    docker rm $(docker ps -a -q)  删除所有容器
