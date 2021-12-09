### Nginx + Node

> 充分使用二级域名映射服务器内部的端口
>
> 在nginx配置文件中再创建一个server

```bash
server {
    listen       80;
    #域名
    server_name  qdzhou.cn www.qdzhou.cn;

    location / {
     		proxy_set_header X-Real-IP $remote_addr;
     		# 默认会将后端服务器的HOST填写进去
				proxy_set_header Host $http_host;
         #node.js应用的端口
        proxy_pass http://127.0.0.1:3000;
        root /home/server/;
   }
    #静态文件交给Nginx直接处理
    location ~ *^.+\.(css | js | txt | swf | mp4)$ {
        root /home/;
        access_log off;
        expires 24h;
   }
}
```

#### 通过设置nginx配置去调整转发报文的头部：

- proxy_set_header X-real-ip $remote_addr;

  - 其中这个X-real-ip是一个自定义的变量名，名字可以随意取，这样做完之后，用户的真实ip就被放在X-real-ip这个变量里了
  - 在web端可以这样获取：request.getHeader("X-real-ip")

- proxy_set_header X-Forwarded-For $remote_addr;

  - 真实的显示出客户端原始ip。
  - nginx更多使用这条配置，X-Forwarded-For为默认字段  

- **proxy_set_header Host $http_host;**

  - 如果想获取客户端访问的头部，可以这样来设置。
  - 但是，如果客户端请求头中没有携带这个头部，那么传递到后端服务器的请求也不含这个头部。  

- **proxy_set_header Host $host;**

  - 这个配置相当于上面配置的增强。
  - 它的值在请求包含"Host"请求头时为"Host"字段的值，在请求未携带"Host"请求头时为虚拟主机的主域名。  

- **proxy_set_header Host $host:$proxy_port;**

  - 服务器名和后端服务器的端口（访问端口）一起传送。

- **proxy_set_header <<<\*>>> "";**

  - 请求头的值为空，请求头将不会传送给后端服务器。

- **proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;**

  - 在默认情况下经过proxy转发的请求，在后端看来远程地址都是proxy端的ip 。
    - 添加这条配置之后：
    - 意思是增加一个$proxy_add_x_forwarded_for到X-Forwarded-For里去，注意是**增加**，而不是覆盖
    - 当然由于默认的X-Forwarded-For值是空的，所以我们总感觉X-Forwarded-For的值就等于$proxy_add_x_forwarded_for的值，
    - 实际上当你搭建两台nginx在不同的ip上，并且都使用了这段配置，那你会发现在web服务器端通过request.getHeader("X-Forwarded-For")获得的将会是***客户端\******ip**和**第一台****nginx的ip***。

  + 在第一台nginx中,使用时

    + proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;**
    + 现在的$proxy_add_x_forwarded_for变量的"X-Forwarded-For"部分是空的，所以只有$remote_addr，而$remote_addr的值是用户的ip，于是赋值以后，X-Forwarded-For变量的值就是用户的真实的ip地址了。

  + 到了第二台nginx，也使用

    + **proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;**

      现在的$proxy_add_x_forwarded_for变量

  + > X-Forwarded-For部分包含的是用户的真实ip
    >
    > $remote_addr部分的值是上一台nginx的ip地址
    >
    > 于是通过这个赋值以后现在的X-Forwarded-For的值就变成了用户的真实ip，第一台nginx的ip