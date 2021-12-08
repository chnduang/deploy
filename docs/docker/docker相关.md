## 关于Docker相关命令



### 使用时报错

> Got permission denied while trying to connect to the Docker daemon

```shell
sudo groupadd docker          #添加docker用户组
sudo gpasswd -a $USER docker  #将当前用户添加至docker用户组
newgrp docker                 #更新docker用户组
```

### docker 安装及设置

```bash
#安装 CentOS已经将Docker软件包放在了Extras软件源中，直接利用即可
yum install docker-io -y

#查看docker的版本 version
docker -v

#开启Docker服务
systemctl start docker.service

#开机启动Docker服务
systemctl enable docker.service

#查看Docker服务启动状态
systemctl status docker.service

#重启Docker服务
systemctl restart docker.service
```

### docker 镜像文件和容器命令
```shell
#搜索镜像
docker search 镜像名称

#查看所有镜像
docker images

#删除一个imageid的镜像
docker rmi [IMAE_ID] 

#删除所有镜像
sudo docker rmi $(docker images -q) 

#查看所有容器运行状态
docker ps -a    
docker container ls -all

#删除一个containerid的容器(实例)
docker rm 6f0c67de4b72 

#删除所有容器
docker rm $(sudo docker ps -a -q)

容器日志
#查看指定时间后的日志，只显示最后100行：
docker logs -f -t --since="2019-06-08" --tail=100 CONTAINER_ID

#查看某时间之后的日志：
docker logs -t --since="2019-06-08" CONTAINER_ID

#查看某时间段日志：
docker logs -t --since="2019-06-08" --until "2019-06-09" CONTAINER_ID

#查看最近30分钟的日志:
docker logs --since 30m CONTAINER_ID

# 设置启动策略, docker 容器自动启动（在容器退出或断电开机后，docker可以通过在容器创建时的 --restart 来指定重启策略）
#--restart 参数：
  # no，不自动重启容器. (默认值)
  # on-failure，  容器发生error而退出(容器退出状态不为0)重启容器,可以指定重启的最大次数，如：on-failure:10
  # unless-stopped，  在容器已经stop掉或Docker stoped/restarted的时候才重启容器，手动stop的不算
  # always， 在容器已经stop掉或Docker stoped/restarted的时候才重启容器
docker run --restart always -it -p {本机端口}:{容器端口} {镜像名称}

#如果容器已经被创建，但处于停止状态，重新启动：
docker start {容器ID}

#如果容器已经被创建，我们想要修改容器的重启策略
docker update --restart always {容器ID}
```

### Dockerfile操作

```dockerfile
FROM nginx  # 定制的镜像都是基于 FROM 的镜像

# RUN两种执行操作
RUN <命令行命令>
# <命令行命令> 等同于，在终端操作的 shell 命令
RUN ["可执行文件", "参数1", "参数2"]
# 例如：
# RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline

# Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大
RUN yum -y install wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN tar -xvf redis.tar.gz
# 以 && 符号连接命令，这样执行后，只会创建 1 层镜像。
RUN yum -y install wget \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && tar -xvf redis.tar.gz

# 如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效
# CMD 在docker run 时运行
CMD <shell 命令> 
CMD ["<可执行文件或命令>","<param1>","<param2>",...] 
CMD ["<param1>","<param2>",...]  # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数

# 设置环境变量，定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>
# 设置 NODE_VERSION = 7.2.0 ， 在后续的指令中可以通过 $NODE_VERSION 引用
ENV NODE_VERSION 7.2.0
RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc"

# ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。
# 构建命令 docker build 中可以用 --build-arg <参数名>=<值> 来覆盖
ARG <参数名>[=<默认值>]

# 仅仅只是声明端口
EXPOSE 80
```

