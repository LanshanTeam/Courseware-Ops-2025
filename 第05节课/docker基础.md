# 虚拟化与容器技术Docker入门

## 介绍

### 什么是虚拟化技术

虚拟化是云计算的基础。简单来说，虚拟化就是在一台物理服务器上，运行多台“虚拟服务器”。这种虚拟服务器，也叫虚拟机（VM，Virtual Machine）。从表面来看，这些虚拟机都是独立的服务器，但实际上，它们共享物理服务器的CPU、内存、硬件、网卡等资源。

物理机，通常称为“宿主机（Host）”。虚拟机，则称为“客户机（Guest）”。

### 什么是Docker

![](docker基础.assets/屏幕截图 2025-11-25 165019.png)

Docker 是一种开源的应用容器引擎，能够让开发者打包他们的应用程序以及依赖包到一个轻量级、可移植的容器中。它利用了 Linux 内核特性来创建独立的工作环境。容器完全使用**沙盒机制**，相互之间不会存在任何接口。几乎没有性能开销，可以很容易的在机器和数据中心运行。Docker使开发、测试、部署变得更加高效，解决了“在我的机器上可以运行但是换台机器就无法实现”的问题，实现跨环境的一致性。

可以这样理解，docker就是集装箱，集装箱和集装箱之间不会互相影响，我们可以直接用一艘大船给它运走，而这艘大船就是云计算（目前流行的）。

### 一些重要概念

#### 镜像（images）

是创建容器的基础，是一个只读的模板文件，里面包含运行容器中的应用程序所需要的所有资料（比如应用程序执行文件、配置文件、动态库文件、依赖包、系统文件和目录等）

#### 容器（container）

是用镜像运行的实例，可以被创建、启动、删除，每个容器之间是默认隔离的。

#### Docker镜像仓库（docker registry）

仓库是用来保存镜像的地方，有公有仓库和私有仓库之分。其中docker的官方镜像仓库是dockerhub。

### Docker与虚拟机的区别

#### 相同点

docker和容器技术和虚拟机技术，都是虚拟化技术。

#### 不同点

#### 优势

- 灵活：即使是最复杂的应用也可以集装箱化
- 轻量级：容器利用并共享主机内核
- 可互换：可以及时部署更新和升级
- 便携式：可以在本地构建，部署到云，并在任何地方运行
- 可扩展： 可以增加并自动分发容器副本

#### 不足

Docker用于应用程序时是最有用的，但并不包含数据。日志、数据库等通常放在Docker容器外。一个容器的镜像通常都很小，不用和存储大量数据，存储可以通过外部挂载等方式使用，或者docker命令 ，-v映射磁盘分区。
总之，docker只用于计算，存储交给别人。

#### docker的特性

文件系统隔离：每个进程容器运行在一个完全独立的根文件系统里。

资源隔离：系统资源，像CPU和内存等可以分配到不同的容器中，使用cgroup。
网络隔离：每个进程容器运行在自己的网路空间，虚拟接口和IP地址。

日志记录：Docker将收集到和记录的每个进程容器的标准流（stdout/stderr/stdin），用于实时检索或者批量检索

变更管理：容器文件系统的变更可以提交到新的镜像中，并可重复使用以创建更多的容器。无需使用模板或者手动配置。

交互式shell：Docker可以分配一个虚拟终端并且关联到任何容器的标准输出上，例如运行一个一次性交互shell。

### 为什么容器越来越受到人们的欢迎

- 灵活：即使是最复杂的应用也可以集装箱化
- 轻量级：容器利用并共享主机内核
- 可互换：可以及时部署更新和升级
- 便携式：可以在本地构建，部署到云，并在任何地方运行
- 可扩展： 可以增加并自动分发容器副本
- 可堆叠：可以垂直和即时堆叠服务

容器在内核中支持2种重要技术
Docker本质就是宿主机的一个进程，docker是通过namespace实现资源隔离，通过cgroup实现资

源限制 ，通过写时复制技术（copy-on-write）实现高效文件操作 （类似于虚拟机的磁盘，比如分

配500g并不是实际占用物理磁盘500g，只有当需要修改时才复制一份数据）

## 如何安装docker

系统：Ubuntu 22.04

### 安装docker,docker-compose

