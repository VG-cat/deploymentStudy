# docker

```
最新笔记资料请访问：
飞书链接：https://b11et3un53m.feishu.cn/wiki/MWQIw4Zvhil0I5ktPHwcoqZdnec?from=from_copylink  密码：j.N?-+4[
```

## 安装

```
# 查看支持的版本
apt-get madison docker-ce
#安装
apt-get install docker-ce -y
```

## 启动

```
systemctl start docker
systemctl status docker

service docker start
service docker stop
```

docker  version

docker -v

## 相关目录

````
cd /etc/docker/

vim deamon.json # 源配置信息

cd /var/lib/docker/   # 资源目录
````

## 常用命令

```
docker search  镜像

docker imgage  ...

docker image inspect ubuntu   # 查看详细信息

docker rmi nginx      # 删除镜像
```

## 镜像



镜像重命名

```
docker tag nginx:latest nginx01:latest
```

打包镜像

```
docker save -o nginx.tar nginx   # 打包配置好的资源

docker load -i nginx.tar      # 解压
```

## 容器

docker container --help

```
docker ps   # 查看容器
docker ps -a   #查看所有容器

docker run -dit --name myd ubuntu  # dit 守护进程

docker logs myd

docker exec -it myd /bin/bash  # 进入容器 -it交互式

docker stop myd  # 停止容器（对于后台容器，退出不会停止）

docker start myd  #启动容器

docker container rm myd  #删除容器（不能删除真正允许的容器）
```
打包容器成为镜像
```
docker commit 容器名  镜像资源名
```

  

## 部署流程2

1. 安装系统（Ubuntu）镜像和容器
2. 更新系统容器的软件源（apt-get update）
3. 在系统容器中安装所有需要的环境
4. 将项目文件上传到系统容器中
5. 将系统容器压缩，生成镜像
6. 发送压缩包给他人，解压并创建容器即可使用

## 私有仓库

```
docker pull registry    # 下载 registry镜像

docker run -d -p 5000:5000 registry   #运行仓库容器

cd  /etc/docker/deamon.json

将 insecure-registries 改为本机私有IP

systemctl restart docker  #重启docker服务
systemctl status docker

docker push image_name  #上传镜像

docker pull image_name  #拉取镜像
```

## 数据卷

创建共享文件目录

将宿主机的某个目录映射到容器中，作为数据存储的目录，我们可以在宿主机上直接对数据进行存储

```
docker run -it -v ~/data:/home ubuntu /bin/bash  # 将本机data目录映射到容器中的home

在data下复制存放项目文件即可
```

另一个作用是，多个容器之间数据共享

## 数据卷容器

创建模板容器

将宿主机的某个目录，使用容器的方式来表示，然后其他的应用容器将数据保存到这个容器中，达到大批量应用数据同时存储的目的。

```
docker create -v --name temp /data ubuntu  # 创建模板容器，将data映射到ubuntu根目录下

docker run -it --volumes-from temp --name myd ubuntu    # 根据模板创建容器
```

## 数据卷操作

docker volume --help

pwd  查看当前路径

## 网络管理

-P 随机分配ip和端口

docker run -dit -P --name myd nginx

-p 指定ip和端口映射(宿主机的4000映射到容器的5000)，ip:port：port

docker run -dit -p 4000:5000 --name myd nginx   

## Dockerfile

利用脚本自动化，创建镜像，完成部署

Dockerfile编写格式：

```
# 指定基础镜像
FROM ubuntu:16.04
# 配置环境变量，JDK的安装目录、容器内时区
ENV JAVA_DIR=/usr/local
ENV TZ=Asia/Shanghai
# 拷贝jdk和java项目的包
COPY ./jdk8.tar.gz $JAVA_DIR/
COPY ./docker-demo.jar /tmp/app.jar
# 设定时区
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
# 安装JDK
RUN cd $JAVA_DIR \
 && tar -xf ./jdk8.tar.gz \
 && mv ./jdk1.8.0_144 ./java8
# 配置环境变量
ENV JAVA_HOME=$JAVA_DIR/java8
ENV PATH=$PATH:$JAVA_HOME/bin
# 指定项目监听的端口，暴露docker该端口
EXPOSE 8080
# 入口，java项目的启动命令
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

```
docker build -t tempimage .  # 执行当前目录下的Dockerfile文件,生成镜像资源tempimage
```

### 常见命令

```
ADD ./data /home     # 关联路径

RUN  command1 && command2   # 执行命令1和命令2

WORKDIR /home   # 指定进入容器后，所处的目录

```

## 部署流程2

django 构建一个容器

nginx 构建一个容器

redis 构建一个容器

MySQL 构建一个容器

.....

```
docker run -dit --network-host ubuntu   #让容器使用宿主机的网络，处于相同的网络

即可使用宿主机redis,mysql
或其他容器的服务
```

 

##Docker compose

同时生成多个镜像，启动并运行

