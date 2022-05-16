# Docker
[TOC]
## 一、Docker命令
### 1. 服务相关命令
+ 启动docker， `systemctl start docker`
+ 查看docker服务状态，`systemctl status docker`
+ 停止docker服务，`systemctl stop docker`
+ 重启docker服务，`systemctl restart docker`
+ 开机启动docker，`systemctl enable docker`

### 2. 镜像相关命令
#### （1）查看镜像
+ 查看本地镜像文件，`docker images`
+ 查看本地所有镜像的id，`docker images -q`
#### （2）搜索镜像
+ 搜索镜像，`docker search ***`。例如搜索redis镜像，可以使用命令`docker search redis`
#### （3）拉取镜像
+ 拉取（下载）镜像，`docker pull ***`。例如拉取redis镜像，可以使用命令`docker pull redis`（如果redis后面没有版本号，默认下载最新版本），下载具体的版本，需要在后面加上版本号，例如下载5.0版本，`docker pull redis:5.0`
+ **tips**:要下载具体的版本，可以去Docker Hub的官网（https://hub.docker.com/）搜索有哪些版本

#### （4）删除镜像

+ 删除镜像，第一种方式，通过IMAGE ID进行删除`docker rmi imageId`(rmi = remove image)，imageId可以通过`docker images`命令查到；如果两个镜像的IMAGE ID相同，则使用第二种方式，第二种方式通过名字进行删除，例如删除redis5.0版本的镜像，`docker rmi redis:5.0`
+ 删除所有镜像，使用命令 docker rmi \`docker images -q\`

### 3. 容器相关命令
#### (1) 查看容器
+ 查看正在运行的容器，`docker ps`
+ 查看运行过的以及正在运行的容器，`docker ps -a`
#### (2) 创建容器

+ 创建容器 `docker run -it --name=c1 imageName:version /bin/bash`， 其中`-i`表示以交互模式运行容器（进入docker容器内容，而`-d`命令并不会进入容器），`-t`表示给容器单独分配一个终端， `-d`表示后台运行容器，并返回容器ID，`--name=`表示给容器指定一个名字，后面的名字可以随便输入，最后面跟上镜像名字以及其对应的版本
+ 退出容器，`exit`，如果是以`-d`的参数形式创建容器，`exit`命令执行后容器并不会停止

#### (3) 进入容器

+ 进入容器，`docker exec -it name /bin/bash`，name处填写创建容器时设置的name值

#### (4) 启动容器

+ 启动容器，`docker start name`，name填写创建容器时容器的名字

#### (5) 停止容器

+ 停止容器，`docker stop name`， name填写创建的容器名字

#### (6) 删除容器

+ 删除容器，`docker rm id/name`，最后面可以填写容器的ID**或者**name

#### (7) 查看容器信息

+ 查看具体某个容器的信息，`docker inspect name`，name处填写容器的name

## 二、容器数据卷
### 1. 数据卷的概念以及作用
+ 宿主机（运行docker的机器）中的一个目录或文件，当容器目录和数据卷目录绑定后，对方的修改会立刻同步（这样解决了Docker容器删除后，防止在容器中的数据被销毁 ）
+ 一个数据卷可以被多个容器同时挂载
+ 一个容器也可以挂载多个容器
### 2. 配置数据卷
+ 创建容器时，使用`-v` 参数设置数据卷， `docker run ... -v 宿主机目录（文件）：容器内目录（文件）`，这个时候命令里宿主机的目录或文件就称为**数据卷**，例如`docker run -it --name=c1 -v /root/data:/root/data_container centos:7 /bin/bash`
	
	tips：宿主机目录必须是绝对路径，目录不存在会自动创建，可以挂载多个数据卷(具体做法是在后面跟上多个`-v`)
### 3. 数据卷容器
+ 创建数据卷容器，`docker run -it --name=c3 -v /volume centos:7 /bin/bash`
+ 创建容器挂载数据卷容器，`docker run -it --name=c1 --volumes-from c3 centos:7 /bin/bash`