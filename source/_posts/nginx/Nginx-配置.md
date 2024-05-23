---
title: Nginx-配置
date: 2024-05-23 15:30:23
tags:
---
# Nginx 架构
![图 0](../1131847985f79f1f463c35f3eb2ebb5ec2b5e8c3e97b41cdfe9233b2c06d5cf4.png)  
从上边这张图，我们可以一览nginx的架构设计，首先我们可以直观得出nginx的几大特点：

1. 事件驱动&异步非阻塞
2. 多进程机制
3. 服务端缓存
4. 反向代理

# Nginx模块
![图 1](../dcb9be47fd4ef765e7f7768f5cb9fe70c8cf416be6f139e49c80a19f6408c47f.png)  
+ **核心模块** ：是nginx 服务器正常运行必不可少的模块，提供错误日志记录、配置文件解析、事件驱动
机制、进程管理等核心功能
+ **标准HTTP模块** ：提供 HTTP 协议解析相关的功能，如：端口配置、网页编码设置、HTTP 响应头设
置等
+ **可选HTTP模块** ：主要用于扩展标准的 HTTP 功能，让nginx能处理一些特殊的服务，如：Flash 多
媒体传输、解析 GeoIP 请求、SSL 支持等
+ **邮件服务模块** ：主要用于支持 nginx  的邮件服务，包括对 POP3 协议、IMAP 协议和 SMTP 协议的支持
+ **第三方模块** ：是为了扩展 Nginx 服务器应用，完成开发者自定义功能，如：Json 支持、Lua 支持等


# nginx目录一览
```shell
[root@localhost /]# tree  /usr/local/nginx/  -L 2
/usr/local/nginx/
├── conf                        #存放一系列配置文件的目录
│   ├── fastcgi.conf           #fastcgi程序相关配置文件
│   ├── fastcgi.conf.default   #fastcgi程序相关配置文件备份
│   ├── fastcgi_params         #fastcgi程序参数文件
│   ├── fastcgi_params.default #fastcgi程序参数文件备份
│   ├── koi-utf           #编码映射文件
│   ├── koi-win           #编码映射文件
│   ├── mime.types        #媒体类型控制文件
│   ├── mime.types.default #媒体类型控制文件备份
│   ├── nginx.conf        #主配置文件
│   ├── nginx.conf.default #主配置文件备份
│   ├── scgi_params      #scgi程序相关配置文件
│   ├── scgi_params.default #scgi程序相关配置文件备份
│   ├── uwsgi_params       #uwsgi程序相关配置文件
│   ├── uwsgi_params.default #uwsgi程序相关配置文件备份
│   └── win-utf          #编码映射文件
├── html                 #存放网页文档
│   ├── 50x.html         #错误页码显示网页文件
│   └── index.html       #网页的首页文件
├── logs                 #存放nginx的日志文件
├── sbin                #存放启动程序
│   ├── nginx           #nginx启动程序
│   └── nginx.old       
```

# nginx.conf文件 解读
首先我们要知道nginx.conf文件是由一个一个的指令块组成的，nginx用{}标识一个指令块，指令块中再设置具体的指令(注意**指令必须以;号结尾**)。
指令块有：
1. 全局模块
2. events模块
3. http模块
4. server模块
5. location模块
6. upstream模块

精简后的结构如下：

```shell
全局模块
event模块
http模块
    upstream模块
    
    server模块
        location块
        location块
        ....
    server模块
        location块
        location块
        ...
    ....    
```
**各模块的功能作用如下描述：**

1. 全局模块： 配置影响nginx全局的指令，比如运行nginx的用户名，nginx进程pid存放路径，日志存放路径，配置文件引入，worker进程数等。
2. events块： 配置影响nginx服务器或与用户的网络连接。比如每个进程的最大连接数，选取哪种事件驱动模型（select/poll epoll或者是其他等等nginx支持的）来处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
3. http块： 可以嵌套多个server，配置代理，缓存，日志格式定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
4. upstream块： 配置上游服务器的地址以及负载均衡策略和重试策略等等。
5. server块： 配置虚拟主机的相关参数比如域名端口等等，一个http中可以有多个server。
6. location块： 配置url路由规则。

