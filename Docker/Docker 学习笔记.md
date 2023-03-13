<h1>Docker 学习笔记</h1>

# Docker概述

开发、上线两套环境，应用环境、应用配置

集群

隔离，docker核心思想，打包装箱，每个箱子是相互隔离的

轻巧

基于Go语言开发，开源



虚拟机技术：虚拟出一套硬件，运行一个完整的操作系统，然后在这个系统上安装和运行软件。资源占用多、冗余步骤多、启动慢

容器化技术：容器内的应用直接运行在宿主机的内核上，容器是没有自己的内核的，也没有虚拟硬件，所以就轻便

每个容器是互相隔离，每个容器内都有一个属于自己的文件系统，互不影响。



DevOps 开发、运维

**应用更快速的交付和部署**，打包镜像发布测试，一键运行

**更便捷的升级和扩容**

**更简单的系统运维**

**更高效的计算资源利用**

# Docker安装

镜像image

容器container

仓库respository

~~~shell
# 卸载旧版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# 需要的安装包
yum install -y yum-utils

# 设置镜像仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
# *可选 更新软件包索引
yum makecache fast

# docker安装 ce 社区版 ee 企业版
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 启动docker
systemctl start docker

# 查看docker版本号，是否安装成功
docker version

----------------------------------------------------------------------
----------------------------------------------------------------------

# hello world
docker run hello-world

# 查看镜像
docker images

# 卸载docker，卸载依赖
yum remove docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-ce-rootless-extras
# 卸载docker，删除资源，/var/lib/docker默认工作路径
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
~~~



~~~shell
# 刷新配置文件
systemctl daemon-reload

# 重启
systemctl restart docker
~~~

## 底层原理

Docker是一个Client-Server结构的系统，Docker的守护进程运行在宿主机上，通过Socket从客户端访问。

Docker-Server接收到Docker-Client的指令，就会执行这个命令



Docker为什么比VM快

1、docker有着比虚拟机更少的抽象层

2、docker利用的是宿主机的内核，vm需要guest os

所以说，新建一个容器的时候，docker不需要向虚拟机一样加载一个操作系统内核，避免引导。虚拟机是加载guest os，分钟级别。而docker是利用宿主机的操作系统，省略了这个复杂的过程，秒级。

# Docker命令

~~~shell
# 显示docker的版本信息
docker version

# 显示docker的系统信息，包括镜像和容器的数量
docker info

# 帮助文档
docker 命令 --help

# 查看cpu的状态
docker stats

# 查看网络状态
docker network
~~~

## 镜像命令

~~~shell
# 查看宿主机上的镜像
docker images
# 显示所有镜像
docker images -a
# 只显示镜像的id
docker images -q

# 搜索镜像
docker search mysql
docker search mysql --filter=STARS=10000

# 下载镜像，分层下载
docker pull mysql

# 删除镜像
docker rmi -f 镜像id/镜像名称
# 删除多个镜像
docker rmi -f 镜像id 镜像id 镜像id
# 删除全部镜像
docker rmi -f $(docker images -aq)

# 查看镜像元数据
docker image inspect 镜像id:镜像版本
~~~

## 容器命令

~~~shell
# 新建容器并启动
docker run [可选参数] image
--name	容器名字，用来区分容器
-d		后台方式运行
-it		使用交互方式运行，可进入容器查看内容
-p		端口映射
	-p ip:主机端口:容器端口
	-p 主机端口:容器端口 (常用)
	-p 容器端口
	容器端口
	
-P		随机指定端口
--rm	一般用于测试，用完即删
-e		环境配置
-v		数据卷挂载

docker run -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql
docker run -it centos /bin/bash

# 列出所有运行的容器
docker ps
		当前正在运行的容器
-a		当前正在运行的容器+历史运行过的容器
-n=?	显示最近创建的容器，?指定数量
-q		只显示容器id

# 退出容器
exit			直接退出容器并停止
Ctrl + P + Q	只退出，不停止

# 删除容器，不能删除正在运行的容器，如果要强制删除，rm -f
docker rm 容器id
# 删除全部容器
docker rm -f $(docker ps -aq)
docker ps -a -q|xargs docker rm

# 启动停止容器
docker start 容器id
docker restart 容器id
docker stop 容器id
docker kill 容器id

# 查看日志
docker logs
-t				显示日志
-f				显示时间戳
--tail number	要显示的日志条数

# 查看容器中的进程信息
docker top 容器id

# 查看容器元数据
docker inspect 容器id

