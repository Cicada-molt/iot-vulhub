# iot-vulhub
iot环境搭建

作者：qingjiegong

# 简介

在做固件漏洞复现环境时，发现基本没啥文章是将容器部署的，可虚拟机又确实是不方便，云主机有条件限制，为此想做容器方向的靶机。

然后发现GitHub上已经有该项目，可惜维护力度不够了（也许是很多人并不接触docker吧），为了方便快捷，本人也进行了一次环境的靶场解析复现（要我给大家讲解源码的话就算了，等支持的哥们和与原作者同意了再给大家一一刨析）。

## 参考

https://github.com/VulnTotal-Team/IoT-vulhub

https://github.com/kal1x/iotvulhub

## 环境准备

* docker环境（docker，docker-compose）
* iot

# 镜像

## docker-compose

```
version: '1'
services:
  ubuntu:
    image: k4l1xx/ubuntu1604
  binwalk:
    image: k4l1xx/binwalk:noentry
  qemu-system-mips:
    image: k4l1xx/qemu-system:mips
  qemu-system-mipsel:
    image: k4l1xx/qemu-system:mipsel
  qemu-system-armel:
    image: k4l1xx/qemu-system:armel
  qemu-system-armhf:
    image: k4l1xx/qemu-system:armhf
  firmae:
    image: k4l1xx/firmae
  firmadyne:
    image: k4l1xx/firmadyne
```

![image-20231022222804748](C:\Users\qingjiegong\AppData\Roaming\Typora\typora-user-images\image-20231022222804748.png)

## docker

### 拉取镜像

```
docker pull k4l1xx/ubuntu1604
docker pull k4l1xx/binwalk:noentry
docker pull k4l1xx/qemu-system:mips
docker pull k4l1xx/qemu-system:mipsel
docker pull k4l1xx/qemu-system:armel
docker pull k4l1xx/qemu-system:armhf
docker pull k4l1xx/firmae
docker pull k4l1xx/firmadyne
```

### 提交镜像方法

> 根据自己情况要不要做该步骤，毕竟存放在自己仓库想要利用时会比较安全

1. 在官网注册帐号
2. 在本地环境登录

```
docker login
```

1. 修改镜像名称

>  如果想构建好直接push到dockerhub，需要遵循一定的命名规范。
>
> 如果没有指定标签名称，默认是latest

```
docker build -t {注册用户名}/{镜像名}:{标签名} .
```

想要将现有不规范名称的镜像推送到dockerhub，需要用tag命令改名：

- tag命令修改为规范的镜像：

```
docker tag {不规范的镜像名称} {注册用户名}/{镜像名}:{标签名}
```

1. 推送镜像

- 推送镜像的规范是：

```
docker push {注册用户名}/{镜像名}:{标签名}
```

**实操：**

>  请巧妙使用"|" 符号，为何？不然一条条命令打多累！！

```
docker tag k4l1xx/ubuntu1604 qingjiegong/ubuntu1604:latest
docker tag k4l1xx/binwalk:latest qingjiegong/binwalk:latest
docker tag k4l1xx/binwalk:noentry qingjiegong/binwalk:noentry
docker tag k4l1xx/qemu-system:mips qingjiegong/qemu-system:mips
docker tag k4l1xx/qemu-system:mipsel qingjiegong/qemu-system:mipsel
docker tag k4l1xx/qemu-system:armel qingjiegong/qemu-system:armel
docker tag k4l1xx/qemu-system:armhf qingjiegong/qemu-system:armhf
docker tag k4l1xx/firmae qingjiegong/firmae:latest
docker tag k4l1xx/firmadyne qingjiegong/firmadyne:latest
```

![image-20231022225756254](C:\Users\qingjiegong\AppData\Roaming\Typora\typora-user-images\image-20231022225756254.png)

### new docker-compose

提交镜像到自己的docker hub仓库后，编写的docker-compose修改为自己的用户名

```
version: '1'
services:
  ubuntu:
    image: qingjiegong/ubuntu1604
  binwalk:
    image: qingjiegong/binwalk:noentry
  qemu-system-mips:
    image: qingjiegong/qemu-system:mips
  qemu-system-mipsel:
    image: qingjiegong/qemu-system:mipsel
  qemu-system-armel:
    image: qingjiegong/qemu-system:armel
  qemu-system-armhf:
    image: qingjiegong/qemu-system:armhf
  firmae:
    image: qingjiegong/firmae
  firmadyne:
    image: qingjiegong/firmadyne
```

## 准备工作

> 关键部分是sock5代理，这点是本人想了很久都没有灵感的关键，简直是茅塞顿开！！！

- 使用sock5代理的方法

[![img](https://github.com/kal1x/iotvulhub/raw/main/img/sock5.png)](https://github.com/kal1x/iotvulhub/blob/main/img/sock5.png)

