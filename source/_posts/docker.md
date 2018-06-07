---
title: Docker
tags: Docker
categories: linux
date: 2017-11-27 15:18:23
---

## 概述
**Docker 是世界领先的软件容器平台。开发人员利用 Docker 可以消除协作编码"在我的机器上可正常工作"的问题。运维人员利用 Docker 可以在隔离容器中并行运行和管理应用，获得更好的计算密度。企业利用 Docker 可以构建敏捷的软件交付管道，以更快的速度、更高的安全性和可靠的信誉为 Linux 和 Windows Server 应用发布新功能。**
**[更加全面的介绍](https://www.docker-cn.com/what-docker)**
<!--more-->

## 容器
**`image`是一个轻量级的，独立的可执行程序包，包含运行一个软件所需的所有东西，包括代码，运行时库，环境变量和配置文件。**
**`container`是`image`的运行时实例 ---- `image`在实际执行时在内存中变成的内容。默认情况下，它与主机环境完全隔离，只有在配置时才访问主机文件和端口。**
**容器(containers)在主机的内核上本地运行应用程序。它们比只能通过管理程序虚拟访问主机资源的虚拟机具有更好的性能特征。容器可以获得本地访问权限，每个容器都以独立的进程运行，不会花费比其他可执行文件更多的内存。**

## 虚拟机 vs. 容器

### 虚拟机架构
![](https://www.docker.com/sites/default/files/VM%402x.png)
**虚拟机运行客户操作系统 ---- 每个框中的操作系统层。这是资源密集型的，产生的磁盘映像和应用程序状态是操作系统设置。系统安装的依赖关系，操作系统安全补丁和其他易于丢失的东西，难以复制重用**

### 容器架构
![](https://www.docker.com/sites/default/files/Container%402x.png)
**容器可以共享一个内核，并且唯一需要在容器镜像中的信息是可执行文件及其包依赖关系(the only information that needs to be in a container image is the executable and its package dependencies)，它们永远不需要安装在主机系统上。这些进程像本地进程一样运行，你可以像`docker ps`一样运行命令来单独管理它们，就像在Linux上运行ps来查看活动进程一样。最后，因为它们包含了所有的依赖关系，所以没有配置纠缠。一个集装箱化的应用程序"随处运行"**

## 安装Docker

### 卸载旧版本
``` bash
$ sudo yum remove docker \
                  docker-common \
                  docker-selinux \
                  docker-engine
```

### 安装Docker CE(Community Edition)
#### 软件库安装
``` bash
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
``` bash
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
``` bash
$ sudo yum install docker-ce
```
#### 开启Docker
``` bash
$ sudo systemctl start docker
```
#### Hello World
``` bash
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
...(snipped)...
```

## Building an app
**现在是开始构建Docker方式的应用程序的时候了。我们将从这个应用程序的层次结构的底部开始，这个应用程序是一个容器。在这个层次上面是一个服务，它定义了容器在生产中的行为方式。最后，在顶层是堆栈，定义了所有服务的交互。**
**过去，如果你要开始编写一个Python应用程序，你的第一步就是在你的机器上安装一个Python运行库。但是，这会造成您的机器上的环境必须如此以使您的应用程序按预期运行。**

**使用Docker，您可以将一个可移植的Python运行时作为一个镜像来获取，无需安装。然后，您的构建可以将基础Python镜像与应用程序代码一起包括在内，确保您的应用程序和依赖项一起运行。**
**这个可移植的镜像被定义为`Dockerfile`**

### `Dockerfile`
**创建一个空的目录，cd进入空目录，创建一个`Dockerfile`文件,`requirement.txt`文件,`app.py`文件**
``` bash
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```
### `requirements.txt`
``` bash
Flask
Redis
```

### `app.py`
``` python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

**使用命令创建一个Docker镜像**
``` bash
$ docker build -t friendlyhello .
$ docker images

REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest              326387cea398
```

## Run the app
**运行应用程序，使用-p将您的机器的端口4000映射到容器的已发布端口80：**
``` bash
$ docker run -p 4000:80 friendlyhello
```
**你应该看到一条消息，Python在`http://0.0.0.0:80`上提供你的应用程序。但是，这个消息来自容器内部，它不知道你将该容器的端口80映射到4000，所以正确的URL为`http://localhost:4000`。**
**在Web浏览器中转到该URL以查看网页上显示的显示内容，包括“Hello World”文本，容器标识和Redis错误消息。**
![](https://docs.docker.com/get-started/images/app-in-browser.png)

**现在让我们以分离模式在后台运行应用程序：**
``` bash
$ docker run -d -p 4000:80 friendlyhello
```

**你得到你的应用程序的长容器ID，然后被踢回你的终端。您的容器正在后台运行。您还可以使用`docker container ls`查看缩略容器标识**
``` bash
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED
1fa4ab2cf395        friendlyhello       "python app.py"     28 seconds ago
```

**现在使用`docker container stop`来结束进程，使用CONTAINER ID，如下所示：**
``` bash
docker container stop 1fa4ab2cf395
```

## Share your image
**为了演示我们刚刚创建的容器的可移植性，我们上传我们构建的映像，并在其他地方运行它。毕竟，当你想将容器部署到生产环境时，你需要学习如何`push`到一个公共地方。**
**如果你还没有Docker帐号，需要在[这里](https://cloud.docker.com/)注册一个帐号**
**本地登录你的Docker库**
``` bash
$ docker login
```

**关联本地镜像像与注册表(registry)中存储库的符号是`username/repository:tag`。该标签是可选的，但推荐使用，因为这是注册管理机构管理Docker镜像版本的机制。给存储库标记有意义的名字，比如`get-started:part2`。这会将镜像放入`get-started`存储库中，并将其标记为`part2`。**
**现在，把它们放在一起来标记镜像。使用您的用户名，存储库和标签名称运行`docker tag image`，以便将镜像上传到您想要的目的地。该命令的语法是:**
``` bash
$ docker tag image username/repository:tag
```
**For example:**
``` bash
$ docker tag friendlyhello john/get-started:part2
```
**查看你新标记的镜像**
``` bash
$ docker images
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
friendlyhello            latest              d9e555c53008        3 minutes ago       195MB
john/get-started         part2               d9e555c53008        3 minutes ago       195MB
python                   2.7-slim            1c7128a655f6        5 days ago          183MB
...
```

**发布镜像**
``` bash
$ docker push username/repository:tag
```

**拉取并运行远端库**
``` bash
docker run -p 4000:80 username/repository:tag
```
### 速查表
``` bash
docker build -t friendlyname .# 使用此目录的 Dockerfile 创建镜像
docker run -p 4000:80 friendlyname  # 运行端口 4000 到 90 的“友好名称”映射
docker run -d -p 4000:80 friendlyname         # 内容相同，但在分离模式下
docker ps                                 # 查看所有正在运行的容器的列表
docker stop <hash>                     # 平稳地停止指定的容器
docker ps -a           # 查看所有容器的列表，甚至包含未运行的容器
docker kill <hash>                   # 强制关闭指定的容器
docker rm <hash>              # 从此机器中删除指定的容器
docker rm $(docker ps -a -q)           # 从此机器中删除所有容器
docker images -a                               # 显示此机器上的所有镜像
docker rmi <imagename>            # 从此机器中删除指定的镜像
docker rmi $(docker images -q)             # 从此机器中删除所有镜像
docker login             # 使用您的 Docker 凭证登录此 CLI 会话
docker tag <image> username/repository:tag  # 标记 <image> 以上传到镜像库
docker push username/repository:tag            # 将已标记的镜像上传到镜像库
docker run username/repository:tag                   # 运行镜像库中的镜像
```


## 服务

### 了解服务
**在分布式应用中，应用的不同部分称为“服务”。例如，假设有一个视频共享网站，它可能提供用于在数据库中存储应用程序数据的服务、用于在用户上传一些内容后在后台进行视频转码的服务、用于前端的服务等。**

**服务实际上是“生产中的容器”。一项服务仅运行一个镜像，但它会编制镜像的运行方式 - 它应使用的端口、容器的多少个从节点应运行才能使服务的容量满足其需求等。扩展服务将更改运行该软件的容器实例数，并将多个计算资源分配给进程中的服务。**

**幸运的是，很容易使用 Docker 平台定义、运行和扩展服务 – 只需编写一个`docker-compose.yml`文件即可。**

### `docker-compose.yml`
``` bash
version:"3"
services:
  web:
    # 将 username/repo:tag 替换为您的名称和镜像详细信息
    image: username/repository:tag
    deploy:
      replicas:5
      resources:
        limits:
          cpus:"0.1"
          memory:50M
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet
networks:
  webnet:
```

**此 docker-compose.yml 文件会告诉 Docker 执行以下操作：**
- 从镜像库中拉取我们上传的镜像。
- 将该镜像的五个实例作为服务`web`运行，并将每个实例限制为最多使用 10% 的 CPU（在所有核心中）以及 50MB RAM。
- 如果某个容器发生故障，立即重启容器。
- 将主机上的端口 80 映射到 web 的端口 80。
- 指示`web`容器通过负载均衡的网络`webnet`共享端口 80。（在内部，容器自身将在临时端口发布到 web 的端口 80。）
- 使用默认设置定义`webnet`网络（此为负载均衡的`overlay`网络）。

### 运行新的负载均衡(load-balanced)的应用
**需要先运行以下命令，然后才能使用`docker stack deploy`命令：**
``` bash
docker swarm init
```
**现在，运行此命令。您必须为应用指定一个名称。在此处该名称设置为`getstartedlab`：**
``` bash
docker stack deploy -c docker-compose.yml getstartedlab
```
**查看您刚才启动的五个容器的列表：**
``` bash
docker stack ps getstartedlab
```
![](/images/docker1.png)

**您可以多次在一行中运行`curl http://localhost`，也可以在浏览器中转至该 URL 并多次点击“刷新”。无论采用哪种方式，您都将看到容器 ID 更改，从而说明负载均衡；借助每项请求，将以循环方式选择五个从节点之一做出响应。**
> **注：在此阶段，容器最多可能需要 30 秒来响应 HTTP 请求。这并不代表 Docker 或 swarm 的性能，而是一项未满足的 Redis 依赖关系，我们稍后将在本教程中讨论此依赖关系。**

### 扩展应用
**您可以通过在`docker-compose.yml`中更改`replicas`值，保存更改并重新运行`docker stack deploy`命令来扩展应用：**
``` bash
docker stack deploy -c docker-compose.yml getstartedlab
```
**Docker 将执行原地更新，而无需先清除技术栈或终止任何容器。**
**现在，重新运行`docker stack ps`命令以查看经过重新配置的已部署实例。例如，如果您扩展了从节点，将有更多处于运行状态的容器。**

### 清除应用和 swarm
**使用`docker stack rm`清除应用：**
``` bash
docker stack rm getstartedlab
```
**这将删除应用，但我们的单节点`swarm`仍处于正常运行状态（如`docker node ls`所示）。使用`docker swarm leave --force`清除 swarm。**

### 速查表
``` bash
docker stack ls              # 列出此 Docker 主机上所有正在运行的应用
docker stack deploy -c <composefile> <appname>  # 运行指定的 Compose 文件
docker stack services <appname>       # 列出与应用关联的服务
docker stack ps <appname>   # 列出与应用关联的正在运行的容器
docker stack rm <appname>                             # 清除应用
```

## Swarm
引用[Docker中文文档 Swarm](https://docs.docker-cn.com/get-started/part4/)

## 技术栈
引用[Docker中文文档 技术栈](https://docs.docker-cn.com/get-started/part5/)

## 引用链接
- [Docker中文](https://www.docker-cn.com/what-docker)
- [Get Docker CE for CentOS](https://docs.docker.com/engine/installation/linux/docker-ce/centos/)
- [Docker官网文档](https://docs.docker.com/get-started/)
- [Docker 服务](https://docs.docker-cn.com/get-started/part3/)