```shell
#更新包管理工具
sudo apt-get update
#添加Docker软件包源
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
sudo curl -fsSL http://mirrors.cloud.aliyuncs.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository -y "deb [arch=$(dpkg --print-architecture)] http://mirrors.cloud.aliyuncs.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
#安装Docker社区版本，容器运行时containerd.io，以及Docker构建和Compose插件
sudo apt-get -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

启动Docker并设置开机自启。

```shell
#启动Docker
sudo systemctl start docker
#设置Docker守护进程在系统启动时自动启动
sudo systemctl enable docker
```

配置镜像源：编辑/etc/docker/daemon.json

```shell
{
    "registry-mirrors": [
        "https://dockerproxy.com",
        "https://docker.m.daocloud.io",
        "https://cr.console.aliyun.com",
        "https://ccr.ccs.tencentyun.com",
        "https://hub-mirror.c.163.com",
        "https://mirror.baidubce.com",
        "https://docker.nju.edu.cn",
        "https://docker.mirrors.sjtug.sjtu.edu.cn",
        "https://github.com/ustclug/mirrorrequest",
        "https://registry.docker-cn.com"
    ]
}
```

也可以选择自己的镜像加速地址。

安装docker compose(v2.39.2)

```shell
sudo apt-get -y install docker-compose-plugin
```

查看版本

```shell
docker --version
```

有输出意味着安好了。

## Docker入门命令

### 基础的镜像操作

#### 搜索镜像

docker search

用来从dockerhub上搜索镜像

```shell
docker search nginx
```

官方提供可放心下载，可以基于此镜像做自己的镜像

#### 拉取镜像

docker pull ，默认是拉去docker hub上搜索到的最新版本

```shell
docker pull tomcat
```

#### 查看镜像

查看已经下载的镜像

```shell
docker images
```

#### 删除镜像

```shell
docker rmi [镜像名:标签]（或者id也可以）
```

#### 添加标签

```shell
docker tag [原名字] [新名字]
```

以nginx为例：

```shell
docker pull nginx    #先拉取镜像
docker images        #查看镜像后发现默认拉取的是最新的镜像
docker tag nginx:latest sz/nginx:latest   #为镜像改标签
```

#### 存出镜像

存储镜像是指将镜像保存成为本地文件。

格式：docker save -o 【存储文件名】 【存储的镜像】

```
docker save -o /opt/nginx.tar nginx:latest		#存出镜像命名为nginx存在当前目录下
ls /opt
```

#### 载入镜像

将镜像文件导入到镜像库中，格式为：

格式：

```shell
docker load < 【存出的文件】
```

或者

```shell
docker load -i 【存出的文件】
```

例如：

```shell
docker load < nginx
```

#### 上传镜像

默认上传到 docker Hub 官方公共仓库，需要注册使用公共仓库的账号。https://hub.docker.com

可以使用 docker login 命令来输入用户名、密码和邮箱来完成注册和登录

在上传镜像之前，还需要先对本地镜像添加新的标签，然后再使用 docker push 命令进行上传

```shell
docker tag nginx:latest sz/nginx:web		#添加新的标签时必须在前面加上自己的dockerhub的username
docker login								    #登录公共仓库
Username：
password：
docker push soscscs/nginx:web					#上传镜像
```

但是，dockerhub的官网我们经常会出现连接超时的问题，所有也可以选择将镜像推送到阿里云的私人镜像仓库里。

### 基础的容器操作

#### 启动容器

```shell
docker run [id 或 容器名]
```

注意：容器是一个与其中运行的 shell 命令/进程共存亡的终端，命令/进程运行容器运行， 命令/进

程结束容器退出，也就是说，Docker容器中必须有一个前台进程，否则认为容器已经挂掉。那么想要解决这个问题，我们可以选择用后台运行的命令。

```shell
docker run -d [id 或 容器名]
```

#### 查看容器

```shell
docker ps
```

用于查看容器状态，up表示健康。

#### 进入容器

```shell
docker exec -it nginx:latest /bin/bash
```

其中：

1. -i 选项表示让容器的输入保持打开；
2. -t 选项表示让 Docker 分配一个伪终端。

退出就用“exit”

#### 停止容器运行

```shell
docker stop [id 或者 容器名]
```

#### 删除容器

```shell
docker rm [id 或者 容器名] #删除已经停止运行的容器
docker rm -f [id 或者 容器名] #删除正在运行的容器
```

## Docker实战部署

### 部署Nginx

第一步：拉取镜像

```shell
docker pull nginx #这里默认用最新版
```

第二步：查看状态

```shell
docker images
```

第三步：启动容器

```shell
docker run -d --name nginx01 -p 80:80 nginx
```

第四步：查看容器

```shell
docker ps
```

第五步：测试访问

```shell
curl 127.0.0.1:80
```

127.0.0.1是本地回环ip，就是本机，也可以用：

```shell
curl localhost:80
```

进入容器修改页面：

```shell
docker exec -it [id] /bin/bash
ls
whereis nginx #这是一个搜索文件的小命令
cd /usr/share/nginx
ls
```

这时你会看到一个html目录，下面有两个html格式的文件，可以对其进行操作：

```shell
echo "xxx(写你喜欢的话就行了)" > index.html
curl localhost:80 #可以查看自己修改的内容
```

想要退出容器就直接输入

```shell
exit
```

## Docker容器数据卷

### 介绍

docker容器在产生数据的时候，如果不通过docker commit生成新的镜像，使得数据作为镜像的一部分保存下来，那么当容器删除之后，数据自然而然的也会消失。为了能保存数据，容器中引用了**数据卷**的概念

### 作用及特点

卷就是**目录或者文件**，存在一个或者多个容器之中，由docker挂载到容器，但是不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或者共享数据的特性。卷的设计目的就是**数据的持久化**，完全**独立于容器的生存周期**，因此docker不会在容器删除时删除其挂载的数据卷。

其特点是：

1、数据卷可在容器之间共享或者重用数据。
2、卷中的更改可以直接生效。
3、数据卷中的更改不会包含在镜像的更新中。
4、数据卷的生命周期一直持续到没有容器使用它为止。

### 一些常用命令

```shell
docker volume ls #查看volume
docker volume create [name] #创建volume
docker volume inspect [数据卷名字] #查看数据卷信息
docker volume rm [name] #删除volume
```

![](docker基础.assets/屏幕截图_22-11-2025_20132_ecs-workbench.aliyun.com.jpeg)

### 如何使用数据卷

命令：-v

```shell
docker run -it -v 主机目录:容器目录 /bin/bash
```

在容器的/home文件夹下写入的任何文件都会同步到主机的/home/ceshi文件夹下，删除操作也是同步的。双向绑定，保证两边文件夹下的数据始终是一致的。

![](docker基础.assets/屏幕截图 2025-11-23 205439.png)

如上图所示，我在容器里创建了一个test目录，退出容器后发现主机也出现了同样的test目录。

当我在主机的test目录下新建一个test01.txt文件并写入hello！时，再次进入容器就会发现之前的test空目录下也出现了这个.txt文件，并且里面写入了同样的内容：

![](docker基础.assets/屏幕截图 2025-11-23 211720.png)

## Dockerfile

### 什么是Dockerfile

Dockerfile是一个创建镜像所有命令的文本文件，包含了一条条指令和说明, 每条指令构建一层,，通过docker build命令，根据Dockerfile的内容构建镜像，因此每一条指令的内容, 就是描述该层如何构建。有了Dockefile,，就可以制定自己的docker镜像规则,只需要在Dockerfile上添加或者修改指令,，就可生成docker 镜像。

### 如何用Dockerfile构建镜像

我们来整理一下docker容器、dockerfile、docker镜像的关系：

dockerfile：是面向开发的，发布项目做镜像的时候就要编写dockerfile文件。用于构建文件，定义了一切的步骤，源代码。
dockerImanges：通过dockerfile构建生成的镜像，最终发布和运行的产品。
docker容器：容器就是镜像运行起来提供服务的。

### Dockerfile指令

```yaml
FROM        # 基础镜像，一切从这里开始构建，必填！
RUN            # 镜像构建的时候需要运行的命令，必填！
ADD            # 用于copy文件，会自动解压
WORKDIR        # 镜像的工作目录，必填！
VOLUME        # 挂载的目录
EXPOSE        # 暴露端口配置，必填！
CMD            # 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代，必填！
ENTRYPOINT    # 指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD        # 当构建一个被继承DockerFile这个时候就会运行ONBUILD的指令。触发指令。
COPY        # 类似ADD，将我们文件拷贝到镜像中，必填！
ENV            # 构建的时候设置环境变量，必填！
```

注意：CMD类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:
CMD 在docker run 时运行。
RUN 是在 docker build。
作用：为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。
CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。
如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效。

### 构建步骤

#### 创建工作目录

```shell
mkdir Dockerfile
cd Dockerfile
ls
```

#### 创建文件

```shell
vi Dockerfile
```

#### 编写Dockerfile

例：

```yaml
FROM centos:7
ENV MYPATH /usr/local
WORKDIR $MYPATH
RUN sed -e 's|^mirrorlist=|#mirrorlist=|g' \
        -e 's|^#baseurl=http://mirror.centos.org|baseurl=https://mirrors.aliyun.com|g' \
        -i /etc/yum.repos.d/CentOS-*.repo
