# Docker初识


# 前言

在我这可以，在你那怎么就跑不起来了？
在以前，项目开发完成后，测试/运维需要从头到尾搭建环境、调试。这样的话经常出现在开发的口中，不过，随着技术的不断发展，在容器技术诞生后，这个问题就可以轻松的解决了。容器技术就像一个打包工具，使我们可以以与管理应用程序相同的方式管理基础架构。部署操作都命令化，集中成一个脚本就可以完成原来复杂的部署过程。打包的不仅是你的程序，也包括运行的环境，因此消除了我们重复的、单调的配置/调试任务。项目在打包后可以在所有的环境下，以相同的方式运行。让开发测试等成员不再关注构建、调试等事情。可以将更多的时间放在业务本身。

# docker 简介

不同于虚拟机，容器技术只隔离应用程序的运行环境但容器之间可以共用同一个操作系统，这使它在继承了虚拟机优点的同时还摒弃了其缺点。而 docker，则是容器技术的其中一种实现。

docker 是一个用 Go 语言实现的开源项目，可以让我们方便的创建和使用容器。

![https://cdn.nlark.com/yuque/0/2022/png/22641131/1671604671214-dfb9ce8d-7f63-4f19-af02-ebd3634db7a4.png](https://img-blog.csdnimg.cn/img_convert/59ae238dfc7be0360ee218363e29bf58.png)

docker 使用客户端-服务器架构。客户端通过一些命令行与守护进程交互，守护进程管理 Docker 对象，如镜像、容器、网络和 volume。

# docker 三要素

- image：镜像，一个只读模版，其中包含创建 Dokcer 容器的说明。
- container：容器，镜像的可运行实例。可以使用 docker cli 创建/启动/停止/删除/移动容器。同一个镜像，可以生成多个同时运行的容器。容器之间默认相互隔离。
- repository：镜像仓库，存放了各种各样的 image，相当于代码仓库中的 github。

# docker 的基本使用

### 根据镜像启动一个容器

```bash
# 前置
# 使用apt（或其他）下载docker
apt install docker

# 例
# 使用 run命令 启动一个容器 docker run <image name>
docker run ubuntu
```

当系统内没有 ubuntu 镜像时会自动去 docker hub 上拉取后再启动容器。默认`attached`模式，会占用当前窗口并展示实时日志。可以通过`Ctrl + c`停止 docker 服务。更适用于容器和程序的调试阶段。

如果想在后台运行可以（使用`detached`模式）`docker run -d < image name >`

```bash
docker run -d ubuntu

# 查看在后台运行的容器的日志
# docker logs <name or id>
```

### 查看已启动的容器

```
docker ps
# 会打印当前运行的docker容器信息
# id 容器id
# image 镜像名称
# status 状态 up/exited ...
# ports 协议和端口
# names 容器名称
...

```

docker ps 只能查看运行的容器，如果要查看所有容器（包括已停止的）可以加个 `-a`参数:`docker ps -a`

只查看容器 id 可以加个`-q`参数：`docker ps -q`(可以搭配 docker rm 使用 进行批量删除实例： `docker rm $(docker container ps -aq)`)

### 停止/启动/重启/删除一个已启动的容器

```
docker stop <name or id>
docker start <name or id>
docker restart <name or id>
docker rm <name or id> # 删除容器前需要先停止容器。或加上强制删除参数 -f
```

### 开启容器的交互

有时候容器不是一个简单的服务，而是需要交互的操作系统，可以在使用`detached`模式启动容器后使用 exec 进入容器进行交互

```
docker exec -it <name or id> sh
# exec 执行
# -it 交互模式
# sh 使用shell脚本进行交互
```

### 其他

```
#image
docker pull <image name> # 在docker hub中拉取镜像

docker image ls #查看已有的镜像信息

docker image inspect <image id> #查看镜像详细信息，例如镜像id，暴露出的端口，volumn位置等。

# 导出一个已有的镜像，执行后，当前文件夹下会生成一个镜像文件
# save：导出；
# <name:tag>:镜像名称：版本；
# -o: 输出
# fileName：导出后的文件名称
docker image save <name:tag> -o <fileName>

# 导入一个已有的镜像
# <./filename> 要导入的镜像文件路径
docker image load -i <./filename>

```

# 使用 Dockerfile 构建自己的镜像

之前使用的是已创建好的镜像， 我们可以自定义一个 Dockerfile 文件来构建自己的镜像。一般我们会在应用程序根目录与 package.json 同级目录下创建 Dockerfile 文件。

Dockerfile 是一个文本文件，它包含了用户可以在命令行上调用的 docker 构建景象相关的所有命令，它遵循特定的格式和指令集，指令名称和参数以空格分隔，指令名称不区分大小写，但 Docker 约定它们是大写的，以便更容易的区分指令和参数。[其他具体指令可以参考 Dockerfile 语法](https://docs.docker.com/engine/reference/builder/),每个语法指令都会在图像中创建一个图层，当您更改 Dockerfile 并重建映像时，只有那些已更改的图层才会被重建。这也是镜像相对小巧、快速的部分原因。

说几个常用的语法指令：

- ARG: 用来定义参数的命令,可以在打包镜像时使用`docker build --build-arg <varname>=<value>` 命令将变量值传递到 Dockerfile 内部，如果 ARG 指令有默认值并且在构建时没有传递任何值，则构建器使用默认值。

```docker
# ARG <name>[=<default value>]
ARG data
ARG name=zhangsan
ARG age=18
```

- FORM: 在 Dockerfile 里，除去 ARG 指令外，一个有效的 Dockerfile 必须由 FROM 指令开头，用来指定你要构建镜像的基础镜像。FROM 指令前面只能有一个或多个 ARG 指令，用来声明 FROM 行使用的参数。
- WORKDIR：用来定义镜像运行后的工作目录
- COPY: 用来将主机的文件复制到镜像中，`. .` 表示将当前目录复制到容器中。
- CMD：打包好的镜像在启动后默认执行的命令。
- ESPOSE：用来暴露镜像的端口号，告知使用容器的人，当前镜像需要监听的的端口和协议。暴露出来后，可以在镜像打包后，通过 `docker inspect <image id>` 来查看暴露出的端口。

```docker
# cd /app
# vi Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
CMD ["node", "src/index.js"]
EXPOSE 3000
```

### 使用`docker build`根据`Dockerfile`构建镜像

```bash
# 根据Dockerfile构建镜像
# build 构建命令
# -t <image name>通过-t给要打包的镜像取一个名字
# path Dockerfile文件所在路径
# docker build -t <image name> <path:相对路径，根据这个路径寻找Dockerfile文件> .表示它应该在当前目录中查找Dockerfile。

# 例
docker build -t tzzimage .
# 路径是必填项 . 指的是当前路径
```

打包完成后，`docker image ls`查看已有镜像发现 tzzimage 镜像已经存在了。

### 运行构建好的镜像

```
# -d 从后台启动容器
# -p port:port: 指定端口映射，第一个port表示主机要发布的端口，第二个是容器暴露的端口。没有端口映射将无法访问在容器中启动的应用程序。
docker run -dp 3000:3000 tzzimage
```

执行后，就可以访问 http://localhost:3000 了。

还可以通过`docker ps`查看容器状态

### 状态管理

启动后的容器是无状态的，虽然可以容器中创建、更新、删除文件，但随着容器的关闭，相关的数据并不会得到保留，但是在日常使用中，经常会遇到一些需要保留数据的容器，或者在多个容器之间进行数据共享。这就涉及到了容器的数据管理。需要对容器中的数据进行持久化。

- Volume

volume 提供了将容器特定文件到主机的能力，如果在容器启动时挂载容器中的目录，则该目录的更改也会在主机上看到。我们跨容器重新启动并挂载同一目录，我们会看到相同的文件。

使用`docker volume create`命令创建卷。

```bash
docker volume create volume1
```

执行 docker run 命令时添加--mount 选项指定 volumn 挂载，

```bash
docker run -dp 3000:3000 --mount type=volume,src=volume1,target=/app tzzimage
# or
docker run -dp 3000:3000 -v volume1:/app tzzimage

```

表示我们使用了 volume1 volume，并将其挂载到了容器内的/app 目录下，如果该 volume 未被创建时，docker 会自动帮我们创建
在容器启动后，我们在/app 目录下创建一个文件 aaa.txt.

把容器删除后重新指定 volume 创建，会发现 aaa.txt 依旧存在。
通过 docker volume inspect 命令来查看 volume 详情, Mountpoint 属性，是主机磁盘存储数据的实际位置。docker 会把数据存储在这个地方，当我们使用这个 volume 启动容器的时候会把这个目录挂载到容器中

```bash
docker volume inspect todo-db
[
    {
        "CreatedAt": "2023-02-22T09:23:11Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/volume1/_data",
        "Name": "volume1",
        "Options": {},
        "Scope": "local"
    }
]
```

通过这种方式，我们可以持久性的存储应用程序的数据，但在某些情况下，还有另外一种类型的状态管理。

- bind

bind 是另一种挂载方式，他允许你将主机文件系统的目录共享到容器中。在处理应用程序时，您可以使用 bind 方式，将程序源代码挂载到容器中。一旦在容器外部保存文件，容器内就会立即看到您对代码所做的更改。这意味着我们可以在容器中运行进程，监视文件系统更改并响应它们。我们可以搭配着 nodemon，对已启动的程序进行热更新。常常在本地开发调试的时候使用。

```docker
docker run -dp 3000:3000 --mount type=bind,src=/app,target=/app tzzimage
```

