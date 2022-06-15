# 搭建服务器

<!-- GFM-TOC -->
- [搭建服务器](#搭建服务器)
  - [吐槽](#吐槽)
  - [目的](#目的)
  - [搭建环境](#搭建环境)
    - [搭建科学上网环境](#搭建科学上网环境)
    - [搭建文件服务器](#搭建文件服务器)
      - [安装supervisord](#安装supervisord)
      - [nginx配置](#nginx配置)
<!-- GFM-TOC -->

## 吐槽
最近我的tx云服务器的ip被封了,无论我怎么配置sshd,防火墙都无效,气的我更换域名,重装机器到最后退还机器。我开始重新搭建服务了,因为有写博客,所以记录一下搭建过程。

新服务器用的阿里云的轻量应用服务器,操作系统使用的是阿里云的操作系统。

## 目的
大家一般买服务器做什么呢?

我归纳了一下我买服务器的目的
- [x] 搭建科学上网环境 
- [x] 搭建文件服务器
- [ ] 搭建个人博客(我用github和github pages写博客)
- [x] 搭建qqbot

## 搭建环境

### 搭建科学上网环境
[参考](https://www.hopol.cn/2015/05/239/)

我试了,可行的,而且脚本还把shadowsocks-go还加到开机启动了,很好用

```
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-go.sh
chmod +x shadowsocks-go.sh
./shadowsocks-go.sh 2>&1 | tee shadowsocks-go.log
```

### 搭建文件服务器
实现文件服务器最简单的方法,就是给服务器装一个nginx,去暴露服务器的某个文件夹,需要绑定一个域名用于wget下载文件,上传文件可以通过scp test.txt root@0.0.0.0:/data/file命令上传

[Nginx 安装](https://www.runoob.com/linux/nginx-install-setup.html)

[Nginx 安装常见问题](https://www.ancii.com/asey6ubbm/)

我按菜鸟教程安装的,重新安装了一下nginx,然后我知道为什么半年前我会觉得nginx难装,实际上确实很有点麻烦,第一,当时我也不懂./configure make make install的含义,只会百度,第二,菜鸟教程的安装教程确实过时了

我按菜鸟教程安装的,主要遇见了三个问题

1. 将警告当成了错误处理。

取nginx的objs/Makefile目录去掉CFLAGS中的-Werror,make就不会停下来了。

-Wall 表示打开gcc的所有警告
-Werror，它要求gcc将所有的警告当成错误进行处理

```
# 问题报错
src/core/ngx_murmurhash.c: In function ‘ngx_murmur_hash2’:
src/core/ngx_murmurhash.c:37:11: warning: this statement may fall through [-Wimplicit-fallthrough=]
   37 |         h ^= data[2] << 16;
      |         ~~^~~~~~~~~~~~~~~~
src/core/ngx_murmurhash.c:38:5: note: here
   38 |     case 2:
      |     ^~~~
src/core/ngx_murmurhash.c:39:11: warning: this statement may fall through [-Wimplicit-fallthrough=]
   39 |         h ^= data[1] << 8;
      |         ~~^~~~~~~~~~~~~~~
src/core/ngx_murmurhash.c:40:5: note: here
   40 |     case 1:
      |     ^~~~

```

2. nginx源码问题,struct crypt_data’没有名为‘current_salt’的成员：cd.current_salt[0] = ~salt[0]

```
vim src/os/unix/ngx_user.c
```

注释36行的 /* cd.current_salt[0] = ~salt[0]; */
![问题](https://cdn.ancii.com/article/image/v1/sw/wV/kP/PkwwVsGDmjDG9swnOi7SkjVAsMQAarn73E9S3mmSmcBMFCbn-jAQ_OUcu_NlK5IZSGYe1v6TwTI5T6kwYDpIiQ.png)

3. 更换openssl版本,默认yum安装的版本太高,需要自己下载低版本的,解压编译安装

```
wget http://www.openssl.org/source/openssl-1.1.0e.tar.gz
tar -zxvf openssl-1.1.0e.tar.gz
cd openssl-1.1.0e/ &&./config shared zlib --prefix=/usr/local/openssl && make && make install

cd /usr/local/openssl
./config -t
make depend

## 危险操作
<!-- cd /usr/local
ln -s openssl ssl
echo "/usr/local/openssl/lib" >>/etc/ld.so.conf

cd - 
ldconfig
echo $? -->

echo "PATH=$PATH:/usr/local/openssl/bin" >> /etc/profile && source /etc/profile
```

最后执行在nginx目录执行

```
./configure --prefix=/usr/local/webserver/nginx --with-http_stub_status_module  --with-pcre=/usr/local/src/pcre-8.35 --with-openssl=../opensll-1.1.0e
```

终于成功安装了


#### 安装supervisord
安装supervisor来管理nginx启动

```
yum install -y epel-release  #安装epel源
yum install -y supervisor    #安装supervisor
systemctl enable supervisord # 开机自启动
systemctl start supervisord  # 启动supervisord服务
systemctl status supervisord # 查看supervisord服务状态
ps -aux|grep supervisord     # 查看是否存在supervisord进程
vim /etc/supervisord.conf    #编辑配置文件
```

```
[program:nginx]
command=/usr/local/webserver/nginx/sbin/nginx
directory=/usr/local/webserver/nginx/sbin
autostart=true
autorestart=true
startsecs=5 
startretries=100 
stderr_logfile=/usr/local/webserver/nginx/sbin/error.log
stdout_logfile=/usr/local/webserver/nginx/sbin/info.log
```

#### nginx配置

```

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;
    autoindex on;
    autoindex_exact_size off;
    autoindex_localtime on;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

	default_type 'text/html';
        charset utf-8;
	client_max_body_size 1024m;	

        #access_log  logs/host.access.log  main;
        
        location /file{
	    root /data;
	}
 
        location / {
            root   html;
            index  index.html index.htm;
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
