---
title: Docker
date: 2020-05-28
tags: notes docker
categories: technology
keywords: docker
---

# 一、入门

在这一章会做三件事情:

- 安装Docker并运行一个`Hello World`容器.
- 从命令行获取帮助.
- 解释Docker的分层系统.

## 1.1 Hello World

- 从Docker的下载界面下载到Docker的安装包并完成安装.

- 使用如下命令测试Docker是否正确完成安装:

  ```dockerfile
  docker --version
  ```

- 使用`hello-world`镜像运行一个`Hello World`容器检查Docker后台进程是否已开启:

  ```dockerfile
  docker run hello-world
  ```

- 若Docker后台进程已开启, 则会看到控制台打印了`Hello from Docker!`和其他的一些信息. 使用`docker container ls --all`可以看到刚才创建的`Hello World`容器, 并且状态为`Exit`(如果容器的状态为`up`则不需要加上`--all`就会直接显示出来).

## 1.2 从命令行获取帮助

Docker提供了许多命令, 可以使用`docker help`获得全局的帮助信息, 但如果想要知道一个特定命令的帮助信息, 可以使用`docker help command`来获取该命令的帮助信息, 如`docker help container`.

## 1.3 Docker分层系统

镜像构建时, 会一层层构建, 前一层是后一层的基础. 每一层构建完就不会再发生改变, 后一层上的任何改变只发生在自己这一层. 比如在当前层执行删除属于前一层的文件的操作, 实际不是真的删除前一层的文件, 而是仅在当前层标记为该文件已删除. 在最终容器运行的时候, 虽然不会看到这个文件, 但是实际上该文件会一直跟随镜像. 因此, 在构建镜像的时候, 需要尽量确保每一层只包含该层需要添加的东西, 任何额外的东西应该在该层构建结束前清理掉.

# 二、镜像

容器由镜像产生, 镜像和容器之间的关系类似于面向对象中类和实例的关系. 创建容器需要首先从镜像仓库下载镜像, 或者自己构建镜像.

## 2.1 获取镜像

获取镜像的方式有两种: 自动获取和手动获取.

- 自动获取: 当执行`run`命令使用指定镜像创建容器的时候, 如果镜像不存在, Docker会自动地下载该镜像.
- 手动获取: 可以手动从Docker镜像仓库获取镜像, 命令为: `docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]`. 需要注意:
  - 如果不指定"Docker Registry地址", 那么默认会是Docker Hub.
  - 如果不指定"标签", 那么默认会是`latest`.

## 2.2 列出镜像

可以使用`docker image ls`列出当前已经安装的镜像. 有以下几个知识点:

- 镜像体积: 使用命令列出来的镜像的体积可能会比Docker Hub中显示的体积大, 原因是Docker Hub中的体积是压缩后的体积, 而下载下来的则是展开后各层所占空间的总和.
- 虚悬镜像: 在列出来的镜像中可能会看到一些没有仓库名也没有标签名的镜像, 如: `<none>               <none>              00285df0df87        5 days ago          342 MB`, 这种镜像叫做"虚悬镜像". 它们原本是有镜像名和标签的, 随着官方镜像维护, 发布了新版本, 重新`docker pull 镜像名`时, "镜像名"被转移到了新下载的镜像身上, 而旧的镜像的名字则被取消, 从而形成了`<none>`. 除了`docker pull`之外, `docker build`也可能导致这种现象, 由于新旧镜像同名, 旧镜像名被取消, 从而出现了镜像名和标签都为`<none>`的情况. 虚悬镜像已经失去存在价值, 可以随意使用`docker image prune`来删除.
- 中间层镜像: 为了加速镜像构建, 重复利用资源, Docker会利用中间层镜像. `docker image ls`只会列出顶层镜像, 如果需要列出中间层镜像的话需要使用`docker image ls -a`. 虽然中间层镜像的镜像名和标签名都为`<none>`, 但它与虚悬镜像不一样(虚悬镜像直接`docker image ls`就可以出来了). 需要注意的是中间层镜像不可以随便删除.

## 2.3 删除镜像

删除镜像的命令为: `docker image rm [选项] <镜像1> [<镜像2> ...]`. 其中"镜像"可以是镜像名或者镜像的长短ID等可以唯一标识镜像的串.

## 2.4 构建镜像

除了可以从镜像仓库下载镜像之外, 还可以构建自己的镜像. 构建镜像的方式有两种:

- `commit`: 对现有容器执行`commit`命令将容器的变更保存并提交为新的镜像.
- `Dockerfile`: 编写`Dockerfile`, 然后使用`build`命令基于该文件执行镜像构建.

### 2.4.1 commit

