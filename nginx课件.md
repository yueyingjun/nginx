# nginx

Nginx是一款轻量级的Web 服务器/反向代理服务器及。其特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好。Nginx常用作负载均衡服务，处理静态文件，反向代理等，中国大陆使用nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等

## 安装
下载必要组件
* nginx下载地址
        http://nginx.org/en/download.html

* pcre库下载地址，用于解析正则

        https://nchc.dl.sourceforge.net/project/pcre/pcre/8.42/pcre-8.42.tar.gz

* zlib下载地址，用来压缩文件
    
        http://www.zlib.net/zlib-1.2.11.tar.gz

* openssl下载地址，用来解析https协议
    
        https://github.com/openssl/openssl

在同级目录下,解压安装zlib、openssl、pcre

进入nginx目录，进行配置安装，下面可直接复制粘贴

    ./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_flv_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre=../pcre-8.42 --with-zlib=../zlib-1.2.11 --with-openssl=../openssl-OpenSSL_1_0_1r

**如果出现 ubuntu Invalid C++ compiler or C++ compiler flags 这个错误，请安装 sudo apt-get install g++**
编译安装

    $ make && sudo make install

Nginx会被安装在/usr/local/nginx目录下（也可以使用参数--prefix=指定自己需要的位置）， 安装成功后 /usr/local/nginx 目录下有四个子目录分别是：conf、html、logs、sbin 。 其中 Nginx 的配置文件存放于 conf/nginx.conf， bin文件是位于 sbin 目录下的 nginx 文件。 确保系统的 80 端口没被其他程序占用，运行 sbin/nginx 命令来启动 Nginx，

启动nginx

    $sudo /usr/local/nginx/sbin/nginx

打开浏览器访问此机器的 IP，如果浏览器出现 Welcome to nginx! 则表示 Nginx 已经安装并运行成功

nginx常用命令

        # 检查配置文件是否正确
        # /usr/local/nginx/sbin/nginx -t
        # 可以看到编译选项
        # /usr/local/nginx/sbin/nginx -V
        #重启Nginx
        #sudo /usr/local/nginx/sbin/nginx -s reload
        #关闭Nginx
        #sudo /usr/local/nginx/sbin/nginx -s stop
        #优雅停止服务
        #sudo /usr/local/nginx/sbin/nginx -s quit
        #kill -s SIGQUIT pid_master
        #kill -s SIGWINCH pid_master

## 正向代理
- 正向代理类似一个跳板机，代理访问外部资源。

<img src="zheng.png">

> 正向代理 是一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端必须要进行一些特别的设置才能使用正向代理。
- 正向代理的用途：
    - 访问原来无法访问的资源，如google
    - 可以做缓存，加速访问资源
    - 对客户端访问授权，上网进行认证
    - 代理可以记录用户访问记录（上网行为管理），对外隐藏用户信息

    ```
    server {
    resolver 192.168.1.1; #指定DNS服务器IP地址
    listen 8080;
    location / {
        proxy_pass http://$http_host$request_uri; #设定代理服务器的协议和地址
    }
}
    ```

## 反向代理
- 反向代理（Reverse Proxy）实际运行方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

- 反向代理的作用
  - 大型网站，通常将反向代理作为公网访问地址，Web服务器是内网
  <img src="fan.png">

  - 负载均衡，通过反向代理服务器来优化网站的负载

  <img src="fu.png">

## 配置

nginx.conf配置文件,基本就分为以下几块：

    main
    events   
    {
      ....
    
    }
    http        
    {
      ....
      upstream myproject 
    {
        .....
      
    }
      server  
    {
        ....
        location 
    {
            ....
        
    }
      }
      server  
    {
        ....
        location 
    {
            ....
        
    }
      }
      ....
    
    }

nginx配置文件主要分为六个区域：

* main(全局设置)
* events(nginx工作模式)
* http(http设置)
* sever(主机设置)
* location(URL匹配)
* upstream(负载均衡服务器设置)

下面依次来看下具体内容

* main模块

