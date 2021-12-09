# scp

```shell
利用scp客户端进行文件（夹）上传、下载

1、上传文件，scp 本地文件路径 用户名@服务器ip:目标路径

$ scp ./ly-facturer.war root@服务器:/usr/local/marcus

2、上传文件夹，scp -r 本地文件夹路径 用户名@服务器ip:目标路径

$ scp -r htdocs root@服务器:/usr/local/marcus

3、scp下载文件，scp 用户名@服务器ip:文件路径 本地文件路径

$ scp root@服务器:/usr/local/marcus/ly-facturer.war

4、scp下载文件夹，scp -r 用户名@服务器ip:文件夹路径 本地文件夹路径

$ scp -r root@服务器:/usr/local/marcus/htdocs 
```

sftp

```shell
pwd, 查看服务器当前目录
lpwd，查看本地当前目录

// 文件
put 本地文件 服务器文件
get 服务器文件 本地文件

sftp> put /home/* /home/demo
sftp> get /home/*
    > quit
// 文件夹
put -r /home/. /home
get -r /home ./home
   >
```

#### 工具

```
Filezilla

```