镜像是多层存储, 每一层是在前一层的基础上进行的修改. 容器同样也是多层存储, 是在以镜像为基础层, 在其基础上加一层作为容器运行时的存储层. 当运行一个容器的时候, 如果不使用卷, 那么所有改动都会发生在存储层上面. 这时候可以使用`docker commit`命令将存储层的改动以镜像的形式保存下来:

```bash
docker commit --author "作者名" --message "commit信息" containerName newImageName:newImageTagName
```

容器成功提交为镜像之后可以使用`docker image ls`来查看. 另外如果想要查看容器内存储层的文件改动情况可以使用`docker diff containerName`命令.

### 2.4.2 Dockerfile

以`commit`命令创建镜像的方式存在的问题有:

- 无法自动构建镜像, 每个步骤都要手动执行.
- 无法与他人共享构建步骤(指的是容器内存储层文件的修改步骤).

基于`Dockerfile`的镜像构建方式可以解决这些问题. 该文件通过使用一组指令来描述镜像是如何构建的. 如"新镜像的基础镜像", "镜像内的文件经过了哪些步骤的变动", "要暴露哪些端口", "使用镜像创建容器时默认运行什么命令"等. 一个示例`Dockerfile`文件内容如下:

```dockerfile
# Use the official image as a parent image.
FROM node:current-slim

# Set the working directory.
WORKDIR /usr/src/app

# Copy the file from your host to your current location.
COPY package.json .

# Run the command inside your image filesystem.
RUN npm install

# Inform Docker that the container is listening on the specified port at runtime.
EXPOSE 8080

# Run the specified command within the container.
CMD [ "npm", "start" ]

# Copy the rest of your app's source code from your host to your image filesystem.
COPY . .
```

`Dockerfile`编写完成后可以使用`docker build [选项] <上下文路径/URL/->`命令执行镜像的构建. 如:  `docker build -t tiangou-daily-crawler:v0.0.1 .`.

#### 2.4.2.1 RUN指令

在`Dockerfile`中, 最重要的一个命令是`RUN`, 它是用来执行命令的. 如`RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html`. 但是要注意`Dockerfile`中每一个指令都会建立一层, 包括`RUN`指令. 所以不要像在shell中写命令那样一个shell命令一个构建指令地执行, 而是将所有需要执行的命令合并在一起当成一个Docker构建指令. 如下:

