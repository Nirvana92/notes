docker下载安装[mac]

`下载网址:https://download.docker.com/mac/stable/Docker.dmg`

`配置镜像加速`

```
点击鲸鱼图标->Preferences..->Docker Engine ->在配置文件中添加registry-mirrors -> 点击应用

如下: 
{
  "experimental": false,
  "debug": true,
  "registry-mirrors": [
    "https://kfp63jaj.mirror.aliyuncs.com"
  ]
}
```

`验证配置是否生效: docker info`

```
 ....
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
  //  配置成功
 Registry Mirrors:
  https://kfp63jaj.mirror.aliyuncs.com/
 Live Restore Enabled: false
 Product License: Community Engine
```

##### 镜像加速

- 网易: `https://hub-mirror.c.163.com/`
- 阿里云: `https://<你的ID>.mirror.aliyuncs.com`
- 七牛云加速器: `https://reg-mirror.qiniu.com`

##### docker hello-world

```
// 拉取hello-world 镜像
> docker pull hello-world
> docker images
// 运行hello-world
> docker run hello-world
```

##### docker 架构

- 应用程序
- Docker Engine(Docker 引擎)
  - docker daemon `server`
  - rest api `http协议`
  - docker CLI `client`
- 可用资源(物理机)

##### 容器与镜像

镜像: windows 7 光盘. 只读的文件, 提供了运行程序完整的软硬件资源。是应用程序的集装箱

容器: 安装了win7 的pc机. 镜像的实例, 由docker创建。容器之间彼此隔离。

##### docker执行流程

- `client:` docker build, docker pull, docker run
- `docker daemon:` 镜像存储在本地, 通过客户端执行run 运行.
- `registry:` 远程镜像仓库



##### docker 常用命令  `https://hub.docker.com/`

- `docker pull 镜像名<:tags>` - 从远程仓库抽取镜像
- `docker images` -查看本地镜像
- `docker run 镜像名<:tags>` -创建容器, 启动应用
- `docker ps` - 查看正在运行中的镜像
- `docker rm <-f> 容器id` -删除容器, -f 强制删除
- `docker rmi <-f> 镜像名:<tags>` -删除镜像

##### docker宿主机与容器通信

`宿主机与容器端口绑定:`

物理机, 宿主机[8000] = docker tomcat:latest 容器: 8080

`映射方法: `

```
docker run -p 8000<宿主机端口>:8080<容器端口> tomcat

访问: localhost:8000
```

**问题**: 运行之后访问`localhost:8000` 报404

**处理**: 

```
> docker exec -it <容器id> /bin/bash
> mv webapps.dist webapps [原项目文件跑到webapps.dist中了]
```

**后台运行: ** `docker run -p 8000:8080 -d tomcat`

**停止运行:**  `docker rm -f d39774da3b0a`

**删除镜像: ** `docker rmi <镜像id>`



##### 容器内部结构

以tomcat:lastest 为例: 

- Linux[Red Hat]
- Jdk[1.8]
- Apache Tomcat [8]

**在容器中执行命令: ** `docker exec [-it] 容器id 命令`

`-it: 采用交互方式执行命令`

如: `docker exec -it d39774da3b0a /bin/bash`



##### 容器的生命周期

`docker ps -a : 查看created 的容器`



##### 容器间单向通信

```
// 创建容器的时候添加 --link mysql-slave-1[别名]

> docker run --name mysql-master --link mysql-slave-1 -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7

> docker run --name mysql-master --link mysql-master -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7

// 添加完之后, 进入mysql-master 直接 ping mysql-slave-1 可以通
```



##### 容器间双向通信[Bridge]

```
// 查看docker 的网络信息
> docker network ls

// 创建网桥
> docker network create -d bridge mybridge

// 绑定网桥[绑定之后, 两个mysql-master 直接可以互通]
> docker network connect mybridge mysql-master
> docker network connect mybridge mysql-slave-1
```



