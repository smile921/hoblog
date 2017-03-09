---
title: docker hub 国内加速
date: 2016-08-10 15:40:01
tags:
  - docker
---

[DaoCloud 加速器](https://www.daocloud.io/mirror)是广受欢迎的 Docker 工具，解决了国内用户访问 Docker Hub 缓慢的问题。DaoCloud 加速器结合国内的 CDN 服务与协议层优化，成倍的提升了下载速度。

# 配置加速器

请先确定您的 Docker 版本在 1.8 及以上。 
登陆[加速器页面](https://www.daocloud.io/mirror#accelerator-doc)可以获取 mirror 地址。
配置好后，您可以像往常一样使用docker pull命令，在拉取 Docker Hub 镜像时会自动采用加速器的镜像服务。

## Linux
### 自动配置 Docker 加速器
_适用于 Ubuntu14.04、Debian、CentOS6 、CentOS7_

```
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://aa62783f.m.daocloud.io  
```

该脚本可以将 --registry-mirror 加入到你的 Docker 配置文件 /etc/default/docker 中。适用于 Ubuntu14.04、Debian、CentOS6 、CentOS7，其他版本可能有细微不同。更多详情请访问文档。

登陆后参考[配置命令](https://www.daocloud.io/mirror#accelerator-doc)
此命令会帮助您配置 registry-mirror 并重启 Docker Daemon。

### 手动配置 Docker 加速器
_适用于各种 Linux 发行版_

您可以找到 Docker 配置文件，一般配置文件在```/etc/default/docker``` ，在配置文件中的```DOCKER_OPTS```加入
CentOS 7  在 /lib/systemd/system/docker.service 中


```
--registry-mirror=加速地址
```

重启Docker，一般可以用下面命令重启


```
service docker restart
```
