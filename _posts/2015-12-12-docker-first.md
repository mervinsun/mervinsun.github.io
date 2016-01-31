---
layout: post
title:  "Docker初探"
date:   2015-4-11 22:14:54
categories: Docker
excerpt: 
---

* content
{:toc}


## 序

Docker很火，貌似有成为下一个OpenStack的节奏。云计算领域今年流行的开源软件，都有个共同的特点：你如果要在项目中用它，还真要动手体验下才行，我把她们都归类为动手型的技术。所以趁着休息闲时，亲自搭建环境折腾一番，否则后面不好和别人吹牛。

---

## Docker 架构

先看看Docker的架构，本人比较懒，直接从网上找了张图看下，Docker是一种虚拟化技术，不同于传统的VM，Doceker是在OS层做的虚拟技术LXC（Linux Container的缩写）的一种实现，其原理实际利用了Linux对通过名称空间将线程隔离的方式。线程实际对应了虚拟化的容器。容器需要运行在Docker引擎中，而Docker引擎运行于OS之上，从而将容器实现了平台无关性，这点类似其它的虚拟机技术。传统的应用可以运行在一个个的容器内部，彼此互相隔离。
Docker提供了镜像仓库支持引擎上传和下载镜像文件，Docker官方提供了中央仓库：[https://registry.hub.docker.com/](https://registry.hub.docker.com/)，全世界的开发者可以下载和上传镜像，但悲催的是，在国内很多镜像要完整下载安装需要代理，一些常用的镜像可以通过国内的[http://dockerpool.com/](http://dockerpool.com/)下载。

![ruby-gems]({{ "/css/pics/image002.png"}})

当然Docker的与众不同不仅仅是这些，这些我们和我们之前使用的所有虚拟机技术、maven这类技术都是类似的，docker的不同之处还在于，镜像并非是虚拟机这样的完整的OS镜像，而是在OS之上逐层累加的，所以如果制作方法得当，可以将应用的镜像做的更小，并且按需累加需要的功能，这一点是虚拟机无法做到的。

![ruby-gems]({{ "/css/pics/image004.png"}})

好了，下面就开始我们的折腾之旅。
我们目标包括以下6个任务：

任务一，安装

任务二，创建一个基础镜像

任务三，创建一个Web应用镜像

任务四，创建一个Mysql镜像

任务五，连接容器

任务六，镜像的导入和导出

---

### 任务一，安装

从Docker的官方网站中有详细的安装指导
[https://docs.docker.com/installation/ubuntulinux/](https://docs.docker.com/installation/ubuntulinux/)
很不幸，和OpenStack一样，官方的安装都是需要连接网络，如果开发环境在内网无法访问互联网，那么可要折腾一下了，可以下载Docker的二进制包[https://docs.docker.com/installation/binaries/](https://docs.docker.com/installation/binaries/)
建议还是找个能连接外网的环境，因在使用中涉及到镜像下载，如果无法联网镜像获取会很不方便，使用过程中还可能会涉及到gcc、util-linux、unzip等工具的安装，没有网络真的是够折腾的。

本次使用的ubuntu-14.04版本作为Docker的OS，安装Docker很简单，见上面的连接。
安装完成后，通过docker info可以查看docker的信息，通过docker version可以查看其版本，我本地的环境如下：
![ruby-gems]({{ "/css/pics/image006.png"}})![ruby-gems]({{ "/css/pics/image007.png"}})

通过docker images命令可以查看docker中的镜像，初始情况下是没有镜像的：
![ruby-gems]({{ "/css/pics/image008.png"}})

至此任务一完成，有关Docker更多的命令的使用见：
[https://docs.docker.com/reference/commandline/cli/](https://docs.docker.com/reference/commandline/cli/)

---

### 任务二，安装基础镜像

这里我们下载第一个基础镜像，因为使用的ubuntu系统，还是下载个ubuntu的镜像（其它镜像是否可以用，待验证）：
使用如下命令下载一个ubuntu的基础镜像，版本为12.04

    docker pull ubuntu:12.04

下载完成后，再次查询images，增加了一个容器:

![ruby-gems]({{ "/css/pics/image010.png"}})

第一个基础镜像则创建完成了。

---

### 任务三，创建Web应用的镜像

基于刚创建的基础镜像，可以创建自己的镜像了，这里将自己原来的web应用镜像迁移到docker中，创建一个原来本地程序的镜像
首先准备程序运行时环境目录，此例中我们需要准备两个环境目录

1. jre（JAVA应用）：jre的运行时目录
2. Tomcat: web容器的运行时目录

再准备一个用于创建镜像的配置文件，名为Dockerfile。在老版本Docker（before1.5）,仅认这个配置文件名字，有点类似ant和maven。
Dockerfile的内容如下:

![ruby-gems]({{ "/css/pics/image012.png"}})


1. 说明此镜像的基于ubuntu:14.10创建
2. 添加运行时目录，创建容器中的目录
3. 导出端口，后面用于外部的端口映射和访问使用
4. 镜像作为容器启动的脚本 

最终的准备制作镜像的准备目录如下：
![ruby-gems]({{ "/css/pics/image013.png"}})

万事具备，现在就可以创建镜像了，执行命令创建：

    docker build -t tomcat11 .

这里我随便起了个名字为了和远程仓库的镜像区别。镜像创建完成后，可以看到，增加了我们新创建的镜像：

![ruby-gems]({{ "/css/pics/image014.png"}})

通过docker images –tree命令，可以看到镜像的层关系：

![ruby-gems]({{ "/css/pics/image016.png"}})

这时我们可以通过如下命令启动刚创建的镜像，我们将宿主机的8088端口映射到容器的8080端口:

    docker run -d -p 8088:8080 tomcat11

![ruby-gems]({{ "/css/pics/image018.png"}})

至此，第3个任务完成。

---

### 任务四，创建mysql镜像

这第四个镜像着实折腾了一下，Docker官方提供的镜像因网络问题，无法下载完成，下载时某些组件总是报404的错误，没办法只好在国内的镜像站点找了一个：

    docker pull dl.dockerpool.com:5000/mysql:latest

下载完成后，可以查看到镜像：

![ruby-gems]({{ "/css/pics/image020.png"}})

我们将仓库名称和标签修改下，因默认的名称带有斜杠“/”后面导入导出会出问题

    docker tag dl.dockerpool.com:5000/mysql:latest my-mysql:latest

修改后可以查看到我们的mysql镜像了:

![ruby-gems]({{ "/css/pics/image022.png"}})

现在可以启动镜像了:

    docker run --name my-mysql -d -e MYSQL_ROOT_PASSWORD=Admin@123 -e DB_REMOTE_ROOT_NAME=root -e DB_REMOTE_ROOT_PASS=Admin@123 -p 3306:3306 my-mysql:latest

默认的容器数据无法持久化，可以通过挂载卷将数据放于容器外部：

    docker run --name my-mysql -d -e MYSQL_ROOT_PASSWORD=Admin@123 -e DB_REMOTE_ROOT_NAME=root -e DB_REMOTE_ROOT_PASS=Admin@123 -v /opt/mysql/data:/var/lib/mysql -p 3306:3306 my-mysql:latest

启动后，我们在本地用Navicat或其它client工具连接下，可以连接成功：

![ruby-gems]({{ "/css/pics/image024.png"}})

第四个任务完成。

---

### 任务五，连接web应用和mysql容器

至此web应用的mysql数据库的镜像均已经创建好了，现在需要将二者连接，使应用能跨容器访问数据库，首先修改web应用的数据库连接：
修改mysql的连接配置，将原来的IP改为容器的名称：

    hibernate.connection.url=jdbc:mysql://my-mysql:3306/test?createDatabaseIfNotExist=true&autoReconnect=true&useUnicode=true&characterEncoding=utf-8

重启任务三中的web应用，这里我们使用—link参数将容器打通：

    docker run -P --name lego --link my-mysql:my-mysql tomcat11

link参数将容器名称映射为数据库连接使用的主机地址,
此时web容器已经可以像在容器外一样正常连接数据库了。

---

### 任务六，镜像的导出导入

很多场景下我们的应用可能在内部网络中运行，所以需要考虑镜像的外部导入的方式来部署。可以通过save命令将镜像导出：

    docker save -o mysql-image.tar my-mysql:latest

也可以通过将镜像的名称和标签换为id:

    docker save -o ./mysql-image.tar 3c6d7e5c8c1b

导入镜像，要通过load的方式导入，此方式会保留历史层和元数据:

    docker load --input mysql-image.tar
    sudo docker tag dl.dockerpool.com:5000/ubuntu:12.04 my-mysql

原镜像：

![ruby-gems]({{ "/css/pics/image026.png"}})

通过load方式导入的镜像:

![ruby-gems]({{ "/css/pics/image028.png"}})

查看原始镜像的层:

![ruby-gems]({{ "/css/pics/image030.png"}})

查看load方式导入的镜像层，包含了所有的历史层：

![ruby-gems]({{ "/css/pics/image032.png"}})

可以看到导入后保留了原来镜像的各层信息，注意在较早的版本通过Load导入后，会丢失标签名字，这里可以通过任务四中的tag命令重新设置。
新导入的镜像可以和任务四中的镜像一样启动，我们仅仅修改下启动容器名称和映射的端口，就可以快速启动多个mysql应用的拷贝：

    docker run --name my-mysql01 -d -e MYSQL_ROOT_PASSWORD=Admin@123 -e DB_REMOTE_ROOT_NAME=root -e DB_REMOTE_ROOT_PASS=Admin@123 -p 3307:3306 my-mysql:latest
    docker run --name my-mysql02 -d -e MYSQL_ROOT_PASSWORD=Admin@123 -e DB_REMOTE_ROOT_NAME=root -e DB_REMOTE_ROOT_PASS=Admin@123 -p 3308:3306 my-mysql:latest
    docker run --name my-mysql03 -d -e MYSQL_ROOT_PASSWORD=Admin@123 -e DB_REMOTE_ROOT_NAME=root -e DB_REMOTE_ROOT_PASS=Admin@123 -p 3309:3306 my-mysql:latest

查看容器，已经启动了3个新的mysql容器了：

![ruby-gems]({{ "/css/pics/image034.png"}})

通过Navicat测试，可以连接新创建的mysql实例了：

![ruby-gems]({{ "/css/pics/image036.png"}})