```dockerfile
# 不要这样做, 一个RUN命令一个Docker镜像层
FROM debian:stretch

RUN apt-get update
RUN apt-get install -y gcc libc6-dev make wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install



# 这样做, 将所有需要执行的shell命令合并在一起, 作为一个Docker镜像层
FROM debian:stretch

RUN buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

#### 2.4.2.2 镜像构建上下文

`docker build`命令的最后一个参数表示上下文路径. Docker daemon是在服务端运行的, 用户在构建镜像的时候, Docker会将上下文中的文件打包, 上传到Docker引擎. 然后在`Dockerfile`里面引用的文件路径都是相对于这个上下文环境中的路径的. 如`COPY ./package.json /app/`实际上复制的是打包的上下文环境中的`./package.json`文件. 如果上下文环境中有文件不想在构建的时候传递给Docker引擎, 可以用类似`.gitignore`的写法写一个`.dockerignore`, 该文件的作用是剔除不需要作为上下文传递给Docker引擎的文件.

#### 2.4.2.3 远程构建

除了在本地构建之外, 还可以使用url从指定的git仓库上执行构建:

```dockerfile
# master分支, 并且clone完项目后到/11.1/下开始构建
docker build https://github.com/twang2218/gitlab-ce-zh.git#:11.1
```

#### 2.4.2.4 Dockerfile 命令

- `FROM`: 指定基础镜像, 从这个镜像开始构建.

- `RUN`: 执行一个命令.

- `COPY`: 将文件从构建上下文目录中复制到新的镜像层的指定目录内. 

- `ADD`: 和`COPY`类似, 但是在它的基础上增加了一些功能, 比如可以用`URL`来获取文件. 在Docker官方的最佳实践文档中告诉我们, 应尽可能地使用`COPY`. 最合适使用`ADD`的场合, 就是需要自动解压缩的场合, 因为如果源路径是一个`tar`压缩文件的话, 该指令会自动解压这个压缩文件.

- `CMD`: 在启动容器的时候, 需要指定所运行的程序及参数. 该命令就是用于指定默认的容器主进程的启动命令的. 在运行的时候可以指定新的命令替代镜像设置中的这个默认命令. 比如`ubuntu`镜像默认的`CMD`是`/bin/bash`, 如果直接`docker run -it ubuntu`的话, 会直接进入`bash`. 如果需要在运行的时候指定别的命令, 可以在启动的时候加上, 用以替代默认的命令. 如: `docker run -it ubuntu cat /etc/os-release`. 另外需要注意: **对于容器而言, 容器是为了主进程而存在的, 主进程退出, 容器就是去了意义, 从而退出, 其他辅助进程不是它需要关心的东西**.

- `ENTRYPOINT`: 和`CMD`类似, 都是指定容器启动时候运行的程序的, 但是区别如下:

  ```dockerfile
  # Dockerfile1
  FROM ubuntu
  CMD [ "curl", "-s", "https://ip.cn" ]
  # docker run myip -i 这里 -i 会替换掉这个容器的启动程序.
  
  #Dockerfile2
  FROM ubuntu
  ENTRYPOINT [ "curl", "-s", "https://ip.cn" ]
  # docker run myip -i 这里 -i 不会会替换掉这个容器的启动程序, 而是作为这个启动程序的参数传递过来.
  ```

- `ENV`: 设置环境变量. 设置完之后, 后面的其他指令, 如`RUN`都可以直接使用这里定义的环境变量. 如:

  ```dockerfile
  ENV NODE_VERSION 7.2.0
  
  RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
    && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
    && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
    && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
    && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs
  ```

- `ARG`: 构建参数. 也是用来设置环境变量, 但是和`ENV`不同的是, 用这个命令设置的变量, 在容器运行时是不会存在这些环境变量的.

- ` VOLUME`: 容器运行时应该尽量保持容器存储层不发生写操作, 对于数据库类需要保存动态数据的应用, 其数据文件应该保存于卷(volume)中. 为了防止运行时用户忘记将动态文件所保存目录挂载为卷, 可以在Dockerfile中事先指定某些目录挂载为匿名卷, 这样尽管运行时用户不指定挂载卷, 应用也可以正常运行, 不会向容器存储层写入大量数据. 如: `VOLUME /data`, 这样该目录在运行时自动挂载为匿名卷, 任何向`/data`中写入的信息都不会记录进容器存储层, 从而保证了容器存储层的无状态话.

- `EXPOSE`: 声明运行时容器提供服务端口, 只是一个声明, 在运行时并不会因为这个声明, 应用就会开启这个端口的服务. 在Dockerfile中写入这样的声明有两个好处: 一个是帮助镜像使用者理解这个镜像服务的守护端口, 以方便映射. 另一个是在运行时使用随机端口映射时, 也就是`docker run -P`时, 会自动随机映射`EXPOSE`端口(指的是将EXPOSE指定的容器的端口映射到宿主的随机端口).

- `WORKDIR`: 指定工作目录, 以后各层的当前目录就被改为指定的目录, 如该目录不存在, `WORKDIR`会帮你建立目录.

- `USER`: 指定当前用户.

- `HEALTHCHECK`: 健康检查.

- `ONBUILD`: ....

# 三、容器

## 3.1 启动容器

启动容器有两种方式: 基于镜像新建一个容器和将终止状态(`stopped`)的容器重新启动.

- 新建容器: `docker run`.  需要注意如果不使用`-d`参数的话, 容器是会在当前shell的前台运行直到该容器的主进程结束为止.
- 将终止状态的容器重新启动: `docker container start`.

## 3.2 进入容器

当使用`-d`参数运行容器时, 容器启动后会进入后台. 如果要进入容器执行某些操作, 可以使用`docker attach`命令或`docker exec`命令.

一般使用`docker exec`命令, 因为使用这个命令进入容器之后执行`exit`命令并不会终止容器, 而是仅退出容器:

```dockerfile
# 进入容器
docker exec -it docker_redis_0 /bin/bash

# 在容器内执行一些操作
# ...