# 进入当前正在运行的容器
docker exec -it 容器id bashshell		# 进入容器后开启一个新的终端
docker attach 容器id					# 进入容器正在执行的终端，不会启动新的进程

# 从容器内拷贝文件到宿主机
docker cp 容器id:容器内路径 目的主机路径

# 查看容器元数据
docker inspect 容器id
~~~



~~~shell
docker run -d centos

# 问题，docker ps，发现centos停止了
# 常见的坑，docker容器使用后台运行，就必须要有一个前台进程，docker发现没有应用，就会自动停止
# nginx，容器启动后，发现自己没有提供服务，就会立即停止，就是没有程序了

docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node"  -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:tag
~~~

## 操作命令



## 可视化

* portainer	docker图形化界面管理工具

~~~shell
docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
~~~



* Rancher

# Docker镜像

## 镜像是什么

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，他包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。

## Docker镜像加载原理

UnionFS	联合文件系统

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统就是UnionFS

bootFS 	系统启动需要引导加载

rootFS



Docker镜像默认都是只读的，当容器启动时，一个新的可写层加载到镜像的顶部。

这一层就是我们通常说的容器层，容器之下的都叫镜像层。

所有操作都是基于容器层的

## commit镜像

~~~shell
docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名:[TAG]
~~~

# 容器数据卷

容器之间可以有一个数据共享的技术，Docker容器中产生的数据，同步到本地

卷技术，目录的挂载，将我们容器内的目录，挂载到Linux目录上

**容器的持久化和同步操作，容器间也是可以数据共享的（多个容器挂载同一目录）**

## 使用数据卷

>  直接使用命令来挂载  -v

~~~shell
docker run -it -v 主机目录:容器目录
~~~

## MySQL的数据持久化

~~~shell
docker pull mysql:5.7

docker run -d --name mysql -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root mysql:5.7
~~~

## 匿名、具名挂载

~~~shell
# 匿名挂载
-v 容器目录

# 具名挂载
-v 卷名:容器目录

# 指定路径挂载
-v /宿主机目录:容器目录

# 查看所有卷的情况
docker volume ls

# 查看卷路径
docker volume inspect 卷名
~~~

所有docker容器内的卷，没有指定目录的情况下，都是在 /var/lib/docker/volumes/xxx/_data 目录下

~~~shell
# 改变读写权限 只读，这个目录只能通过宿主机操作，容器内部无法操作
-v 容器目录:ro
# 可读可写
-v 容器目录:rw
~~~

##  DockerFile挂载



## 数据卷容器

~~~shell
--volumes-from
~~~

删除一个容器，其他容器依旧可以使用共享卷

拷贝的概念

容器之间配置信息的传递，数据卷容器的生命周期一直持续到没有容器使用未为止

但是一旦持久化到了本地，本地的数据是不会删除的

# DockerFile

用来构建docker镜像的文件，命令参数脚本

通过这个脚本可以生成镜像，镜像是一层一层的，脚本是一个一个的命令，每个命令都是一层

## DockerFile 构建过程

**基础知识**

* 每个保留关键字（指令）必须是大写字母

* 指令从上到下顺序执行
* \# 表示注释
* 每一个指令都会创建提交一个新的镜像层，并提交



DockerFile是面向开发的，我们以后要发布项目，做镜像，就需要编写DockerFile文件

Docker镜像逐渐成为企业交付的标准

DockerFile：构建文件，定义了一切的步骤，即代码

Dockerlmages：通过DockerFile构建生成的镜像，最终发布和运行的产品

Docker容器：容器就是镜像运行起来提供服务的



基础镜像 scratch

## DockerFile指令

~~~dockerfile
FROM			# 基础镜像，一切从这里开始构建
MAINTAINER		# 镜像作者，姓名<邮箱>
RUN				# 镜像构建的时候需要执行的命令
ADD				# 步骤，添加内容，会自动解压
COPY			# 复制，将文件拷贝到镜像中
WORKDIR			# 镜像工作目录
VOLUME			# 挂载的目录
EXPOSE			# 暴露端口
ENV				# 构建时设置环境变量
CMD				# 指定容器启动时需运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT		# 指定容器启动时需运行的命令，可以追加命令，直接拼接在ENTRYPOINT命令的后面
ONBUILD			# 构建一个被继承DockerFile时运行，是一个触发指令
~~~



~~~shell
# 构建镜像，DockerFile文件名字随意，官方命名 Dockerfile，build 会自动寻找这个文件，就不需要 -f 指定了
docker build -f Dockerfile文件地址 -t 镜像名:[TAG] .

# 查看镜像构建历史
docker history 镜像id
~~~



