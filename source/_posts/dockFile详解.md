---
title: DockerFile详解
date: 2019-08-08 16:26:02
tags: docker
categories: 容器及虚拟化技术
---

docker 可以通过dockerfile中的指令来构建镜像，dockerFile中描述了构建镜像的详细步骤。每一条指令构建一层，因此每一条指定的内容，就是描述该层应当如何构建。

构建镜像的指令：docker build -t {镜像名字} {项目的路径可以是相对路径}

#### dockerfile中指令详解

##### from指令

FROM 指定基础镜像,所谓的定制镜像，必须是以一个镜像为基础，在其上进行定制。一个 Dockerfile 中的 FROM 是必备的指定，并且必须是第一条指令。
例如SpringBoot 需要已jdk为基础镜像 。jdk需要以具体的操作系统的为基础镜像


##### RUN 执行命令

RUN 指令是用来执行命令行命令的，一个Dockerfile可以包含多个 RUN，按照定义顺序执行。
RUN 支持两种运行格式：
> * shell 格式:RUN <cmd>，这个会当做/bin/sh -c “cmd” 运行。eg: RUN sh -c 'touch /app.jar'

> *  exec 格式:RUN ["可执行文件", "参数1", "参数2", ...]，Docker 把它当做json的顺序来解析，必须使用双引号，且可执行文件必须是完整路径。
    
RUN 就像Shell脚本一样可以执行命令，但是也不建议一个命令对应一个RUN，像下面这样的写法

```shell
FROM centos
RUN yum install -y gcc make cmake
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redi s-3.2.5.tar.gz" 
RUN mkdir -p /usr/src/redis
RUN tar xf redis.tar.gz -C /usr/src/redis --stript-components=1
RUN cd /usr/src/redis
RUN make
RUN make install

```
 因为Dockerfile 中的每一个指令都会建立一层，RUN 也不例外，每一个RUN 的行为，就和我们手工创建景象的过程一样，新建一层，上面的这种写法创建了7层，这样会产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也容易出错。上面的Dockerfile正确写法应该如下：
 
```shell
FROM centos
RUN yum install -y gcc make cmake \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redi s-3.2.5.tar.gz"  \
    && mkdir -p /usr/src/redis \
    && tar xf redis.tar.gz -C /usr/src/redis --stript-components=1 \
    && cd /usr/src/redis \
    && make \
    && make install

```
首先，之前所有的命令只有一个目的，就是编译、安装redis 可执行文件。因此没有必要建立很多层，这只是一层的事情。因此 只需要使用一个RUN 指令，使用&& 将各个所需命令串联起来。将前面的7层简化为1层。


##### COPY 复制文件

> * COPY <源路径> ... <目标路径>
> * COPY ["<源路径1>", ... "<目标路径>"]

COPY 指令将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置


##### ADD 更高级的复制文件

ADD 指令和COPY 的格式和性质基本一致。但是在 COPY 基础上增加了一些功能。

比如 <源路径> 可能是一个 URL，这种情况下，会自动去下载这个链接的文件到 <目标路径>里，下载完成的文件权限自动设置为 600，如果不是想要的权限，那么可以通过 RUN 进行权限调整。

如果 <源路径> 为一个 tar 压缩文件的话，或者压缩格式为 gzip，bzip以及xz 的情况下，ADD 指令会自动解压缩这个压缩文件到 <目标路径>去

注意：如果我们只是单纯的想复制一个文件或者一个压缩包，那么建议还是使用COPY，如果需要像上面这样用到下载，或解压缩，采用ADD


##### CMD & ENTRYPOINT 

CMD和ENTRYPOINT的作用是一样的都是设置在容器启动时候要执行的命令

eg: ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"] 当容器启动就是执行java -jar xxx.jar

cmd和ENTRYPOINT 区别就是 CMD的命令会被 docker run 的命令覆盖而ENTRYPOINT不会。
使用CMD ["/bin/bash"] 如果启动镜像的命令为docker run -ti image /bin/ps，使用CMD后面的命令就会被覆盖转而执行bin/ps命令，
而ENTRYPOINT的则不会，而是会把docker run 后面的命令当做ENTRYPOINT执行命令的参数。

当CMD和ENTRYPOINT都存在时，CMD的指令变成了ENTRYPOINT的参数，并且此CMD提供的参数会被 docker run 后面的命令覆盖，如：

```javascript
...
ENTRYPOINT ["echo","hello","i am"]
CMD ["docker"]

```
之后启动构建之后的容器

> * 使用docker run -ti image  输出“hello i am docker”
> * 使用docker run -ti image world 输出“hello i am world”


##### ENV 设置环境变量

> * ENV <key><value>
> * ENV<key1>=<value1> <key2>=<value2>

这个指令很简单，就是设置环境变量，无论是后面的其它指令，如RUN，还是运行时的应用，都可以直接使用这里定义的环境变量。


##### VOLUME 定义匿名卷

> * VOLUME ["<路径1>","<路径2>"...]
> * VOLUME <路径>

将容器目录映射到宿主机，容器运行时应该尽量保持容器存储层不会发生写操作，对于数据库类需要保存动态数据的应用，其数据文件应该保存在数据卷中，为了防止运行时忘记将动态文件所保存目录挂载为卷，在Dockerfile中，可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。


##### EXPOSE 声明端口

> * EXPOSE <端口1> [<端口2>...]

EXPOSE 指令是声明运行容器时提供的服务端口，这只是一个声明，在运行时并不会因为这个声明就会开启这个端口的服务。在Dockerfile中写入这样的声明有两个好处，一个是帮助使用镜像者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射EXPOSE的端口。


##### WORKDIR 指定工作目录

> * WORKDIR <工作目录路径>


使用 WORKDIR 指令可以来制定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，则会自动创建。

可以为RUN、CMD、ENTRYPOINT指令配置工作目录，可以使用多个WORKDIR指令，后续参数如果是相对路径，则会基于之前的命令指定的路径。如：WORKDIR /home   WORKDIR test。最终路径为/home/test。path路径也可以是环境变量，比如有环境变量HOME=/home,   WORKDIR $HOME/test也就是/home/test