**下面看下nginx.conf长啥样并对一些指令做个解释：**

```shell
# 注意：有些指令是可以在不同指令块使用的（需要时可以去官网看看对应指令的作用域）。这里只是演示

[root@localhost /usr/local/nginx]# cat /usr/local/nginx/conf/nginx.conf

#user  nobody; # 指定Nginx Worker进程运行用户以及用户组，默认由nobody账号运行

worker_processes  1;  # 指定工作进程的个数，默认是1个。具体可以根据服务器cpu数量进行设置， 比如cpu有4个，可以设置为4。如果不知道cpu的数量，可以设置为auto。 nginx会自动判断服务器的cpu个数，并设置相应的进程数

#error_log  logs/error.log;  # 用来定义全局错误日志文件输出路径，这个设置也可以放入http块，server块，日志输出级别有debug、info、notice、warn、error、crit可供选择，其中，debug输出日志最为最详细，而crit输出日志最少。
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info; # 指定error日志位置和日志级别

#pid        logs/nginx.pid;  # 用来指定进程pid的存储文件位置

events {
    accept_mutex on;   # 设置网路连接序列化，防止惊群现象发生，默认为on
    
    # Nginx支持的工作模式有select、poll、kqueue、epoll、rtsig和/dev/poll，其中select和poll都是标准的工作模式，kqueue和epoll是高效的工作模式，不同的是epoll用在Linux平台上，而kqueue用在BSD系统中，对于Linux系统，epoll工作模式是首选
    use epoll;
    
    # 用于定义Nginx每个工作进程的最大连接数，默认是1024。最大客户端连接数由worker_processes和worker_connections决定，即Max_client=worker_processes*worker_connections在作为反向代理时，max_clients变为：max_clients = worker_processes *worker_connections/4。进程的最大连接数受Linux系统进程的最大打开文件数限制，在执行操作系统命令“ulimit -n 65536”后worker_connections的设置才能生效
    worker_connections  1024; 
}

# 对HTTP服务器相关属性的配置如下
http {
    include       mime.types; # 引入文件类型映射文件 
    default_type  application/octet-stream; # 如果没有找到指定的文件类型映射 使用默认配置 
    # 设置日志打印格式
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    # 
    #access_log  logs/access.log  main; # 设置日志输出路径以及 日志级别
    sendfile        on; # 开启零拷贝 省去了内核到用户态的两次copy故在文件传输时性能会有很大提升
    #tcp_nopush     on; # 数据包会累计到一定大小之后才会发送，减小了额外开销，提高网络效率
    keepalive_timeout  65; # 设置nginx服务器与客户端会话的超时时间。超过这个时间之后服务器会关闭该连接，客户端再次发起请求，则需要再次进行三次握手。
    #gzip  on; # 开启压缩功能，减少文件传输大小，节省带宽。
    sendfile_max_chunk 100k; #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    
    # 配置你的上游服务（即被nginx代理的后端服务）的ip和端口/域名
    upstream backend_server { 
        server 172.30.128.65:8080;
        server 172.30.128.65:8081 backup; #备机
    }

    server {
        listen       80; #nginx服务器监听的端口
        server_name  localhost; #监听的地址 nginx服务器域名/ip 多个使用英文逗号分割
        #access_log  logs/host.access.log  main; # 设置日志输出路径以及 级别，会覆盖http指令块的access_log配置
        
        # location用于定义请求匹配规则。 以下是实际使用中常见的3中配置（即分为：首页，静态，动态三种）
       
        # 第一种：直接匹配网站根目录，通过域名访问网站首页比较频繁，使用这个会加速处理，一般这个规则配成网站首页，假设此时我们的网站首页文件就是： usr/local/nginx/html/index.html
        location = / {  
            root   html; # 静态资源文件的根目录 比如我的是 /usr/local/nginx/html/
            index  index.html index.htm; # 静态资源文件名称 比如：网站首页html文件
        }
        # 第二种：静态资源匹配（静态文件修改少访问频繁，可以直接放到nginx或者统一放到文件服务器，减少后端服务的压力），假设把静态文件我们这里放到了 usr/local/nginx/webroot/static/目录下
        location ^~ /static/ {
            alias /webroot/static/; #访问 ip:80/static/xxx.jpg后，将会去获取/url/local/nginx/webroot/static/xxx.jpg 文件并响应
        }
        # 第二种的另外一种方式：拦截所有 后缀名是gif,jpg,jpeg,png,css.js,ico这些 类静态的的请求，让他们都去直接访问静态文件目录即可
        location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
            root /webroot/static/;
        }
        # 第三种：用来拦截非首页、非静态资源的动态数据请求，并转发到后端应用服务器 
        location / {
            proxy_pass http://backend_server; #请求转向 upstream是backend_server 指令块所定义的服务器列表
            deny 192.168.3.29; #拒绝的ip （黑名单）
            allow 192.168.5.10; #允许的ip（白名单）
        }
        
        # 定义错误返回的页面，凡是状态码是 500 502 503 504 总之50开头的都会返回这个 根目录下html文件夹下的50x.html文件内容
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        
    }
    # 其余的server配置 ,如果有需要的话
    #server {
        ......
    #    location / {
               ....
    #    }
    #}
    
    # include /etc/nginx/conf.d/*.conf;  # 一般我们实际使用中有很多配置，通常的做法并不是将其直接写到nginx.conf文件，
    # 而是写到新文件 然后使用include指令 将其引入到nginx.conf即可，这样使得主配置nginx.conf文件更加清晰。
    
}
```