~~~dockerfile
FROM openjdk:8
EXPOSE 8080
COPY application.yml /app/application.yml
COPY logback-spring.xml /app/logback-spring.xml
COPY test-1.0.0.1.jar /app/test.jar
ENV APP_OPTS=""
ENV TZ="Asia/Shanghai"
WORKDIR /app
ENTRYPOINT [ "java", "-Djava.awt.headless=true", "-jar","/app/test.jar" ]
~~~

## 发布镜像

~~~shell
# 登录
docker login -u 用户名

# 给镜像生成版本号 作者/镜像名
docker tag 镜像id 镜像名:[TAG]

# 发布镜像
docker push 镜像名:[TAG]
~~~

提交的时候也是按照镜像的层级来提交的

DockerFile：构建文件，定义了一切的步骤，源代码

Dockerlmages：通过 DockerFile 构建生成的镜像，最终发布和运行的产品，可能是jar war

## 虚悬镜像

仓库名、标签都是\<none>的镜像，俗称dangling image

~~~shell
# 创建
docker build .

# 单独查看
docker image ls -f dangling=true

# 删除
docker image prune
~~~

# Docker网络原理

## Docker0

~~~shell
[root@localhost /]# ip addr
# lo：本机回环地址
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
# ens33：内网地址
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:65:49:26 brd ff:ff:ff:ff:ff:ff
    inet 192.168.127.131/24 brd 192.168.127.255 scope global noprefixroute dynamic ens33
       valid_lft 1541sec preferred_lft 1541sec
    inet6 fe80::9581:3f83:6568:8151/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
# docker0：docker地址
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:2d:3b:09:1c brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:2dff:fe3b:91c/64 scope link
       valid_lft forever preferred_lft forever
~~~



~~~shell
[root@localhost /]# docker exec -it tomcat ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
# docker分配的ip地址
28: eth0@if29: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
       
# linux 可以 ping通docker容器内部
# 容器和容器之间是可以互相ping通的
# 所有的容器，不指定网络的情况下，都是docker0来路由的，docker会给容器分配一个默认的可用ip
~~~



~~~shell
[root@localhost /]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:65:49:26 brd ff:ff:ff:ff:ff:ff
    inet 192.168.127.131/24 brd 192.168.127.255 scope global noprefixroute dynamic ens33
       valid_lft 1618sec preferred_lft 1618sec
    inet6 fe80::9581:3f83:6568:8151/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:2d:3b:09:1c brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:2dff:fe3b:91c/64 scope link
       valid_lft forever preferred_lft forever
29: veth6ed8794@if28: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether ba:47:05:c2:31:28 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::b847:5ff:fec2:3128/64 scope link
       valid_lft forever preferred_lft forever
~~~

我们每启动一个docker客器，docker就会给docker客器分配一个ip，我们只要安装了doker，就会有一个网卡docker0

桥接模式，使用的技术是 evth·pair 技术

这些容器带来的网卡，都是一对一对的

evth·pair 技术就是一对的虚拟设备接口，他们都是成对出现的，一端连着协议，一端彼此相连

正因为有这个特性，所以利用 evth·pair 充当一个桥梁，用于连接各种虚拟网络设备

OpenStac，docker容器之间的链接，OVS的连接，都是使用evth·pair技术



Docker使用的是Linux的桥接，宿主机中是一个Docker容器的网桥 docker0

Docker中所有的网络接口都是虚拟的，虚拟的转发效率高

只要容器删除，对应的一对网桥也会删除

## --link

不建议使用

~~~shell
# 使用容器名 ping 容器
# 不能互相ping通，正向可以，反向不可以
docker run -d -P --name tomcat01 --link tomcat tomcat

# --link 就是在hosts配置中增加了一个ip与容器名的映射
[root@localhost /]# docker exec -it tomcat01 cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2      tomcat 69cd02104682
172.17.0.3      5f3ef2e83287
~~~

## 自定义网络

容器互联

**网络模式**

* bridge：桥接，默认
* none：不配置网路
* host：和宿主机共享网络
* container：容器网络连通，局限很大，使用极少

~~~shell
# --driver bridge 桥接
# --subnet 192.168.0.0/16 子网
# --gateway 192.168.0.1 网关
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 test-net

# 可以使用容器名ping 通
docker run -d -P --name tomcat01 --net test-net tomcat
docker run -d -P --name tomcat02 --net test-net tomcat
~~~

自定义网络docker已经维护好了对应关系，推荐

不同的集群使用不同的网络，保证集群是安全健康的

## 网络连通

