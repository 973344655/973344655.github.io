---
title: docker
date: 2019-01-16 17:25:54
tags: [tools,docker]
---
docker的本意是希望容器是无状态的，即箱子有什么就用什么，我们外部不做修改，如果我需要改代码则重新build一个image出来。<br>
镜像与容器，我的理解是将自己的程序做成镜像，放到容器中运行，容器提供运行环境。
#### 一.install
ubuntu 16.04 LTS<br>
##### 1.sudo apt install docker.io
##### 2.阿里镜像加速
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://etuznpiq.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
保存为sh,执行
##### 3.重启
servie docker restart
##### 4.登录
sudo docker login --username=17888824094 registry.cn-hangzhou.aliyuncs.com
##### 5.拉取远程镜像
sudo docker pull microsoft/aci-helloworld
##### 6.启动
这是启动镜像<br>
docker run  镜像 (-d deamon模式)<br>
例: docker run -d -p 8888:80 microsoft/aci-helloworld<br>
-p：物理机端口，要转发的端口，可以多个端口 -p 8888:8888 8889:8889 3000:3000<br>
目录挂载:-v $PWD/www:/www  将主机中当前目录下的www挂载到容器的/www
##### 7.查看
docker ps 查看容器信息<br>
docker logs 容器id   查看容器内日志
##### 8.关闭
这是操作的容器<br>
docker stop 容器id<br>
docker ps -a  查看关闭的容器<br>
docker restart 容器id    重启容器<br>
docker rm 容器id  删除容器
##### 9.进入container
sudo docker exec  -it 9a8(容器id前三位) /bin/sh<br>
交互模式， /bin/sh为要使用的$PATH,进入容器内进行操作
##### 10.保存对容器的修改
对容器的修改默认是保存的<br>
docker commit 698 name(生成一个新的版本名)
##### 11.生成自己的image