下面是一个main区域，他是一个全局的设置：

    user nobody nobody
    ;
    worker_processes 2
    ;
    error_log  /usr/local/var/log/nginx/error.log  notice
    ;
    pid        /usr/local/var/run/nginx/nginx.pid
    ;
    worker_rlimit_nofile 1024
    ;

user 来指定Nginx Worker进程运行用户以及用户组，默认由nobody账号运行。

worker_processes来指定了Nginx要开启的子进程数。每个Nginx进程平均耗费10M~12M内存。根据经验，一般指定1个进程就足够了，如果是多核CPU，建议指定和CPU的数量一样的进程数即可。我这里写2，那么就会开启2个子进程，总共3个进程。

error_log用来定义全局错误日志文件。日志输出级别有debug、info、notice、warn、error、crit可供选择，其中，debug输出日志最为最详细，而crit输出日志最少。

pid用来指定进程id的存储文件位置。

worker_rlimit_nofile用于指定一个nginx进程可以打开的最多文件描述符数目，这里是65535，需要使用命令“ulimit -n 65535”来设置。

* events 模块

events模块来用指定nginx的工作模式和工作模式及连接数上限，一般是这样：

    events 
    {
        use epoll
    ; #Linux平台
        worker_connections  1024
    ;
    }

use用来指定Nginx的工作模式。Nginx支持的工作模式有select、poll、kqueue、epoll、rtsig和/dev/poll。其中select和poll都是标准的工作模式，kqueue和epoll是高效的工作模式，不同的是epoll用在Linux平台上，而kqueue用在BSD系统中,对于Linux系统，epoll工作模式是首选。

worker_connections用于定义Nginx每个进程的最大连接数，即接收前端的最大请求数，默认是1024。最大客户端连接数由worker_processes和worker_connections决定，即Max_clients=worker_processes*worker_connections，在作为反向代理时，Max_clients变为：Max_clients = worker_processes * worker_connections/4。 进程的最大连接数受Linux系统进程的最大打开文件数限制，在执行操作系统命令“ulimit -n 65536”后worker_connections的设置才能生效。

* http 模块

http模块可以说是最核心的模块了，它负责HTTP服务器相关属性的配置，它里面的server和upstream子模块，至关重要，等到反向代理和负载均衡以及虚拟目录等会仔细说。

    http
    {
        include       mime.types
    ;
        default_type  application/octet-stream
    ;
        log_format  main  
    '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
        access_log  /usr/local/var/log/nginx/access.log  main
    ;
        sendfile        on
    ;
        tcp_nopush      on
    ;
        tcp_nodelay     on
    ;
        keepalive_timeout  10
    ;
        #gzip  on;
        upstream myproject 
    {
            .....
        
    }
        server 
    {
            ....
        
    }
    }

- 配置选项的含义。

include 来用设定文件的mime类型,类型在配置文件目录下的mime.type文件定义，来告诉nginx来识别文件类型。

default_type设定了默认的类型为二进制流，也就是当文件类型未定义时使用这种方式，例如在没有配置asp 的locate 环境时，Nginx是不予解析的，此时，用浏览器访问asp文件就会出现下载了。

log_format用于设置日志的格式，和记录哪些参数，这里设置为main，刚好用于access_log来纪录这种类型。

main的类型日志如下：也可以增删部分参数。

    127.0.0.1 - - 
    [21/Apr/2015:18:09:54 +0800
    ] "GET /index.php HTTP/1.1" 200 87151 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.76 Safari/537.36"

access_log

用来纪录每次的访问日志的文件地址，后面的main是日志的格式样式，对应于log_format的main。

sendfile参数用于开启高效文件传输模式。将tcp_nopush和tcp_nodelay两个指令设置为on用于防止网络阻塞。

keepalive_timeout设置客户端连接保持活动的超时时间。在超过这个时间之后，服务器会关闭该连接。

* server 模块

sever 模块是http的子模块，它用来定一个虚拟主机

