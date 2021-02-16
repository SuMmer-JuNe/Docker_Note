# Docker 基本知识

docker相当于起到了承上启下的**桥梁**作用：将APP连带环境一同打包直接部署到服务器。

> 如果是使用Mac或者windows系统使用docker的话，建议使用**Vagrant**，它是不需要使用iso镜像就可以创建虚拟机的，这样的好处是方便我们的使用以及删除。

## 虚拟机与docker之间的区别

![img](http://yqfile.alicdn.com/3e1800924c962b75e71deba35fe0d94c30058ae7.png)

![img](http://yqfile.alicdn.com/46e10e9243c9a64e2b179ca2f07c593e0b857177.png)

传统虚拟机：在宿主机上添加新的操作系统，直接导致了虚拟机的臃肿与不适合迁移。

docker：直接寄存在宿主机上，其实是一个**黑盒的进程**，独立出一个自己的空间，不会使得docker中的行为以及变量溢出到宿主机上。

## 为什么用docker

在docker出现之间，我们完成的java web项目需要打成一个war，然后在服务器中配置各种参数，例如jdk，tomcat，数据库等。配置周期相当的冗杂繁琐。在有了docker后，我们不但可以使用一个空的镜像，从头开始构建，还可以使用之前各种大牛已经build好的镜像，直接使用。而且在项目需要迁移的时候，我们只需要在需要部署的地方，直接使用之前项目使用docker放置自己的项目即可，方便快捷。

## docker底层技术支持

- NameSpaces：用于进程之间的隔离
- Control Groups：用于资源控制，根据需求划分资源的核心数、内存、硬盘等，例如我们之前新建一个虚拟机一样
- Union File Systems（UFS，联合文件系统）：Container和image的分层

## docker的基本概念

docker 最重要的三个概念：**镜像（image）**，容器（container），仓库（repository）。

### 镜像

- 镜像是文件与meta data的集合
- 分层的、并且每一层都可以添加、删除文件，从而形成新的镜像
- 不同的镜像可以共享相同的层（layout）
- 只读的

镜像可以理解为树状结构，每个镜像都会依赖另一个镜像，这个依赖关系是体现在docker镜像制作的dockerfile中的FROM指令中的。如果是树的根，那么我们需要“FROM scratch"，这个是值得注意的。如果需要修改，就涉及到容器了。

### 容器

- 通过image创建
- 在image的最后一层上面再添加一层，这一层比较特殊，可读写
- image负责存储和分发，container负责运行

容器是镜像的一个运行实例，可以准确的把镜像当做类，容器当做对象。容器的结构与镜像是相似的，底部也是一层层的只读层，只不过在最上层会存在一个存储层，我们在这一层定制化我们的容器，还可以通过build命令，把容器打包成我们自己需要的镜像。另外镜像启动后会形成一个容器，容器在计算机中是一个进程，但这个进程对其他进程并不可见。

**容器的启动过程：**

检查镜像是偶发在本地存在，如果不存在区远程仓库下载

1. 利用镜像创建一个容器
2. 启动刚刚创建的容器
3. 分配一个文件系统给容器，并且在镜像层外挂载一个可读可写层
4. 从宿主机主机的网桥接口中桥接一个给容器
5. 从网桥中分一个ip地址给容器
6. 执行用户指定的应用程序
7. 执行完毕后容器自动终止

### 仓库

类比git，有一个远程的仓库，这个仓库记录着我们的代码和每一次我们提交的记录。这里把docker的仓库比作maven仓库更加恰当，就想当与我们可以到maven远程仓库取我们需要的依赖，多个依赖构成了我们整个项目，这个思想同样适用于docker。默认情况下，我们都是从docker hub中获取镜像（[http://registry.hub.docker.com/](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fregistry.hub.docker.com%2F)）

## 使用docker

不建议在windows上安装docker

安装的链接：
Mac：[mac安装docker的方法](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fwww.runoob.com%2Fdocker%2Fmacos-docker-install.html)
Linux：[centos安装docker的方法](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fwww.runoob.com%2Fdocker%2Fcentos-docker-install.html)

### 1. docker search <镜像名称>

当我们在对docker的镜像一无所知的时候，我们可以通过查询镜像，查看自己想要的镜像存不存在。

例：docker search tomcat

### 2. docker pull <镜像名称>:<镜像版本>

默认情况下是latest，即最新的版本。

### 3. docker run -d -p <外部端口>:<内部端口> <镜像名称>:<镜像的tag>

打开terminal终端，输入：docker run -d -p 8080:8080 tomcat 这里启动一个tomcat镜像

说明：docker在运行的时候，会启动一个tomcat，这个tomcat的端口为8080。外部端口指的就是宿主电脑的端口。即我们可以通过我们的电脑访问的端口。

- -d <指的是后台运行容器>

- -p <指的是指定端口>

- --link <连接的容器名称>:<连接的别名>

  可以使容器和容器之间相互连接

- -v <挂载到容器的目录>

- --volumes-from <可以挂载的数据卷容器名称>（通过启动一个新容器，使用-v命令挂载一个目录，然后通过这个命令把容器挂载到数据卷容器上，可以多个容器挂载同一个数据卷容器上，而且数据卷容器本身可以不是启动的状态）

- -v <本地已有的目录>（这个路径必须是绝对路径）：<容器的目录>

注：docker run 是一个组合命令，实际上组合的是 create+start

### 4. docker create --name <容器名称> <镜像名称>:<镜像tag>

### 5. docker start <容器的id>

create命令是在镜像的只读层的最上面加一个存储的可读可写层。上面的-d和-p的option都是create命令的，创建一个容器可配的option相当多。create命令后，容器是处于stop状态的，需要start命令启动。

start命令则是给容器分配一个进程，然后启动起来。

### 6. docker images

查看本地已有的docker镜像。

通过docker images --format[.ID]:[.Repository] 指定显示镜像id以及名称

### 7. docker rmi <容器名称>:<容器tag>

移除已存在的指定容器

### 8. docker ps -a

查看所有的容器。

当把-a换成-qa的时候就是查看所有容器的id

### 9. docker exec -it <容器ID> /bin/bash

进入docker容器docker exec -it(后面可以接上容器id或者容器名称) /bin/bash

- -it 是 -i以及-t
- -i 是保持标准输入打开
- -t 是分配一个伪终端

### 10. docker tag <已有镜像名>:<已有镜像版本> <新镜像名>:<新镜像版本>

使用docker tag tomcat:latest mytomcat:latest  这样就会新生成一个镜像名字叫做mytomcar，版本为latest，只有别名不同而已，但是同样指向了同一个镜像。

### 11. docker inspect <镜像名称>:<镜像版本>

显示镜像的详细信息，包括制作者、适应框架，各层的数字摘要。

### 12. docker history <镜像名称>:<镜像版本>

使用此命令，可以看出镜像发生的变化。过程的信息会被自动拦截，可以使用--no-trunc显示完整的信息。

### 13. docker commit <进程号>/<容器名称> <镜像名称>:<版本号>

基于现有镜像，形成自己的镜像。原理是在原有的镜像基础上增加自己修改过的存储层，叠加为一个新的镜像，保存下来。成功后，会返回一个sha256的码作为镜像的唯一标识。

形成自己的新镜像，有三个方法：

- 通过commit命令
- 通过模板构建
- 通过openVZ为我们提供的模板创建，也可以通过自己导出的模板创建，openvz网址[openvz模板下载地址。](https://yq.aliyun.com/go/articleRenderRedirect?spm=a2c4e.11153940.0.0.f2dd63b9Oqju9R&url=https%3A%2F%2Fwiki.openvz.org%2FDownload%2Ftemplate%2Fprecreated)

以ubantu的模板为例。命令为cat ubuntu-x86_64-minimal.tar.gz | docker import ubuntu:14.04。执行完以上命令，就可以通过docker images 看到我们的镜像了。

- 通过Dockerfile文件构建

**docker commit 需要慎用**，删除操作，并不是真正意义上的删除，每一次的修改都是在上一次的基础上，这使得 镜像越来越臃肿。因此想要定制我们的docker镜像可以使用dockerfile。

### 13. docker save -o 导出的镜像名称（后缀为.tar）<镜像名称>:<镜像的版本>

### 14. docker load --input 导出的镜像名称（后缀为.tar）

### 15. docker export -o 导出的镜像名称（后缀为.tar）

### 16. docker import 导出的镜像名称（后缀为.tar）<镜像名称>:<镜像的版本>

前两个命令是搭配在一起的，通过save命令，把镜像打包成tar文件的形式，这就可以分享给别人使用，然后其他人可以通过load命令将导出的文件，保存到本地。

后两个命令也是搭配在一起的，将正在运行的容器，打包成一个tar文件，然后把tar文件import进来，形成一个镜像。

load和import的区别：

- load 导入镜像存储文件到本地镜像库
- import 导入一个容器快照到本地镜像库

镜像存储文件保持完整记录，相对的体积也会更大。export比save速度快很多。

### 17. docker stop <容器的ID>

停止正在运行的容器实例

### 18. docker rm <容器的ID>

删除一个容器实例之前，应该先将容器停止。

- -f，--force = false 强制删除一个正在运行的容器
- -l，--link = false 删除容器的连接，但是保存容器
- -v，--volumes=false 删除容器挂载的数据卷

### 19. docker restart <容器的ID>

重启容器，将容器先停止，后启动

### 20. docker build -t <生成的镜像名称>:<镜像的版本>

这是三种创建自己镜像的方法之一，也是最好的一种方法。

dockerfile的位置上为“.”，表示的是当前目录，说明cmd和dockerfile在同一目录下。

### 21. docker container prune

清理所有已经停止的容器，在容器失败停止后，做集体的清理特别方便。