##### Dockerfile 构建镜像

- Dockerfile 是一个包含用于组合镜像的命令的文本文档
- Docker 通过读取Dockerfile中的指令按步自动生成镜像
- `docker build -t 机构/镜像名<:tags> Dockerfile目录`

构建项目, 自动发布到docker中的tomcat 中运行: 

```
1. 新建first-docker-web 文件夹

2. 新建 docker-web 文件夹
添加 hello.html 文件, 编写测试代码

3. 新建 Dockerfile, 添加如下配置
FROM tomcat:latest
MAINTAINER nirvana
# 切换工作目录, 不存在则创建
WORKDIR /usr/local/tomcat/webapps
# 本地的docker-web 项目发不到 workdir 中的docker-web[不存在创建]
ADD docker-web ./docker-web

4. 将first-docker-web 文件夹放到docker 的images目录中[/var/lib/docker] mac 环境有出入

5. 进入到 /var/user/images/first-docker-web 目录中

6. docker build -t nirvana/mywebapp:1.0 .

7. docker images #查看镜像

8. docker run -d -p 8000:8080 nirvana/mywebapp:1.0 # 创建容器

```

##### 镜像分层(Layer) 概念

构建的时候会有多个临时镜像, 当镜像版本迭代的时候前面步骤不变, 则使用cache



##### 容器间单向通信

```
场景: tomcat 和mysql 需要通信

> docker run -d --name web tomcat
> docker run -d --name database -it centos /bin/bash

查看docker 虚拟ip[查看原始信息] 
> docker inspect 357d94c8581a

// 添加--link 参数
> docker run -d --name web --link database tomcat
// 在进入tomcat 命令行之后, 使用ping database 可通
```



**容器间的数据共享**

作用: 在宿主机上开辟一个路径, 路径中的文件共容器使用

背景: 比如成千上万的tomcat容器, 里面的webapps 内容做了修改, 不能遍历去修改每个docker中的tomcat中的webapps内容.可以共享volumns中的文件内容.

方法一: 

格式: 

```shell
> docker run --name 容器名 -v 宿主机路径:容器内挂载路径 镜像名
```

实例: 

```
> docker run --name t1 -v /usr/webapps:/usr/local/tomcat/webapps tomcat:lastest
```

方法二: 

创建共享容器: 

```
> docker create --name webpage -v /webapps:/tomcat/webapps tomcat /bin/true
```

共享容器挂载点: 

```
> docker run --volumes-from webpage --name t1 -d tomcat
```



**容器编排工具[docker compose]: **

docker compose 只支持单机多容器部署工具

docker compose 安装: `https://docs.docker.com/compose/install/`

安装: 

```shell
> curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

> chmod +x /usr/local/bin/docker-compose
```

上述方法下载很慢, 可以使用如下方法下载: 

```
curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```



使用docker-compose 部署wordpress 项目: 

参考: `https://docs.docker.com/compose/wordpress/`

新建文件夹: `my-wordpress`

新建docker-compose.yml文件: 

```
version: '3.3'

services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
volumes:
    db_data: {}
```

执行命令: `docker-compose up -d`



#### Docker上磁盘空间不足问题处理

**环境: **

mac + virtual box + centos7

**背景: **

在准备搭建 mgr的时候提示: docker: Error response no space left on device.



在虚拟机中查看磁盘使用情况: `df -hl /var/lib/docker` 发现磁盘使用 100%



**处理流程: **

```
1. 关闭vb
2. 查看centos7磁盘存储路径的信息 
	[/Users/mac/VirtualBox VMs/centos-7/centos-7.vdi]
3. 查看centos7 的信息, VBoxManage list hdds
4. 调整磁盘大小: VBoxManage modifyhd 0276e01a-ac0b-4531-bcbe-74702f15623c --resize 40960
```



还需要对centos 的虚拟空间转换为实际空间: 

参考: `https://www.cnblogs.com/straycats/p/11261364.html`



