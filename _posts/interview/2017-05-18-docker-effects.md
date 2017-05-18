---
layout: post
title: "Docker虚拟化实战：解放你的双手"
date: 2017-05-18 09:55:18 +0800
categories: 工具学习
tag: docker
---

* content
{:toc}


　　在 ```JavaScript``` 当中一个变量的作用域（scope）是程序中定义这个变量的区域。变量分为两类：全局（global）的和局部的。其中全局变量的作用域是全局性的，即在 ```JavaScript``` 代码中，它处处都有定义。而在函数之内声明的变量，就只在函数体内部有定义。它们是局部变量，作用域是局部性的。 **函数的参数也是局部变量**，它们只在函数体内部有定义，**局部变量省略了 var 也就默认成为了全局变量**。

　　在我们每次在云上面搭建自己服务的时候，是不是每次都得先去apt-get 或者 yum我们的一下必要的组件，如果这个时候，我们想从腾讯云换成阿里云了怎么办？难道还要再次apt-get,yum一下吗？在docker没有出来之前，好像就是这个样子滴。至少我是这样，每次换个云都会这么搞，现在docker出现了，这一次是真的解放了我的双手！！！！

  
  
  ###1.什么是Docker
  在一台服务器上同时运行一百个虚拟机，估计这个服务器是天价级别，不知道现在造出来没有，但是在服务器上运行一千个Docker容器，这已经是现实了。
  Docker是基于G语言实现的云开源项目，出生于2013年初。Docker自开源后收到了很多的关注和讨论，最初他的发起者的公司是dotCloud后面也改了名字叫Docker Inc。Docker项目目前已加入了Linux基金会，遵循Apache 2.0协议.
  基本所有的主流系统都支持Docker，包括在windows和mac下也都推出了Boot2Docker,Docker的主要目标是达到对应用组件级别的一次封装，到处运行，类似我们的java一次编译到处运行，也就是说只要你安装了docker,我们就可以使用我们的docker镜像。这里的应用组件代表很多，可以使web应用，也可以是数据库服务，也可以是操作系统或编译器。而且在Docker社区中有很多镜像，基本很多时候直接可以拿来使用，在进行一次封装即可。
  ###2.Docker和虚拟机比较
  1.Docker容器启动很快，启动和停止都可以在秒级实现，这比我们那些虚拟机快很多。
  2.Docker容器对系统资源需求很少，一台主机上可以同时运行数千个Docker容器。
  3.Docker操作是非常的简单，用过Git的人相信学习这个是非常快，学习成本很低。
  4.Docker通过Dockerfile可以完成一键部署，这是虚拟机不能完成的。
  
  传统的虚拟机，每启动一个，都会为那个单独分配独占的内存，磁盘等资源，但是对于Docker来说只需要启动N个隔离的容器，并将应用放到容器中，并且在Docker中有挂载这个功能，本地主机可以和容器共享同一文件夹或者文件，这也是虚拟机没有的。
  ![虚拟机](http://7u2qr4.com1.z0.glb.clouddn.com/blog_%E5%9B%BE%E7%89%871.png)
  　　　　　　**虚拟机**
  ![docker](http://7u2qr4.com1.z0.glb.clouddn.com/blog_%E5%9B%BE%E7%89%872.png)
   　　　　　　**docker**
   上图展示两个的具体差别：
   docker有着比虚拟机更少的抽象层。由于docker不需要Hypervisor实现硬件资源虚拟化，运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。每个传统的虚拟机都会有一个自己的独立os。
   
  
  ###3.实战
  ==
  这里不会介绍基本知识，只介绍我在实战中怎么使用，全程使用Dockfile快速构建。
  
  首先docker自带的镜像源，那个速度在国内简直太慢了，这个时候就需要国内的镜像源，我一般使用两个国内的源，一个是阿里云hub，另外一个是daocloud,如果你是使用阿里云的服务器使用阿里云hub的加速源，速度真的是无敌。
  这里给大家搭建一个以tomcat7和jdk1.7为基础的docker,这两个东西请自行去网上下载压缩包
  我这里使用的是阿里云的服务器，首先我们需要去阿里云:https://dev.aliyun.com/search.html
  搜索ubuntu
  支持mirro加速的不要选，那个速度很慢，直接选支持阿里云内网加速，然后选择一个ubuntu14.04,在docker中输入下面命令:
  docker pull registry.cn-hangzhou.aliyuncs.com/docker/ubuntu14.04
  这样我们的虚拟机上面就会有一个这样的镜像了可以使用docker images查看是否有:
  然后我们使用如下命令
  
  ```
  mkdir tomcat_ubuntu
  cd tomcat_ubuntu
  touch Dockerfile run.sh
  ##然后把之前的压缩都cp进来
  vim Dockerfile
  
  
  ##继承我们之前的某一个镜像，这里是继承ubuntu:14.04也就是我们刚才下的
  FROM ubuntu:14.04
  ##作者信息
  MAINTAINER lz from lclizhao.cn
  ##环境变量设置为非交互式
  ENV DEBIAN_FRONTEND noninteractive
  ##设置时区
  RUN echo "Asia/Shanghai" > /etc/timezone && \
          dpkg-reconfigure -f noninteractive tzdata
  ##执行apt-get命令
  RUN apt-get install -yq --no-install-recommends wget pwgen ca-certificates && \
      apt-get clean && \
      rm -rf /var/lib/apt/lists/*
  ##设置tomcat和java的环境变量
  ENV CATALINA_HOME /tomcat
  ENV JAVA_HOME /jdk
  ##添加本地内容到镜像中
  ADD tomcat7 /tomcat
  ADD jdk /jdk
  ADD run.sh /run.sh
  RUN chmod +x /*.sh
  RUN chmod +x /tomcat/bin/*.sh
  ##开放端口
  EXPOSE 8080
  ##启动容器时我们执行run.sh
  CMD ["/run.sh"]
  
  ##上面完成后编辑我们的run.sh
  vim run.sh
  exec ${CATALINA_HOME}/bin/catalina.sh run
  ```
  上面的步骤完成以后，我们就可以生成我们的镜像了
  ```
  sudo docker build -t tomcat7:java7 .
  ```
  上面的意思是创建一个名字是tomcat7,标签名字是Java7的镜像，最后有一个.，代表当前执行当前目录下以及当前子目录下的Dockerfile。
  这样我们就生成了镜像了，输入命令
  
  ```
  sudo docker images
  ```
  就可以看见会出现我们所需要的，既然镜像有了，下面就是容器了我们可以使用
  
  ```
  docker run -d --name tomcat -p 80:8080 -v /webapp:/tomcat/webapps tomcat7:java7
  ```
  上面的意思是后台运行一个名字叫tomcat的容器，映射端口为80到8080的容器，并且挂载了webapps到容器的webapps，在真正的开发工程也可以使用-v让tomcat7的日志文件都挂载出来。
  这时候我们需要添加我们的web应用怎么办，我们直接把war导入webapp，然后进入容器的tomcat重启
  
  ```
  docker exec -it d39 /bin/bash
  ```
  上面的d39是我们docker ps过后看到的id值。这样可以进入我们的容器然后让tomcat重启。我们就可以使用正常服务了。
  ###4.容器转移
  上面我们已经开始正常使用了，这个时候我们需要更换云服务器怎么办，我们再也不需要像以前一样如此麻烦了，只要几个命令即可
  
  ```
  sudo docker export d39 >tomcat_ubuntu.tar
  ```
  意思是将d39容器导入到tomcat_ubuntu.tar这样就可以传输到其他底仓了
  在其他地方地方可以使用
  
  ```
  cat tomcat_ubuntu.tar | sudo docker import -tomcat/ubuntu:java7
  ```
  这样也就导入进来了，又可以继续使用，完全不需要像我们以前那样辛苦的操作了
  
   

<br>