以上就是nginx.conf文件的配置了，主要讲了一些指令的含义，当然实际的指令有很多，我在配置文件并没有全部写出来，准备放到后边章节详细阐述这些东西，比如：location匹配规则，反向代理，动静分离，负载均衡策略，重试策略，压缩，https,限流，缓存，跨域这些 我们都没细说，这些东西比较多比较细不可能把使用规则和细节都写到上边的配置文件中，所以我们下边一一解释说明关于这些东西的配置和使用方式。

另外值的注意的是： 因为有些指令是可以在不同作用域使用的，如果在多个作用域都有相同指令的使用，那么nginx将会遵循就近原则或者我愿称之为 内层配置优先。 eg: 你在 http配了日志级别，也在某个server中配了日志级别，那么这个server将使用他自己配置的已不使用外层的http日志配置。


# 反向代理
![图 2](../906719353d4c09aff2cc5893b4f7268ea6c9212e6b2abe5e8029ad1c30606f5d.png)  

要让 nginx 代理 我们的 服务 很简单，简单描述一下就是 两步：

通过upstream指令块来定义我们的上游服务（即被代理的服务）
通过location指令块中的 proxy_pass指令，指定该location要路由到哪个upstream

配置好1和2后，如果来了请求后 会通过url路由到对应的location, 然后nginx会将请求打到upstream定义的服务地址中去。
![图 3](../10a064ec18361b3d955e76e0ca66fd30f73116bb5b64f3fc94e68e99d9fccf3c.png)  

> **注意上边的 proxy_pass http://mybackendserver/ 后边这个斜线加和不加区别挺大的，加的话不会拼接/backend , 而不加的话会拼接 /backend**

## 消除 proxy_pass 转发 /的恐怖阴影

在配置Nginx过程中路由转发的一个小小 / 可能会给你带来很多麻烦，虽然只是一个小小的斜杠，但是转发的结果千差万别

1. proxy_pass 不带/

```python
location /alpha/ {
    proxy_pass  http://192.168.xxx.xxx:80;
}
http://domain/alpha/ --> http://192.168.xxx.xxx:80/alpha/
http://domain/alpha/beta/abc --> http://192.168.xxx.xxx:80/alpha/beta/abc

```

2. proxy_pass 带/

```python
location /alpha/ {
    proxy_pass  http://192.168.xxx.xxx:80/;
}
http://domain/alpha/ --> http://192.168.xxx.xxx:80/
http://domain/alpha/beta/abc --> http://192.168.xxx.xxx:80/beta/abc

```

