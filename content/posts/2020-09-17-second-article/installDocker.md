---
title: CentOS install Docker
author: Confused Bear
date: 2019-04-30
hero: ./images/article-image-2.jpg
excerpt: come to talk with me.
tags: docker
---

### 1 查看内核版本

> docker官方说至少3.8以上，建议3.10以上（ubuntu下要linux内核3.8以上， RHEL/Centos 的内核修补过， centos6.5的版本就可以——这个可以试试）

```shell
[root@VM_0_15_centos ~]# uname -a
Linux VM_0_15_centos 3.10.0-862.el7.x86_64 #1 SMP Fri Apr 20 16:44:24 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```



### 2 更新yum包

```shell
[root@VM_0_15_centos ~]# yum update
已加载插件：fastestmirror, langpacks
Determining fastest mirrors
epel                                                     | 5.4 kB     00:00     
extras                                                   | 2.9 kB     00:00     
os                                                       | 3.6 kB     00:00     
updates                                                  | 2.9 kB     00:00     
正在解决依赖关系
--> 正在检查事务
---> 软件包 GeoIP.x86_64.0.1.5.0-11.el7 将被 升级
---> 软件包 GeoIP.x86_64.0.1.5.0-14.el7 将被 更新
......
```



### 3 安装所需yum包

```shell
[root@VM_0_15_centos ~]# yum install -y yum-utils device-mapper-persistent-data lvm2
已加载插件：fastestmirror, langpacks
Loading mirror speeds from cached hostfile
软件包 yum-utils-1.1.31-52.el7.noarch 已安装并且是最新版本
软件包 device-mapper-persistent-data-0.8.5-1.el7.x86_64 已安装并且是最新版本
软件包 7:lvm2-2.02.185-2.el7_7.2.x86_64 已安装并且是最新版本
无须任何处理
```



### 4 设置docker的yum源

```shell
[root@VM_0_15_centos ~]# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
已加载插件：fastestmirror, langpacks
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
```



### 5 查看docker版本

```shell
[root@VM_0_15_centos ~]# yum list docker-ce --showduplicates | sort -r
已加载插件：fastestmirror, langpacks
可安装的软件包
Loading mirror speeds from cached hostfile
docker-ce.x86_64            3:19.03.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.0-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.9-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.8-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.7-3.el7                     docker-ce-stable
......
```



### 6 选择docker版本进行安装

```shell
[root@VM_0_15_centos ~]# yum install docker-ce-18.06.3.ce
已加载插件：fastestmirror, langpacks
Loading mirror speeds from cached hostfile
正在解决依赖关系
--> 正在检查事务
---> 软件包 docker-ce.x86_64.0.18.06.3.ce-3.el7 将被 安装
--> 正在处理依赖关系 container-selinux >= 2.9，它被软件包 docker-ce-18.06.3.ce-3.el7.x86_64 需要
--> 正在处理依赖关系 libcgroup，它被软件包 docker-ce-18.06.3.ce-3.el7.x86_64 需要
--> 正在处理依赖关系 libltdl.so.7()(64bit)，它被软件包 docker-ce-18.06.3.ce-3.el7.x86_64 需要
......
```



### 7 启动docker

```shell
[root@VM_0_15_centos ~]# systemctl start docker
[root@VM_0_15_centos ~]# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
```



### 8 验证docker安装并启动成功

```shell
[root@VM_0_15_centos ~]# docker version
Client:
 Version:           18.06.3-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        d7080c1
 Built:             Wed Feb 20 02:26:51 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.3-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  Git commit:       d7080c1
  Built:            Wed Feb 20 02:28:17 2019
  OS/Arch:          linux/amd64
  Experimental:     false
```



### 9 查看docker当前信息

```shell
[root@VM_0_15_centos ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@VM_0_15_centos ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@VM_0_15_centos ~]# docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 18.06.3-ce
Storage Driver: overlay2
 Backing Filesystem: extfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
......
```



### 10 docker开启端口监听

# !!!!!慎重，不要在公网开2375端口，很可能会被挖矿盯上!!!!!

```shell
cd /lib/systemed/system
vim docker.service
```

修改docker.service内容：

```shell
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
```

重启docker守护进程和docker，使配置生效

```shell
systemctl daemon-reload
systemctl restart docker
```

检查端口是否在监听

```shell
netstat -an | grep 2375
tcp6       0      0 :::2375                 :::*                    LISTEN   
```

> 还需要服务器安全组开放2375端口的访问
>
> 开放docker端口有可能会有安全问题

