## Dockerfile指令

### FROM 指定基础镜像

* 必备指令,必须是第一条指令
* [Docker Hub](https://hub.docker.com/search?q=&type=image&image_filter=official) 有很多高质量的官方镜像.
* 服务镜像: [nginx](https://hub.docker.com/_/nginx/)、[redis](https://hub.docker.com/_/redis)、[mongo](https://hub.docker.com/_/mongo/)、[mysql](https://hub.docker.com/_/mysql/)、[httpd](https://hub.docker.com/_/httpd/)、[php](https://hub.docker.com/_/php/)、[tomcat](https://hub.docker.com/_/tomcat/) 等
* 语言应用镜像: [node](https://hub.docker.com/_/node)、[openjdk](https://hub.docker.com/_/openjdk/)、[python](https://hub.docker.com/_/python/)、[ruby](https://hub.docker.com/_/ruby/)、[golang](https://hub.docker.com/_/golang/) 等
* 操作系统镜像: [ubuntu](https://hub.docker.com/_/ubuntu/)、[debian](https://hub.docker.com/_/debian/)、[centos](https://hub.docker.com/_/centos/)、[fedora](https://hub.docker.com/_/fedora/)、[alpine](https://hub.docker.com/_/alpine/) 等
* 特殊的镜像: `scratch` ,虚拟概念,表示一个空白镜像

### RUN 执行命令

* exec 格式: `RUN ["可执行文件", "参数1", "参数2"]`

> Docker每一个指令都会建立一层镜像.

### 镜像构建上下文 Context

> 当我们进行镜像构建的时候，并非所有定制都会通过 RUN 指令完成，经常会需要将一些本地文件复制进镜像，比如通过 COPY 指令、ADD 指令等。而 docker build 命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎中构建的。那么在这种客户端/服务端的架构中，如何才能让服务端获得本地文件呢？

> 这就引入了上下文的概念。当构建的时候，用户会指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

### `Docker build` 用法

* **URL构建**

1. git repo 构建:

```
$ docker build https://github.com/twang2218/gitlab-ce-zh.git#:11.1
Sending build context to Docker daemon    141MB
```

2. tar压缩包 构建:

```
$ docker build http://server/context.tar.gz
Sending build context to Docker daemon    141MB
```

* **从标准输入中读取构建** 

1. 从标准输入中读取 **Dockerfile** 构建

```
$ docker build - < Dockerfile
```

```
cat Dockerfile | docker build -
```

2. 从标准输入中读取 **上下文压缩包** 构建

```
$ docker build - < context.tar.gz
```

### COPY 复制文件

* 格式: `COPY [--chown=<user>:<group>] <源路径>... <目标路径>`
* 格式: `COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`

```
COPY hom* /mydir/
COPY hom?.txt /mydir/
COPY --chown=55:mygroup files* /mydir/
COPY --chown=bin files* /mydir/
COPY --chown=1 files* /mydir/
COPY --chown=10:11 files* /mydir/
```

### ADD 更高级的复制文件

* ADD 指令与 COPY 格式和性质基本一致
* 区别在于, COPY 就是复制文件而已,  ADD 指令将会自动解压缩这个压缩文件到 <目标路径> 去.
* 使用遵循的原则, 所有的文件复制均使用 COPY 指令, 仅在需要自动解压缩的场合使用 ADD.

```
ADD --chown=55:mygroup files* /mydir/
ADD --chown=bin files* /mydir/
ADD --chown=1 files* /mydir/
ADD --chown=10:11 files* /mydir/
```

### CMD 容器启动命令

* shell 格式: `CMD <命令>`
* exec 格式: `CMD ["可执行文件", "参数1", "参数2"...]`
* 参数列表格式: `CMD ["参数1", "参数2"...]`. 在指定了 ENTRYPOINT 指令后, 用 CMD 指定具体的参数.

```
CMD echo $HOME
// 实际执行时会转换为:
CMD [ "sh", "-c", "echo $HOME" ]
```

* 运行时可以指定新的命令来替代镜像设置中的这个默认命令.

```
// 用 cat /etc/os-release 命令替换了 ubuntu 镜像默认的 CMD - /bin/bash
docker run -it ubuntu cat /etc/os-release
```

> 注意: Docker 不是虚拟机, 容器中的应用都应该以前台执行, 而不是像虚拟机、物理机里面那样, 用 upstart/systemd 去启动后台服务, 容器内没有后台服务的概念.

```
CMD service nginx start
```

* 上述命令,容器执行后就会立即退出.
* 在容器内去使用 systemctl 命令结果却发现根本执行不了.
* 对于容器而, 其启动程序就是容器应用进程, 容器就是为了主进程而存在的, 主进程退出, 容器就失去了存在的意义, 从而退出, 其它辅助进程不是它需要关心的东西.
* 使用 `service nginx start` 命令, 则是希望 upstart 来以后台守护进程形式启动 nginx 服务.
* 而 `CMD service nginx start` 会被理解为 `CMD [ "sh", "-c", "service nginx start"]`, 因此主进程实际上是 `sh`. 那么当 `service nginx start` 命令结束后, `sh` 也就结束了, `sh` 作为主进程退出了, 自然就会令容器退出.
* 正确的直接执行 `nginx` 可执行文件, 并要求以前台形式运行. `CMD ["nginx", "-g", "daemon off;"]`

### ENTRYPOINT 入口点

* 指定容器启动程序及参数.
* 指定了 `ENTRYPOINT`, CMD 含义就发生了改变, 不是直接运行其命令, 而是将 CMD 内容作为参数传给 ENTRYPOINT 指令, 实际执行时变为:

```
<ENTRYPOINT> "<CMD>"
```

### ENV 设置环境变量

* 格式: `ENV <key> <value>`, `ENV <key1>=<value1> <key2>=<value2>...`

```
ENV VERSION=1.0 DEBUG=on \
	 NAME="Happy Feet"
```

```
ENV NODE_VERSION 7.2.0
// 定义了环境变量 NODE_VERSION, $NODE_VERSION 来操作使用.
RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz"
```

* 支持环境变量展开的指令: `ADD`、`COPY`、`ENV`、`EXPOSE`、`LABEL`、`USER`、`WORKDIR`、`VOLUME`、`STOPSIGNAL`、`ONBUILD`.

### ARG 构建参数

* 构建参数和 `ENV` 的效果一样, 都是设置环境变量.
* 不同的是, `ARG` 所设置的构建环境的环境变量, 在将来**容器运行**时是**不会存在**这些环境变量的.
* `Dockerfile` 中的 `ARG` 指令是定义参数名称, 以及定义其默认值. 默认值可以在构建命令 `docker build` 中用 `--build-arg <参数名>=<值>` 来覆盖.

### VOLUME 定义匿名卷

* 格式: `VOLUME <路径> `, `VOLUME ["<路径1>", "<路径2>"...]`
* 容器运行时应该尽量保持容器存储层不发生写操作.
* 对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(VOLUME)中.
* 为了防止运行时用户忘记将动态文件所保存目录挂载为卷, 在 `Dockerfile` 中, 我们可以事先指定某些目录挂载为匿名卷, 这样在运行时如果用户不指定挂载, 其应用也可以正常运行, 不会向容器存储层写入大量数据.

```
VOLUME /data
```

### EXPOSE 声明端口

* 格式: `EXPOSE <端口1> [<端口2>...]`
* 容器运行时使用 `-p <宿主端口>:<容器端口>` 区分开来. -p, 是映射宿主端口和容器端口.

### WORKDIR 指定工作目录

* 格式: `WORKDIR <工作目录路径>`

### USER 指定当前用户

* 格式: `USER <用户名>[:<用户组>]`

```
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN [ "redis-server" ]
```

### HEALTHCHECK 健康检查

* 格式: `HEALTHCHECK [选项] CMD <命令>` 设置检查容器健康状况的命令
* `HEALTHCHECK NONE` 如果基础镜像有健康检查指令, 使用这行可以屏蔽掉其健康检查指令

### ONBUILD 为他人做嫁衣裳

* 格式: `ONBUILD <其它指令>`



