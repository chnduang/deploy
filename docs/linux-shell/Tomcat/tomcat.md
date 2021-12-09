# 应用部署tomcat根目录
```shell
应用部署tomcat根目录，localhost:8080直接访问应用目录--亲测
博客分类： tomcat
 
参考地址：1）、http://rongjih.blog.163.com/blog/static/335744612011426103345778/

2）、http://www.cnblogs.com/iyangyuan/archive/2013/09/12/3316444.html

 

本文通过查阅google/百度,通过自己亲测，部署，并测试

本文的目的：

应用部署到Tomcat根目录的目的是可以通过“http://[ip]:[port]”直接访问应用，而不是使用“http://[ip]:[port]/[appName]”上下文路径进行访问。

法一：

删除原webapps/ROOT目录下的所有文件，将应用下的所有文件和文件夹复制到ROOT文件夹下，开启tomcat服务，直接可通过localhost:8080访问应用

法二：

删除原webapps/ROOT目录下的所有文件，修改文件“conf/server.xml”,在HOST节点下增加如下Context的内容配置：

Server.xml配置代码  收藏代码
<Host name="localhost"  appBase="webapps"  
           unpackWARs="true" autoDeploy="true">  
    <Context path="" docBase="E:/project/apache-tomcat-7.0.54/myapps/aa.war"></Context>  
lt;/Host>  
 这里跟文章提供的链接中不一样，需要注意的是（黄色部分），需要将webapps/ROOT目录以及目录下的所有文件都删除，才会有效果，如果不删除ROOT目录，不会去读取context中配置的war包，如果指定已经解压好的应用目录，可以不用删除ROOT目录，通过localhost:8080访问

法三：

与法二类似，但不修改全局配置文件“conf/server.xml”,而是在“conf/Catalina/localhost”目录下增加新的文件“ROOT.xml”（注意大小写），文件内容如下：

Conf/catalina/localhost/root.xml代码  收藏代码
<?xml version="1.0" encoding="UTF-8"?>  
<Context path="" docBase="E:/project/apache-tomcat-7.0.54/myapps/aa.war"></Context>  
注意：要删除ROOT目录，不然也会不能访问 

这三种方法都是经过测试，可以直接访问，无需增加应用目录名才能够访问
```

