# Docker
[TOC]
## 一、Docker命令
### 1. 服务相关命令
+ 启动docker， `systemctl start docker`
+ 查看docker服务状态，`systemctl status docker`
+ 停止docker服务，`systemctl stop docker`
+ 重启docker服务，`systemctl restart docker`
+ 开机启动docker，`systemctl enable docker`
+ docker概要信息，`docker info`
+ docker命令文档，`docker --help`
+ docker具体命令的帮助文档，`docker 具体命令 --help`

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

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

Options:

| Name, shorthand        | Default | Description                                        |
| ---------------------- | ------- | -------------------------------------------------- |
| `--detach` , `-d`      |         | Run container in background and print container ID |
| `--interactive` , `-i` |         | Keep STDIN open even if not attached               |
| `--name`               |         | Assign a name to the container                     |
| `--tty` , `-t`         |         | Allocate a pseudo-TTY                              |
+ 创建容器 `docker run -it --name=c1 imageName:version /bin/bash`， 其中`-i`表示以交互模式运行容器（进入docker容器内容，而`-d`命令并不会进入容器），`-t`表示给容器单独分配一个终端， `-d`表示后台运行容器，并返回容器ID，`--name=`表示给容器指定一个名字，后面的名字可以随便输入，最后面跟上镜像名字以及其对应的版本
+ 退出容器，`exit`，如果是以`-d`的参数形式创建容器，`exit`命令执行后容器并不会停止

#### (3) 进入容器

+ 进入容器，`docker exec/attach -it name /bin/bash`，name处填写创建容器时设置的name值
  + exec是在容器中打开新的终端，用 exit 命令退出不会导致容器停止
  + attach直接进入容器的终端，用 exit 命令退出会导致容器的停止


#### (4) 启动容器

+ 启动容器，`docker start name`，name填写创建容器时容器的名字

#### (5) 停止容器

+ 停止容器，`docker stop name`， name填写创建的容器名字

#### (6) 删除容器

+ 删除容器，`docker rm id/name`，最后面可以填写容器的 ID **或者** name

#### (7) 查看容器信息

+ 查看具体某个容器的信息，`docker inspect name`，name 处填写容器的name

#### (8) 导入/导出容器

+ 导出容器： export命令，`docker export CONTAINERID > filename.tar`
+ 导入称为image文件：import，`cat filename.tar | docker import - IMAGE User/Image Name:Version`，例如`cat a.tar | docker import - Ecifics/ubuntu:1.1.1`



## 二、容器镜像

### 2.1 相关概念

每一个容器都是一个简易版的Linux环境和运行在其中的应用程序。

Docker镜像实际上是一层一层的文件系统组成的，这种层级文件系统叫做 UnionFS 。

Docker镜像的最底层是引导文件系统 bootfs， bootfs 主要包含 bootloader 和 kernel， Linux启动时会加载bootfs文件系统， 然后通过bootloader 加载 Kernel 到内存中，之后内存的使用权从 bootfs 转交给了内核， 此时系统会卸载掉 bootfs。

Docker 镜像层都是只读的，而容器层是可写的。当容器启动时，一个新的可写层加载到镜像的顶部。

Docker 镜像分层类似于 Java 中子类继承父类，子类在父类的基础上进行扩展。新镜像则是在 bootfs 和 Base Image 的基础上进行扩展，例如一个ubuntu 镜像文件创建的容器，只有最基本的命令，没有 vim ，如果我们需要 vim，需要进行安装，相当于在原来的 Base Image 上加上了一层镜像，并且可以按照此时的镜像状态创建 Image，此时通过该 Image 创建的容器就自带 vim了。



### 2.2 通过容器创建镜像

Docker命令：`docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]`，Create a new image from a container’s changes

```shell
[root@VM-0-16-centos ~]# docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS              NAMES
c3f279d17e0a        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            desperate_dubinsky
197387f1b436        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            focused_hamilton
```

```shell
[root@VM-0-16-centos ~]# docker commit c3f279d17e0a  svendowideit/testimage:version3

f5283438590d
```

```shell
[root@VM-0-16-centos ~]# docker images

REPOSITORY                        TAG                 ID                  CREATED             SIZE
svendowideit/testimage            version3            f5283438590d        16 seconds ago      335.7 MB
```





## 三、容器数据卷
### 1. 数据卷的概念以及作用
+ 宿主机（运行docker的机器）中的一个目录或文件，当容器目录和数据卷目录绑定后，对方的修改会立刻同步（这样解决了Docker容器删除后，防止在容器中的数据被销毁 ）
+ 一个数据卷可以被多个容器同时挂载
+ 一个容器也可以被挂载多个数据卷
### 2. 配置数据卷
+ 创建容器时，使用`-v` 参数设置数据卷， `docker run ... -v 宿主机目录（文件）：容器内目录（文件）`，这个时候命令里宿主机的目录或文件就称为**数据卷**，在容器内对应目录下的修改，会同步到宿主机对应目录中，例如`docker run -it --name=c1 -v /root/data:/root/data_container centos:7 /bin/bash`
	
	> tips: 
	> + 上面这种创建数据卷的方式默认是容器可读可写，相当于 `docker run -it --name=c1 -v /root/data:/root/data_container:rw centos:7 /bin/bash`，如果只想让容器只读，可以改成 ro （read only） `docker run -it --name=c1 -v /root/data:/root/data_container:ro centos:7 /bin/bash`，此时宿主机可以在宿主机目录下进行读写操作，但是容器无法在对应文件夹下面做写操作（例如创建文件、向文件中写入内容等）
	> + 宿主机目录必须是绝对路径，目录不存在会自动创建，可以挂载多个数据卷(具体做法是在后面跟上多个`-v`)
	
+ 如果忘记了当时绑定的是哪两个目录，可以通过 `docker inspect 容器ID` 命令查找，可以在 Mounts 部分找到绑定的两个目录
### 3. 容器卷的绑定

+ 创建容器时，需要加上`--volumes-from 绑定容器名`，例如`docker run -it --privileged=true --volumes-from c1 centos:7 /bin/bash`，此时新建容器的和绑定容器的目录配置相同，并且对其中任何一个容器的修改，都会同步到另一个容器中。并且如果其中一个容器和主机目录进行了映射，其中一个容器挂了，那么主机目录下的修改也会同步到另一个正在运行的容器中



## 四、Dockerfile