RUN yum -y install vim
RUN yum -y install net-tools
EXPOSE 80
CMD echo $MYPATH
CMD /bin/bash
```

编写完成后用docker build来构建镜像

```shell
docker build -f dockerfile文件路径 -t 镜像名:[tag]
```

如果Dockerfile就在当前目录，比如我的Dockerfile就在当前root目录，那么我会用这个指令来构建：

```shell
docker build -t centos . #其中 . 表示当前目录作为构建上下文
```

构建成功的截图是这样的：

![](docker基础.assets/屏幕截图_23-11-2025_212612_ecs-workbench-disposable.aliyun.com.jpeg)

我们用docker images检查一下：

![](docker基础.assets/屏幕截图_23-11-2025_212756_ecs-workbench-disposable.aliyun.com.jpeg)

这里就说明我们的镜像构建成功了。

## 作业

level 0：安装好docker，用docker拉取并运行hello_world，提交成功的截图

level 1：按照课堂上讲的复现：拉取nginx，并修改访问界面，提供访问截图

level 2：打包一个自己的镜像，上传到镜像仓库（比如阿里云的私人镜像仓库或者在dockerhub官网注册账号），提供访问截图

level 3：举一反三，自己尝试用docker来部署wordpress

level 4（选做）：利用Dockerfile构建镜像，可以是自己写的脚本 ，也可以自己去找，最终要让它在公网可访问