# 退出容器
# exit仅退出容器, 不会终止容器
exit
```

## 3.3 终止容器

如果需要将一个运行中的容器终止的话, 可以使用`docker container stop`命令. 除了使用该命令之外, 当容器内的主进程终止的时候, 容器也自动终止. 终止的容器可以使用`docker container ls -a`来查看, status为`Exited`的容器即为终止的容器.

## 3.4 删除容器

可以删除一个终止状态的容器: `docker container rm docker_redis_0`. 注意不可以删除运行中的容器, Docker会直接报错"You cannot remove a running container.....". 如果需要一次性删除所有终止状态的容器, 可以使用`docker container prune`. 

## 3.5 列出容器

列出容器的命令为`docker container ls`, 但这仅会列出运行中的容器. 如果需要将终止状态的容器也列出, 可以使用`docker container ls -a`.

# 四、仓库

仓库时集中存放镜像的地方. 一般使用[Docker Hub](https://hub.docker.com/).

# 五、数据管理

在Docker容器中管理数据主要有两种方式: 数据卷(Volumes)和挂载主机目录(Bind mounts).

## 5.1 数据卷

数据卷是一个可供一个或多个容器使用的特殊目录, 它有如下特性:

- 可以在容器之间共享和重用.
- 对数据卷的修改会立马生效.
- 对数据卷的更新不会影响到镜像.
- 数据卷会一直存在, 即使容器被删除.

数据卷类似于Linux下对目录或文件的mount. 容器中被指定为挂载点的目录中的文件将会被隐藏掉, 替换之的则是被挂载的数据卷.

### 5.1.1 创建数据卷

创建数据卷的命令为`docker volume create my-vol`.

### 5.1.2 查看数据卷

对于数据卷的查看, 可以列出所有存在的数据卷, 也可以查看指定数据卷的信息:

- 列出存在的数据卷: `docker volume ls`.
- 查看指定数据卷的信息: `docker volume inspect volumeName`.

### 5.1.3 启动挂载数据卷的容器

如果需要将数据卷挂载到容器上, 可以在启动容器的时候加上`--mount`参数来挂载数据卷. 在一次`docker run`中可以同时挂载多个数据卷. 下面示例将`my-vol`数据卷挂载到`/webapp`目录.

```bash
docker run -d --name web --mount source=my-vol,target=/webapp nginx
```

### 5.1.4 删除数据卷

数据卷是被设计用来持久化数据的, 它的生命周期独立于容器, Docker不会在容器被删除后自动删除数据卷, 并且也没有垃圾回收机制来回收没有被任何容器引用的数据卷. 所以如果需要删除数据卷的话可以手动执行`docker volume rm volumeName`命令来删除. 如果需要一次性删除全部没有使用的数据卷, 可以使用命令`docker volume prune`.

### 5.1.5 查看容器挂载的数据卷

如果需要查看一个容器挂载了哪些数据卷, 可以使用`docker inspect containerName`, 该命令列出的信息中包含了容器数据卷的挂载信息.

## 5.2 挂载主机目录

除了可以挂载数据卷之外, 还可以将主机目录作为挂载点挂载到容器上. 容器中对目录的所有操作都会持久化到该目录. 挂载主机目录和挂载数据卷的命令不同点在于挂载主机目录多了一个`type=bind`选项. 示例命令如下:

```bash
# 将本机目录/Users/jeb/TestDir/Docker/mounttestdir挂载到容器的/webapp目录下
docker run -d --name web --mount type=bind,source=/Users/jeb/TestDir/Docker/mounttestdir,target=/webapp nginx

# 容器对该目录的所有操作都会反映到主机上
# 进入容器并在/webapp目录下创建一个文件
docker exec -it web /bin/bash
cd /webapp
echo 'Hello World!' > hello.out
exit

# 进入主机的/Users/jeb/TestDir/Docker/mounttestdir目录, 可以看到在容器中创建的文件
cd /Users/jeb/TestDir/Docker/mounttestdir
```



Docker挂载主机目录的默认权限是**读写**, 如果需要设置为只读, 可以添加`readonly`: `docker run -d --name web --mount type=bind,source=/Users/jeb/TestDir/Docker/mounttestdir,target=/webapp,readonly nginx`.

# 六、网络

Docker允许通过外部访问容器或容器互联的方式来提供网络服务.

## 6.1 外部访问网络

容器中可以运行一些网络应用, 要让外部也可以访问这些应用, 可以通过`-P`或`-p`参数来指定物理主机到容器的端口映射. `-P`参数表示将一个随机的物理主机端口映射到容器内部的开放端口上, 如Redis容器内部使用了6379端口, 假如在运行容器的时候使用了`-P`参数, 这时会将一个随机的物理主机端口映射到容器的6379端口上. `-p`表示将指定的物理主机端口映射到指定的容器端口, 如`docker run --name redis_0 -d -p 6379:6379 redis`, 需要注意的是端口参数中前者表示物理主机端口, 后者表示容器端口. 如果需要查看一个容器的端口映射情况, 可以使用`docker port containerName`命令.

## 6.2 容器互联

可以将容器加入自定义的Docker网络来连接多个容器. 步骤如下:

- 新建Docker网络: `docker network create -d bridge my-net`(`-d`用于指定网络类型, 有`bridge`和`overlay`).
- 运行容器并连接到创建的网络:
  - 第一个容器加入网络(在终端A中): `docker run -it --rm --name busybox1 --network my-net busybox sh`.
  - 第二个容器加入网络(新建另一个终端B): `docker run -it --rm --name busybox2 --network my-net busybox sh`.
- 测试Docker网络内容器的互联情况: 在终端A中`ping`容器B(`busybox2`): `ping busybox2)`, 可以看到有报文响应信息, 说明此时容器A(`busybox1`)和容器B(`busybox2`)已经成功互联.