~~~shell
# 将CONTAINER放到了NETWORK网络下，一个容器两个ip
docker network connect NETWORK CONTAINER
~~~

# IDEA整合Docker

## Spring Boot 打包镜像



# Docker Compose

**容器编排**

* 是什么

Docker官方的开源项目，负责实现对Docker容器集群的快速编排

Docker公司推出的一个工具软件，可以管理多个Docker容器组成一个应用。需要定义一个 YAML 格式的配置文件docker-compose.yml，写好多个容器之间的调用关系。然后，只要一个命令，就能同时启动/关闭这些容器

* 能干嘛

docker建议我们每一个容器中只运行一个服务，因为docker容器本身占用资源极少，所以最好是将每个服务单独的分割开来

但是如果需要同时部署多个服务，每个服务单独写Dockerfile、构建镜像、构建容器，工作量大，所以docker官方提供了docker-compose多服务部署的工具

Compose允许用户通过一个单独的docker-compose,yml模版文件来定义一组相关联的应用容器为一个项目 (project)

可以很容易地用一个配置文件定义一个多容器的应用，然后使用一条指令安装这个应用的所有依赖，完成构建

docker-compose 解决了容器与容器之间如何管理编排的问题



~~~shell
# 安装docker-compose
curl -SL "https://github.com/docker/compose/releases/download/v2.15.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 或
curl -SL https://get.daocloud.io/docker/compose/releases/download/v2.15.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 设置权限
chmod +x /usr/local/bin/docker-compose

# 检查是否安装成功
docker compose version

# 卸载
rm /usr/local/bin/docker-compose
~~~

一文件：docker-compose.yml

两要素：

​	服务（service）:一个个应用容器实例

​	工程（project）：由一组关联的应用容器组成的一个完整业务单元，在docker-compose.yml文件中定义



步骤：

编写Dockerfile定义各个微服务应用并构建出对应的镜像文件

使用 docker-compose.yml定义一个完整业务单元，安排好整体应用中的各个容器服务

执行docker-compose up命令来启动并运行整个应用程序，完成一键部署上线

~~~shell
docker-compose -h					# 查看帮助
docker-compose up					# 启动所有docker-compose服务
docker-compose up -d				# 启动所有docker-compose服务并后台运行
docker-compose down					# 停止并删除容器、网络、卷、镜像
docker-compose exec yml里面的服务id	 # 进入容器实例内部
docker-compose exec yml里面的服务id /bin/bash
docker-compose ps					# 展示当前docker-compose编排过的运行的所有容器
docker-compose top					# 展示当前docker-compose编排过的容器进程
docker-compose logs yml里面的服务id	 # 查看容器输出日志
docker-compose config				# 检查配置
docker-compose config -q 			# 检查起置，有问题才有输出
docker-compose restart 				# 重启服务
docker-compose start				# 启动服务
docker-compose stop					# 停止服务
~~~

## 编排微服务

编写docker-compose.yml文件

~~~shell
# 指定本 yml 依从的 compose 哪个版本制定的
version

# 指定为构建镜像上下文路径
build

# 设置依赖关系
depends_on

# 指定与服务的部署和运行有关的配置。只在 swarm 模式下才会有用
deploy
~~~


~~~yml
version: "3.7"

services:
  mysql:
    image: mysql
    environment:
      - TZ=Asia/Shanghai
      - MYSQL_ROOT_PASSWORD=123456
    ports:
      - 3306:3306
    working_dir: /app
    restart: always
    volumes:
      - /home/mysql/conf:/etc/mysql/conf.d
      - /home/mysql/data:/var/lib/mysql
    command:
      --default-authentication-plugin=caching_sha2_password
    networks: 
      test-net:
        aliases: 
          - mysql
  test:
    image: devops.test.cn:5000/test
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 80:80
    working_dir: /app
    restart: always
    volumes:
      - ./log:/app/log
      - ./application.yml:/app/application.yml
    networks: 
      test-net:
        aliases: 
          - test 
networks: 
  test-net:
    external: true
~~~

执行 docker-compose up或者执行 docker-compose up -d

# Docker Swarm

集群部署

# CI\CD jenkins

流水线

# CIG

CAdvisor

InfluxDB

Granfana



---

CSDN：[https://blog.csdn.net/dkbnull/article/details/129506197](https://blog.csdn.net/dkbnull/article/details/129506197)

微信：[https://mp.weixin.qq.com/s/mArC9fSIyQtp_wszH4YJyw](https://mp.weixin.qq.com/s/mArC9fSIyQtp_wszH4YJyw)

---

