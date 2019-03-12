---
title: docker搭建漏洞环境
date: 2019-02-14 11:53:54
tags: [docker]
---
#### 1.docker-compose
pip install docker-compose<br>
Compose 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排,负责快速的部署分布式应用。
#### 2.git
git clone https://github.com/c0ny1/vulstudy.git
#### 3.start
###### 1.单独运行一个漏洞平台
cd到要运行的漏洞平台下运行以下命令<br>

cd vulstudy/DVWA<br>
docker-compose up -d #启动容器<br>
docker-compose stop #停止容器<br>
###### 2.同时运行所有漏洞平台
在项目根目录下运行以下命令<br>

cd vulstudy<br>
docker-compose up -d #启动容器<br>
docker-compose stop #停止容器<br>

##### 3.查看信息
docker-compose ps 查看启动信息<br>
docker-compose logs 查看日志<br>
```
docker ps  
sudo docker exec  -it 9a8(容器id前三位) /bin/sh<br>
进入容器查看/修改
```
