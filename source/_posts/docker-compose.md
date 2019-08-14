---
title: docker-compose
date: 2019-08-12 14:33:58
tags: docker
categories: 容器及虚拟化技术
---

有关docker-compose的安装以及详细介绍可以参考下面这个帖子
[Docker快速入门——Docker-Compose](https://blog.51cto.com/9291927/2310444)

使用docker-compose部署mysql和springboot实例

```dockerfile

version: '2'
services:
  mysqldbserver:
    image: mysql:5.7
    ports:
      - 3306:3306
    volumes:
      - /opt/docker-mysql/conf.d:/etc/mysql/mysql.conf.d
      - /opt/docker-mysql/var/lib/mysql:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD : 12345678
  spring-boot:
    image: springboot/springboot-web-security
    ports:
      - 8080:8080
    network_mode: "host"
    volumes:
      - /opt/logs:/logs  
    depends_on:
      - mysqldbserver

```
