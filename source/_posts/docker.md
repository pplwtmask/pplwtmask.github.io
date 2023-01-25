---
title: Docker
date: 2021-11-28 15:38:06
---

## docker
[官方手册](https://docs.docker.com/desktop/)
### 基本介绍
**镜像**（Image）和**容器**（Container）的关系，就像是java中的 类 和 实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。
同镜像使用的是分层存储，容器也是如此。每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为**容器存储层**。
容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。
按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用 数据卷（Volume）、或者 绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。
数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。
**仓库**（Repository），通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。
仓库名经常以 两段式路径 形式出现，比如 jwilder/nginx-proxy，前者往往意味着 Docker Registry 多用户环境下的用户名，后者则往往是对应的软件名。

### 安装
[安装教程](https://docs.docker.com/engine/install/#server)

### dockerfile

```
FROM debian:stretch

RUN set -x; buildDeps='gcc libc6-dev make wget' \
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

#### FROM 指定基础镜像
FROM 就是指定 基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。
除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 scratch。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。

#### RUN 执行命令
每一条指令(Instruction)构建一层，因此每一条指令的内容，就是描述该层应当如何构建，因此不应该写多个run

#### WORKDIR 指定工作目录（当前目录）
以后各层的当前目录就被改为指定的目录，如该目录不存在，WORKDIR 会帮你建立目录。
错误示例：以下操作并不会在/app下创建world.txt
```
RUN cd /app   //这一层进去/app目录下
RUN echo "hello" > world.txt      //在当前目录（工作目录）下创建world.txt
```

#### COPY 复制上下文文件到工作目录
包含元数据，比如读、写、执行权限、文件变更时间等。
在 Docker 官方的 [Dockerfile 最佳实践文档](https://yeasy.gitbook.io/docker_practice/appendix/best_practices) 中要求，尽可能的使用 COPY，因为 COPY 的语义很明确，就是复制文件而已，而 ADD 则包含了更复杂的功能，其行为也不一定很清晰。最适合使用 ADD 的场合，就是所提及的需要自动解压缩的场合。
```
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

#### CMD 指定容器启动时，执行的命令
在运行时可以指定新的命令来替代镜像设置中的这个默认命令，比如`docker run -it ubuntu cat /etc/os-release`,`cat /etc/os-release`代替了`/bin/bash`
`CMD ["可执行文件", "参数1", "参数2"...]`，推荐使用这种格式，shell脚本格式会被解析成`CMD [ "sh", "-c", "echo $HOME" ]`
>注意：对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出。

#### ENTRYPOINT 给容器启动命令指定参数
CMD 的内容作为参数传给 ENTRYPOINT 指令，`<ENTRYPOINT> "<CMD>"`,启动容器的时候可以传递CMD指定覆盖dockerfile中定义的CMD指令。

#### ENV 、ARG 
ARG指定的变量只能被下一层使用，在build的时可以通过参数`--build-arg`覆盖

#### EXPOSE 声明暴漏端口
* 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射
* 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口

#### HEALTHCHECK 健康检查
```
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
```
`docker inspect`可以来查看日志



>注意：CMD、ENTRYPOINT、HEALTHCHECK 只可以出现一次，如果写了多个，只有最后一个生效。


#### 构建镜像
```
$ docker build -t nginx:v3 .
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM nginx
 ---> e43d811ce2f4
Step 2 : RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
 ---> Running in 9cdc27646c7b
 ---> 44aa4490ce2c
Removing intermediate container 9cdc27646c7b
Successfully built 44aa4490ce2c
```
`.`是指定上下文，构建镜像时会将上下文上传给docker引擎，通过上下文拿到构建所需的所有文件。
`-f ../Dockerfile.php`可以指定docker文件的位置



### docker常用命令

启动容器：
#### run
```
$ docker run -it --rm ubuntu:18.04 bash

root@e7009c6ce357:/# cat /etc/os-release
NAME="Ubuntu"
VERSION="18.04.1 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.1 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
```
* it：这是两个参数，一个是 -i：交互式操作，一个是 -t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。
* --rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 --rm 可以避免浪费空间。
* ubuntu:18.04：这是指用 ubuntu:18.04 镜像为基础来启动容器。
* bash：放在镜像名后的是 命令，这里我们希望有个交互式 Shell，因此用的是 bash。
* -d: 此时容器会在后台运行并不会把输出的结果 (STDOUT) 打印到宿主机上面(输出结果可以用 `docker logs [container ID or NAMES]`查看)。


进入容器:
#### attach
```
$ docker run -dit ubuntu
243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550

$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
243c32535da7        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           nostalgic_hypatia

$ docker attach 243c
root@243c32535da7:/#
```
注意：如果从这个 stdin 中 exit，会导致容器的停止。（不推荐使用）

#### exec
docker exec 后边可以跟多个参数，这里主要说明 -i -t 参数。
只用 -i 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。
当 -i -t 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。
```
$ docker run -dit ubuntu
69d137adef7a8a689cbcb059e94da5489d3cddd240ff675c640c8d96e84fe1f6

$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
69d137adef7a        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           zealous_swirles

$ docker exec -i 69d1 bash
ls
bin
boot
dev
...

$ docker exec -it 69d1 bash
root@69d137adef7a:/#
```


导入导出：
#### export、import、load
```
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
7691a814370e        ubuntu:18.04        "/bin/bash"         36 hours ago        Exited (0) 21 hours ago                       test
$ docker export 7691a814370e > ubuntu.tar
---------------------------------------------------------------------------------------------------------------------------------------------
$ cat ubuntu.tar | docker import - test/ubuntu:v1.0
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
test/ubuntu         v1.0                9d37a6082e97        About a minute ago   171.3 MB
$ docker import http://example.com/exampleimage.tgz example/imagerepo
```
>注：用户既可以使用 docker load 来导入镜像存储文件到本地镜像库，也可以使用 docker import 来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。

### 删除
```
$ docker container rm trusting_newton
trusting_newton
```
`-f` 参数。Docker 会发送 SIGKILL 信号给容器.
删除所有终止状态的容器
```
$ docker container prune
```








[参考手册](https://yeasy.gitbook.io/docker_practice/)
[参考手册](https://vuepress.mirror.docker-practice.com/)


