基础的server配置

    server 
    {
            listen       8080
    ;
            server_name  localhost 192.168.12.10 www.yangyi.com
    ;
            # 全局定义，如果都是这一个目录，这样定义最简单。
            root   /Users/yangyi/www
    ;
            index  index.php index.md index.htm
    ; 
            charset utf-8
    ;
            access_log  usr/local/var/log/host.access.log  main
    ;
            aerror_log  usr/local/var/log/host.error.log  error
    ;
            ....
    
    }

server标志定义虚拟主机开始。

listen用于指定虚拟主机的服务端口。

server_name用来指定IP地址或者域名，多个域名之间用空格分开。

root 表示在这整个server虚拟主机内，全部的root web根目录。注意要和locate {}下面定义的区分开来。

index 全局定义访问的默认首页地址。注意要和locate {}下面定义的区分开来。

charset用于设置网页的默认编码格式。

access_log用来指定此虚拟主机的访问日志存放路径，最后的main用于指定访问日志的输出格式。

* location 模块

location模块是nginx中用的最多的，也是最重要的模块，负载均衡、反向代理、虚拟域名都与它相关

location 根据它字面意思就知道是来定位的，定位URL，解析URL，所以，它也提供了强大的正则匹配功能，也支持条件判断匹配，用户可以通过location指令实现Nginx对动、静态网页进行过滤处理。

我们先来看这个，设定默认首页和虚拟机目录。

    location / 
    {
                root   /Users/yangyi/www
    ;
                index  index.php index.md index.htm
    ;
            }

location /表示匹配访问根目录。

root指令用于指定访问根目录时，虚拟主机的web目录，这个目录可以是相对路径（相对路径是相对于nginx的安装目录）。也可以是绝对路径。

    #反向代理配置
      location /itcast/ 
    {
                 proxy_pass https://127.0.0.1:12345;
                 proxy_set_header X-real-ip $remote_addr;
                 proxy_set_header Host $http_host;
    }
    
    
      #采用uwsgi方式
      location /python/ 
    {
                 include uwsgi_params;
                 uwsgi_pass 127.0.0.1:33333;
             }
    
    
    
        #访问nginx本机目录的文件
        location / 
    {
                root   /home/itcast/xwp/itcast/
    ;
                index  index.md index.htm
    ;
            }
    
        location  /static/ 
    {
                 alias /var/static/
    ;
            }

* upstram 模块

upstream 模块负债负载均衡模块，通过调度算法来实现客户端IP到后端服务器的负载均衡。

    upstream test.com
    {
        ip_hash
    ;
        server 192.168.123.1:80
    ;
        server 192.168.123.2:80 down
    ;
        server 192.168.123.3:8080  
    max_fails=3  fail_timeout=20s
    ;
        server 192.168.123.4:8080
    ;
    }

在上面的例子中，通过upstream指令指定了一个负载均衡器的名称test.com。这个名称可以任意指定，在后面需要的地方直接调用即可。

里面是ip_hash这是其中的一种负载均衡调度算法。

Nginx的负载均衡模块目前支持4种调度算法:

* weight 轮询（默认）。每个请求按时间顺序逐一分配到不同的后端服务器，如果后端某台服务器宕机，故障系统被自动剔除，使用户访问不受影响。weight。指定轮询权值，weight值越大，分配到的访问机率越高，主要用于后端每个服务器性能不均的情况下。
* ip_hash。每个请求按访问IP的hash结果分配，这样来自同一个IP的访客固定访问一个后端服务器，有效解决了动态网页存在的session共享问题。
* fair。比上面两个更加智能的负载均衡算法。此种算法可以依据页面大小和加载时间长短智能地进行负载均衡，也就是根据后端服务器的响应时间来分配请求，响应时间短的优先分配。Nginx本身是不支持fair的，如果需要使用这种调度算法，必须下载Nginx的upstream_fair模块。
* url_hash。按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，可以进一步提高后端缓存服务器的效率。Nginx本身是不支持url_hash的，如果需要使用这种调度算法，必须安装Nginx 的hash软件包。

