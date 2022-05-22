# Nginx



## 一、Nginx的安装与初步使用

### 1.1 Nginx安装

Nginx下载地址 [Nginx Download Page](http://nginx.org/en/download.html)，下载对应的文件

将文件传入linux服务器/root目录下，通过`tar -zxvf`命令将其解压

<img src="../../assets/nginx解压后文件-16531404571032.png" align="left">

然后进入解压后的文件夹

<img src="images/nginx文件夹内文件.png" align="left">



可以通过 `./configure --prefix=/usr/local/nginx` 编译安装并将其安装在`/usr/local`目录下，但是在这之前有3样东西需要准备前

+ gcc（可以通过`yum install -y gcc`来安装）

+ perl库（通过`yum install -y pcre pcre-devel`命令来安装）

+ zlib库（通过`yum install -y zlib zlib-devel`命令来安装）

接下来执行`make`和`make install`命令进行编译安装

安装后的nginx目录

<img src="images/编译安装后nginx目录.png" align="left">



### 1.2 Nginx启动

进入/usr/local/nginx目录下的sbin目录，然后输入`./nginx`命令后启动nginx服务器

<img src="./images/nginx下的sbin目录.png" align="left">

如果想访问到相关页面，需要关闭防火墙`systemctl stop firewalld.service`也可以开放外网端口`firewall-cmd --zone=public --add-port=80/tcp --permanent`（命令执行完成之后需要重启防火墙才能生效`systemctl restart firewalld.service`）

>注：如果采用开放端口的方式，可以通过`firewall-cmd --list-ports`命令来查看当前开放了哪些端口

然后输入服务器的ip地址，就可以看见`Welcome to nginx!`

<img src="images/nginx主页.png" align="left">

nginx重启或者停止命令

```shell
./nginx 			启动
./nginx -s stop 	快速停止
./nginx -s quit 	优雅关闭，在退出前完成已经接受的连接请求
./nginx -s reload 	重新加载配置（不需要重启nginx服务器，热部署）
```



### 1.3 创建启动脚本

通过脚本文件可以快速启动nginx而不需要到对应的目录下面去执行命令开启

通过命令`vi /usr/lib/systemd/system/nginx.service`在相应文件夹下面创建脚本文件

```shell
[Unit]
Description=nginx - web server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
ExecQuit=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

重新加载系统服务

```
systemctl daemon-reload
```

启动服务 （在启动服务之前如果nginx还在运行，可以通过上面nginx暂停命令将其暂停）

```shell
systemctl start nginx.service
```



查看服务状态

```shell
systemctl status nginx.service
```

<img src="images/nginx服务状态.png" align="left">



 开机启动 

```shell
systemctl enable nginx.service
```



## 二、正向代理、反向代理、负载均衡和动静分离

### 2.1 正向代理

如果把局域网外的Internet想象成一个巨大的资源库，那么局域网中的客户端要访问Internet，则需要通过代理服务来访问，这种代理访问叫做正向代理

例如通过客户端（浏览器）访问Google需要代理服务器，在客户端（浏览器）这边需要配置好代理服务器，然后通过代理服务器来访问Google



### 2.2 反向代理

反向代理，客户端不需要配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，再返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器地址

<img src="images/反向代理.png" align="left">



### 2.3 负载均衡

当访问量十分大的时候，单个服务器无法解决，我们可以增加服务器的数量，将请求平均分发到各个服务器上，将原先请求集中到单个服务器的情况改为请求分发到多个服务器上，就是**负载均衡**



### 2.4 动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度，降低原来单个服务器的压力