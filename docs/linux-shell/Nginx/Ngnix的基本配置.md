### Ngnix的基本配置

```shell
#使员工Nginx的用户名
#user  nobody;

#cpu数，一般设置成和服务器的cpu数一致
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#进程id
#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
   #设置mime类型，类型由mime.type文件定义
    include       mime.types;
    default_type  application/octet-stream;
    
    #设定日志格式
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;
    #sendfile指令指定Nginx是否调用sendfile函数（zero copy方式）来输出文件，对于普通应用，必须设定为on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的uptime
    sendfile        on;
    #tcp_nopush     on;
    #设置超时时间
    #keepalive_timeout  0;
    keepalive_timeout  65;
    #是否开启gzip压缩（网页速度优化非常有用，开启后通常可以达到70%的压缩率）
    #gzip  on;

    server {
        #侦听端口
        listen       80;
        #域名
        server_name  localhost;
        #编码设置
        #charset koi8-r;
        #设定虚拟主机的访问日志
        #access_log  logs/host.access.log  main;
   
        #默认请求
        location / {
            #默认网站的根目录
            root   html;
            #首页索引文件的名称
            index  index.html index.htm;
        }
        
        #定义错误提示页面，你还可以在这里添加500,403等，以空格分开
        #error_page  404              /404.html;
        
        #重定向
        # redirect server error pages to the static page /50x.html
        #定义错误提示页面
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

