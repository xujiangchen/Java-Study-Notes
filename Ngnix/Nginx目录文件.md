# Nginx目录文件

## Nginx核心目录
```
- conf  #所有配置文件目录
    - nginx.conf    #默认的主要的配置文件
    - nginx.conf.default  #默认模板
    
- html  # 这是编译安装时Nginx的默认站点目录
    - 50x.html #错误页面
    - index.html #默认首页
    
- logs  # nginx默认的日志路径，包括错误日志及访问日志
  - error.log  #错误日志
  - nginx.pid  #nginx启动后的进程id
  - access.log #nginx访问日志
  
- sbin  #nginx命令的目录
  - nginx  #启动命令
```

## Nginx核心配置文件
```
#user  nobody; # 指定Nginx Worker进程运行以及用户组
worker_processes  1; #保持和cup的核数一致

#error_log  logs/error.log; # 错误日志的存放路径  和错误日志
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid; # 进程PID存放路径

# 事件模块指令，用来指定Nginx的IO模型，Nginx支持的有select、poll、kqueue、epoll等。
#不同的是epoll用在Linux平台上，而kqueue用在BSD系统中，对于Linux系统，epoll工作模式是首选
events {
    use epoll;
    # 定义Nginx每个进程的最大连接数， 作为服务器来说: worker_connections * worker_processes,
    # 作为反向代理来说，最大并发数量应该是worker_connections * worker_processes/2。
    # 因为反向代理服务器，每个并发会建立与客户端的连接和与后端服务的连接，会占用两个连接
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    # 自定义服务日志
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    # 是否开启高效传输模式 on开启 off关闭
    sendfile        on;
    
    # 减少网络报文段的数量
    #tcp_nopush     on;

    #keepalive_timeout  0;
    # 客户端连接保持活动的超时时间，超过这个时间之后，服务器会关闭该连接
    keepalive_timeout  65;

    # 是否进行压缩，带宽减少，性能下降
    #gzip  on;

    # 虚拟主机的配置
    server {
        listen       80; # 虚拟主机的服务端口
        server_name  localhost; #用来指定IP地址或域名，多个域名之间用空格分开

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        # URL地址匹配，可以添加多个
        location / {
            root   html; # 服务默认启动目录
            index  index.html index.htm; # 默认访问文件，按照顺序找
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
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