## 反向代理流程与原理
对于上边演示的反向代理案例的流程与原理，我们来个示意图如下
![图 4](../02ccd71b11335caae443ba0c6391dd9fe671526e6a630859838ed2f63b91273f.png)  

# 负载均衡
说到负载均衡很多人应该并不陌生，总而言之负载均衡就是：避免高并发高流量时请求都聚集到某一个服务或者某几个服务上，而是让其均匀分配（或者能者多劳），从而减少高并发带来的系统压力，从而让服务更稳定。对于nginx来说，负载均衡就是从 upstream 模块定义的后端服务器列表中按照配置的负载策略选取一台服务器接受用户的请求。

## nginx常用的负载策略
1. 轮询(默认方式)
   1. 每个请求会按时间顺序逐一分配到不同的后端服务器 
   2. 在轮询中，如果服务器down掉了，会自动剔除该服务器
   3. 缺省配置就是轮询策略
   4. 此策略适合服务器配置相当，无状态且短平快的服务使用
2. weight(权重方式)
   1. 在轮询策略的基础上指定轮询的几率
   2. 权重越高分配到的请求越多
   3. 此策略可以与least_conn和ip_hash结合使用
   4. 此策略比较适合服务器的硬件配置差别比较大的情况
3. ip_hash(依据ip的hash值来分配)
   1. 在nginx版本1.3.1之前，不能在ip_hash中使用权重（weight）
   2. ip_hash不能与backup同时使用
   3. 此策略适合有状态服务，比如session
   4. 当有服务器需要剔除，必须手动down掉
4. least_conn(最少连接方式)
   1. 此负载均衡策略适合请求处理时间长短不一造成服务器过载的情况
5. fair(响应时间方式)
   1. 根据后端服务器的响应时间来分配请求，响应时间短的优先分配
   2. Nginx本身不支持fair，如果需要这种调度算法，则必须安装upstream_fair模块
6. url_hash(依据URL分配方式)
   1. 按访问的URL的哈希结果来分配请求，使每个URL定向到一台后端服务器
   2. Nginx本身不支持url_hash，如果需要这种调度算法，则必须安装Nginx的hash软件包

## 轮询
轮询策略是默认的，所以只需要如下这样修改配置文件就可以了

```shell
upstream backend_server { 
    ip_hash
    server 172.30.128.65:8080;
    server 172.30.128.65:8081;
    server 172.30.128.65:8082;
}
```

## weight
weight指令用于指定轮询机率，weight的默认值为1，weight的数值与访问比率成正比。 接下来我们指定8082端口的服务的weight=2，如下：

```shell
upstream backend_server { 
    ip_hash
    server 172.30.128.65:8080;
    server 172.30.128.65:8081;
    server 172.30.128.65:8082 weight=2;
}
```

## ip_hash
设定ip哈希很简单，就是在你的upstream中 指定 ip_hash;即可，如下：
```shell
upstream backend_server { 
    ip_hash;
    server 172.30.128.65:8080;
    server 172.30.128.65:8081;
    server 172.30.128.65:8083;
}
```

## least_conn
同ip_hash一样，设定最小连接数策略也很简单，就是在你的upstream中 指定 least_conn;即可，如下：
```shell
upstream backend_server { 
    least_conn;
    server 172.30.128.65:8080;
    server 172.30.128.65:8081;
    server 172.30.128.65:8083;
}
```

# 动静分离


# Rewrite语法规则
1. last – 基本上都用这个Flag，表示完成Rewrite
2. break – 中止Rewirte，不在继续匹配
3. redirect – 返回临时重定向的HTTP状态302，地址栏显示跳转后的地址
4. permanent – 返回永久重定向的HTTP状态301，地址栏显示跳转后的地址

## last和break的区别

1. last一般写在server和if中，而break一般使用在location中
2. last不终止重写后的url匹配，即新的url会再从server走一遍匹配流程，而break终止重写后的匹配
3. break和last都能组织继续执行后面的rewrite指令

## redirect 和 permanent

1. 临时重定向：对旧网址没有影响，但新网址不会有排名
2. 永久重定向：新网址完全继承旧网址，旧网址的排名等完全清零


