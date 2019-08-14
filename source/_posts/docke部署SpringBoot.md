---
title: Docker部署SpringBoot
date: 2019-08-07 09:46:00
tags: SpringBoot
categories: 容器及虚拟化技术

---

### 利用maven插件部署

#### 编写dockerfile

dockerfile的指令介绍可以看以下博文

[Docke--Dockerfile指令介绍](https://www.cnblogs.com/yanjieli/p/10246117.html)

以下是部署springboot时的dockfile
```java

FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE
ADD springboot-web-security-0.0.1-SNAPSHOT.jar app.jar
RUN sh -c 'touch /app.jar'
EXPOSE 8080
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]

```

####  添加maven插件


再pom中添加docker的插件
```java

 <!-- Docker maven plugin -->
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.0.0</version>
                <configuration>
                    <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
                    <!--DockerFile的路径 -->
                    <dockerDirectory>src/main/resources</dockerDirectory>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>

```

#### 构建

查看已有镜像
![查看已有镜像](https://dashuaishuaishuai.github.io/image/docker-image.png)


将整个项目放到安装了docker,maven的主机上 执行 mvn clean & mvn package & mvn docker:build

![构建过程](https://dashuaishuaishuai.github.io/image/docker-image1.png)

构建完成再看镜像


![构建好的镜像](https://dashuaishuaishuai.github.io/image/docker-image2.png)


#### 启动容器

```javascript
    docker run --net=host -p 8080:8080  -v /opt/logs:/logs -t springboot/springboot-web-security

```

--net=host 指定容器的网络模式
-v /opt/logs:/logs  将项目的日志目录映射到宿主机
-p 8080:8080  端口映射

![容器启动成功](https://dashuaishuaishuai.github.io/image/docker-image3.png)


### 利用编译好的jar包部署

Dockerfile跟利用maven插件部署的方式是一样的。

#### 构建

新建一个文件夹将Dockerfile和编译好的jar放进来

![](https://dashuaishuaishuai.github.io/image/docker-jar.png)

进入目录执行命令

    docker build -t springboot/springboot-web-security .
    
 其中 springboot/springboot-web-security 是镜像名称  
 . 代表是当前目录，也就是Dockerfile所在的目录，也可以输入具体路径
 
 
![构建过程](https://dashuaishuaishuai.github.io/image/docker-image4.png)


构建完成后查看构建好的镜像

![构建好的镜像](https://dashuaishuaishuai.github.io/image/docker-image5.png)

 构建完成后启动容器即可测试