在HTTP Upstream模块中，可以通过server指令指定后端服务器的IP地址和端口，同时还可以设定每个后端服务器在负载均衡调度中的状态。常用的状态有：

down，表示当前的server暂时不参与负载均衡。

backup，预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的压力最轻。

max_fails，允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误。

fail_timeout，在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用。

注意 当负载调度算法为ip_hash时，后端服务器在负载均衡调度中的状态不能是weight和backup。

备注： nginx的worker_rlimit_nofile达到上限时，再有客户端链接报502错误. 用了log_format指令设置了日志格式之后，需要用access_log指令指定日志文件的存放路径.

## 负载均衡

负载均衡是由多台服务器以对称的方式组成一个服务器集合，每台服务器都具有等价的地位，都可以单独对外提供服务而无须其他服务器的辅助。通过某种负载分担技术，将外部发送来的请求按照事先设定分配算法分配到对称结构中的某一台服务器上，而接收到请求的服务器独立地回应客户的请求。

均衡负载能够平均分配客户请求到服务器列阵，籍此提供快速获取重要数据，解决大量并发访问服务问题。

1. upstream块
    
        upstream name {...}
        配置块 http
        upstream块定义一个上游服务器的集群,便于反向代理中的proxy_pass使用
        
        upstream mynet{
            server www.wopai1.com;
            server www.wopai2.com;
            server www.wopai3.com;
        }
        server {
            location /{
                proxy_pass http://mynet;
            }
        }

2. server
    
        server name [paramenters]
        配置块upstream
        server配置项指定了一台上游服务器的名字,可以是域名 IP地址端口 UNIX句柄
        weight= number;设置向这台服务器转发的权重,默认为1
        max_fails=number;该选项域fail_timeout配合使用
                指在fail_timeout时间段内如果转发上游失败超过number次就认为当前的fail_timeout时间内
                这台服务器不可用,max_fails默认为1 如果设置为0 表示不检查失败次数
        fail_timeout=time; fail_timeout表示该时间内转发多少次失败后就认为上游不可用.默认10s
        down    表示上游服务器永久下线,只能在ip_hash配置时才有效
        backup  在ip_hash配置时无效.只有所有非备份机都失败,才向上游备份服务器转发请求.
        upstream mynet{
            server www.wopai1.com weight=5;
            server www.wopai2.com:8081 max_fails=3 fail_timeout=300s;
            server www.wopai2.com down;
        }

3. ip_hash
    
        配置块 upstream
        希望来自某一个用户的请求始终落在固定的一台服务器上进行处理.
        根据客户端的IP散列计算出一个key,将key按照upstream集群中的上游服务器进行取模,求得的值对应的主机接收转发请求.
        ip_hash不可以与weight同时使用
        如果upstream配置中有一台服务器暂时不可用,不能直接删除该配置,而应该使用down标识.
        upstream mynet{
            ip_hash;
            server www.wowpai1.top;
            server www.wowpai2.top;  
            server www.wowpai3.top down;
        }

例子,服务器负载均衡基本配置,nginx中可以进行负载均衡的相关设置:

    upstream my.net
    {    #my.net是自定义的命名 在server结构中引用即可
    
            #代理服务器为 两台机器192.168.22.136 192.168.22.147做负载均衡操作 
            #两台机器上 可以跑apache负载功能更为强大的网页相关任务
    
            #max_fails 表示尝试出错最大次数 即可认为该服务器 在fail_timeout时间内不可用
            # server servername:port   servername可以写主机名 或者点分式IP
            server 192.168.22.136:80 
    max_fails=1 fail_timeout=300s
    ;
            server 192.168.22.147:80 
    max_fails=1 fail_timeout=300s
    ;  
            }
    
    
            server 
    {
            listen       80
    ;
            server_name  localhost
    ; 
            location / 
    {
                #upstream 块名
                proxy_pass http://my.net
    ;
                root   html
    ;
                index  index.md index.htm
    ;
            }

动手开始配置才能慢慢上手nginx配置和使用。

淘宝团队出品，高质量讲nginx的电子书

    http://tengine.taobao.org/book/
