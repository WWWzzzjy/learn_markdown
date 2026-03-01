# Docker

+++

* 镜像-image
* 容器-container
* 仓库地址/注册表-registry
* 镜像库-repository

+++

## Dockerfile

```dockerfile
FROM——#基于的基础镜像
WORKDIR——#切换到镜像内的一个目录，后面的工作都是在这个目录下执行的
COPY . .——#把当前目录copy到工作目录下
RUN——#安装依赖
CMD ["xxx", "xxx"]——#容器启动时的默认启动命令,唯一
ENTRYPOINT——#优先级高
```

```cmd
# 构建镜像
docker build -t image_name .
# 基于镜像创建容器
docker run
# 登录docker.hub
docker login
# 推送Dockerfile
docker build -t username/image_name .
docker push username/image_name
```

+++

## Docker 网络

* 默认是 Bridge 桥接模式
* 每个容器都有分配一个IP地址，172.17开头

```cmd
# 创建子网
docker network create network1
```

* 另一种模式是Host和none模式

+++

## Docker-compose



+++

## Docker command

```cmd
# 查看Docker版本
docker --version
# 下载镜像；repository{仓库地址/命名空间/image:版本号}
docker pull docker.io/library/nginx:latest
# 查看镜像
docker images
# 删除镜像
docker rmi image_name/id
# 查看容器
docker ps -a
# 删除容器
docker rm
# 在image镜像下创建一个容器
docker run image
-p 端口映射：浏览器查看：localhost:port
-v 目录绑定，挂载卷，对容器内的数据在宿主机进行备份
-d 后台运行
-e 传递环境变量
--name 为容器设置名称
-it 允许我的控制台进入容器
--rm 停止容器时删除容器
--restart always/unless-stopped(意外停止重启，手动停止不会重启) 容器停止了就立刻重启
--network network_name 加入一个子网
# 创建挂载卷
docker volume create v_name
# 查看挂载卷参数
docker volume inspect v_name
# 停止容器
dokcer stop
# 启动容器
docker start
# 创建容器但不启动
docker create
# 查看日志
docker logs container -f
# 在宿主机端对容器内执行Linux命令
docker exec container linux命令
# 进行容器进行交互式命令行环境
docker exec -it container /bin/sh
```

**容器里访问 huggingface.co 的网络是通的**（HTTP/2 200）

我手动下载的包，在重新使用compose后会消失吗

![image-20260128154033071](D:\ZJU\自学\Docker.assets\image-20260128154033071.png)

对于json文件的修改，重启之后也会改变吗

改造一台ubuntu的家庭服务器有什么作用，是可以远程进行工作吗

Web 服务最常见的协议（HTTP/HTTPS 都是跑在 TCP 上）

如果是 UDP 服务（比如某些音视频、游戏、DNS），你可能会看到 `7860/udp`
