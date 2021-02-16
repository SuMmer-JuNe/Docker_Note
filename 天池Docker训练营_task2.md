# Dockerfile

## 引子

如何将改变了一些配置的Container再生成一个镜像？

通过docker commit命令，这个命令的目的是将我们最新修改作为镜像的一层进行构建，详情参考：

[https://docs.docker.com/engine/reference/commandline/commit/](https://yq.aliyun.com/go/articleRenderRedirect?url=https%3A%2F%2Fdocs.docker.com%2Fengine%2Freference%2Fcommandline%2Fcommit%2F)

但是不推荐使用这种方式，因为这种构建方式相当于一个黑盒的构建，这时需要引入**DockerFile**

dockerfile并不是向镜像里直接写入，因为镜像是只读的。docker在这个时候创建了一个临时的容器，然后写入内容之后，再把临时容器删除。

## Dockerfile使用说明

我们创建自己需要的镜像的时候，可以通过commit和dockerfile的形式进行构建。官方推荐的还是dockerfile的形式，可以把它理解为一个构建脚本。

### 基本指令说明

#### 1. ARG指令

定义创建镜像过程中使用的变量，相当于我们为docker bulid --build-arg赋值。镜像编译结束后，这个变量不会被保存

```shell
ARG version=1.0
```

#### 2. FROM指令

指定我们要在哪个image之上再进行构建，尽量使用官方image进行base image。

一个Dockerfile必须要以From指令作为开头（ARG是唯一一个可以先于From命令的）

```shell
FROM debian:latest
```

#### 3. LABEL指令

像是代码里的注释一样，一些概括的维护者信息。

```shell
LABLE author=harry
```

#### 4. ENV指令

定义变量，可以在dockerfile下方进行使用，例如定义了ENV USER harry，那么下面可以这样使用"${USER}"

```shell
ENV FILE_LOCATION /usr/local/file
```

#### 5. USER指令

指定运行容器时的用户是谁

#### 6. WORKDIR指令

进入到我们指定的目录中，**如果没有这个目录会自动进行创建**，用WORKDIR代替RUN cd。尽量使用绝对目录，不要使用相对目录

```shell
WORKDIR /usr/local
WORKDIR tomcat/config
# 可以连续指定路径；如果向上述一样，指定的路径为/usr/local/tomcat/configs
```

#### 7. RUN指令

每执行一次RUN会产生镜像的一层，使用"&&"将多个命令串联起来，如果需要换行在最后需要使用反斜杠。环境的运行与搭建，大多数情况下需要使用这个命令

```shell
RUN yum update \
		&& yum install -y nginx
# 上述操作先更新yum，在下载nginx
```

#### 8. CMD指令

设置启动后默认执行的命令和参数。如果docker run进行了指定了命令，例如docker run -it ... /bin/bash。则不会运行CMD中的命令，而且CMD定义多个，后面会覆盖之前的。

```shell
启动tomcat命令
CMD ['catalina.sh', 'run']
```

#### 9. ENTRYPOINT指令

设置容器启动时默认执行的命令和参数，该命令会在启动容器后作为根命令执行，通过名称可以看出来是入口。让容器以应用程序或者服务去执行。并且ENTERPOINT一定会执行

```shell
将一个shell脚本作为docker启动的入口
ENTERPOINT ['/entry.sh']
```

#### 10. COPY指令

将本地文件拷贝到docker里去，**COPY指令优于ADD指令**，如果需要添加远程文件可以使用curl或者wget

```shell
COPY . /temp
```

#### 11. ADD指令

把本地的文件复制到docker里去，还会对**压缩文件自动进行解压缩**

```shell
ADD . /temp
```

#### 12. VOLUME指令

启动容器时，可以在本地或者其他容器创建数据卷挂载点，用于存放数据库和持久化数据

```shell
# 指定挂载点为 /temp/mount
VOLUME /temp/mount
```

#### 13. EXPOSE指令

声明镜像内部服务监听的端口，一次可以暴露多个端口

```shell
# 暴露22端口和8888端口
EXPOSE 22 8888
```

#### 14. ONBUILD指令

指定自己的子镜像都会执行哪些命令

```shell
# 把当前目录下的所有东西拷贝到/app/src目录下
ONBUILD COPY . /app/src
```

DockerFile的写法关键在于：环境 + 工程代码 + 运行

DockerFile的最佳实践：[https://docs.docker.com/develop/develop-images/dockerfile_best-practices/](https://yq.aliyun.com/go/articleRenderRedirect?url=https%3A%2F%2Fdocs.docker.com%2Fdevelop%2Fdevelop-images%2Fdockerfile_best-practices%2F)



##### ARG和ENV的区别

两者均用于声明变量，区别在于ARG是创建镜像过程中使用的变量，在启动后的容器中不能再使用。而ENV在容器中可以使用。

##### RUN，CMD以及ENTERPOINT的相同点以及区别

三者均可使用exec格式和shell格式

exec格式举例：

```shell
CMD ['/bin/echo', 'hell world']
```

shell格式举例：

```shell
CMD echo 'hello world'
```

注意：

1. 使用exec格式，打印一个环境变量：`CMD['/bin/echo', 'hello world $name']`，结果是 hello world name

   这时可以考虑使用shell格式，或者改造exec格式：`CMD['/bin/bash','-c','echo hello world $name ']`

2. RUN命令用于构建镜像，CMD和ENTERPOINT用于指定容器启动时的一些默认指令和参数

###### CMD和ENTERPOINT的共同点

两者在dockerfile中各自都只能声明一次，声明多次不会报错，但只有最后一条命令生效。

###### CMD和ENTERPOINT的区别

官网解释：ENTERPOINT是docker容器的主命令，而默认的一些参数会在CMD中指定。

```shell
ENTERPOINT ['/bin/echo', 'hello']
CMD ['world']
```

运行docker run -it <image> 会输出hello world

而运行docker run -it <image> harry 会输出hello harry

说明：如果run的时候没有指定会执行CMD，如果指定了命令就不会执行CMD。

**总结**：CMD会被作为命令或者参数再ENTERPOINT参数后追加；CMD可被覆盖，ENTERPOINT不会被覆盖。

###### CMD结合ENTERPOINT的使用

假设一个简单的场景，我们希望docker不作为一个应用程序启动，而是用作一个工具。假设为一个压力测试的工具，这个工具需要被指定一些参数，例如 --vm，我们可以通过`CMD["/usr/bin/strees",'--vm 1']`这种形式启动。但是这样变量值就别限制死了，如何做到docker启动的时候动态传入呢？

```shell
FROM ubuntu
RUN apt-get update && apt-install -y strees
ENTERPOINT ["/usr/bin/strees"]
CMD[]
```

这样在启动的时候可以动态地将变量传入：

```shell
docker run -it dockerImage 名称 --vm 1
```

**总结**：如果CMD和ENTERPOINT结合起来，那么ENTERPOINT是用来指定命令的，而CMD中则是用来指定参数的