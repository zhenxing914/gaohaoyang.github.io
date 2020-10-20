



## 1. Openresty简介

OpenResty® 是一个基于 [Nginx](https://openresty.org/cn/nginx.html) 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。

OpenResty® 通过汇聚各种设计精良的 [Nginx](https://openresty.org/cn/nginx.html) 模块（主要由 OpenResty 团队自主开发），从而将 [Nginx](https://openresty.org/cn/nginx.html) 有效地变成一个强大的通用 Web 应用平台。这样，Web 开发人员和系统工程师可以使用 Lua 脚本语言调动 [Nginx](https://openresty.org/cn/nginx.html) 支持的各种 C 以及 Lua 模块，快速构造出足以胜任 10K 乃至 1000K 以上单机并发连接的高性能 Web 应用系统。

OpenResty® 的目标是让你的Web服务直接跑在 [Nginx](https://openresty.org/cn/nginx.html) 服务内部，充分利用 [Nginx](https://openresty.org/cn/nginx.html) 的非阻塞 I/O 模型，不仅仅对 HTTP 客户端请求,甚至于对远程后端诸如 MySQL、PostgreSQL、Memcached 以及 Redis 等都进行一致的高性能响应。

参考 [组件](https://openresty.org/cn/components.html) 可以知道 OpenResty® 中包含了多少软件。

参考 [上路](https://openresty.org/cn/getting-started.html) 学习如何从最简单的 hello world 开始使用 OpenResty® 开发 HTTP 业务，或前往 [下载](https://openresty.org/cn/download.html) 直接获取 OpenResty® 的源代码包开始体验。

## 2. [官网安装说明](https://openresty.org/cn/installation.html)



![img](https://user-gold-cdn.xitu.io/2019/10/6/16da193bfdd24154?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





![img](https://user-gold-cdn.xitu.io/2019/10/6/16da193bfdc2fd37?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



根据官网的描述提供了不同的安装方式，例如：使用yum安装、源码编译安装等等。我目前暂时使用yum安装方式进行部署看看。

## 3. 设置安装的yum源

你可以在你的 CentOS 系统中添加 `openresty` 仓库，这样就可以便于未来安装或更新我们的软件包（通过 `yum update` 命令）。运行下面的命令就可以添加我们的仓库：

```
sudo yum install yum-utils -y
sudo yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo
复制代码
```

根据上面的命令，先执行看看，如下：



![img](https://user-gold-cdn.xitu.io/2019/10/6/16da193bfdb3838c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





![img](https://user-gold-cdn.xitu.io/2019/10/6/16da193bfde2ccac?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 安装Openresty

然后就可以像下面这样安装软件包，比如 `openresty`：

```
sudo yum install openresty -y
复制代码
```



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="980" height="748"></svg>)



### 安装命令行工具 `resty`

安装 `openresty-resty` 包：

```
sudo yum install openresty-resty -y
```



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="980" height="748"></svg>)



安装之后，就可以查看一下版本号，如下：

```
[root@centos7 ~]# resty -V
resty 0.21
nginx version: openresty/1.13.6.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) 
built with OpenSSL 1.1.0h  27 Mar 2018
TLS SNI support enabled
configure arguments: --prefix=/usr/local/openresty/nginx --with-cc-opt='-O2 -DNGX_LUA_ABORT_AT_PANIC -I/usr/local/openresty/zlib/include -I/usr/local/openresty/pcre/include -I/usr/local/openresty/openssl/include' --add-module=../ngx_devel_kit-0.3.0 --add-module=../echo-nginx-module-0.61 --add-module=../xss-nginx-module-0.06 --add-module=../ngx_coolkit-0.2rc3 --add-module=../set-misc-nginx-module-0.32 --add-module=../form-input-nginx-module-0.12 --add-module=../encrypted-session-nginx-module-0.08 --add-module=../srcache-nginx-module-0.31 --add-module=../ngx_lua-0.10.13 --add-module=../ngx_lua_upstream-0.07 --add-module=../headers-more-nginx-module-0.33 --add-module=../array-var-nginx-module-0.05 --add-module=../memc-nginx-module-0.19 --add-module=../redis2-nginx-module-0.15 --add-module=../redis-nginx-module-0.3.7 --add-module=../ngx_stream_lua-0.0.5 --with-ld-opt='-Wl,-rpath,/usr/local/openresty/luajit/lib -L/usr/local/openresty/zlib/lib -L/usr/local/openresty/pcre/lib -L/usr/local/openresty/openssl/lib -Wl,-rpath,/usr/local/openresty/zlib/lib:/usr/local/openresty/pcre/lib:/usr/local/openresty/openssl/lib' --with-pcre-jit --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module --with-http_v2_module --without-mail_pop3_module --without-mail_imap_module --without-mail_smtp_module --with-http_stub_status_module --with-http_realip_module --with-http_addition_module --with-http_auth_request_module --with-http_secure_link_module --with-http_random_index_module --with-http_gzip_static_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-threads --with-dtrace-probes --with-stream --with-stream_ssl_module --with-http_ssl_module
[root@centos7 ~]# 
复制代码
```

### 查看Opm工具

命令行工具 `opm` 在 `openresty-opm` 包里，而 `restydoc` 工具在 `openresty-doc` 包里头。

列出所有 `openresty` 仓库里头的软件包：

```bash
sudo yum --disablerepo="*" --enablerepo="openresty" list available

```



![img](https://user-gold-cdn.xitu.io/2019/10/6/16da193c33fc9d03?imageslim)



从上图可以看出，还有很多工具包可以安装。但是目前，我先不安装了。等到需要的时候，再进行安装。

参考 [OpenResty RPM 包](https://openresty.org/cn/rpm-packages.html)页面获取这些包更多的细节。



## 4. 入门

> 在上面已经安装好了Openresty之后，下面可以部署一个Hello world的示例。

### 创建相关工作目录

> 创建work目录以及相应的log日志目录、conf配置目录

```bash
mkdir ~/work
cd ~/work
mkdir logs/ conf/

```

配置命令：

```bash
[root@centos7 ~]# cd ~
[root@centos7 ~]# 
[root@centos7 ~]# mkdir work
[root@centos7 ~]# 
[root@centos7 ~]# cd work/
[root@centos7 work]# ls
[root@centos7 work]# mkdir logs
[root@centos7 work]# 
[root@centos7 work]# mkdir conf
[root@centos7 work]# 
[root@centos7 work]# ls
conf  logs
[root@centos7 work]# pwd
/root/work
[root@centos7 work]# 

```

创建`logs/`用于记录文件和`conf/`配置文件的目录。

### 准备nginx.conf配置文件

> 创建一个简单的纯文本文件，`conf/nginx.conf`其中包含以下内容：

```conf
worker_processes  1;
error_log logs/error.log;
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;
        location / {
            default_type text/html;
            content_by_lua '
                ngx.say("<p>hello, world</p>")
            ';
        }
    }
}
```



![img](https://user-gold-cdn.xitu.io/2019/10/6/16da193c38501e2a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



如果您熟悉[Nginx](https://openresty.org/cn/nginx.html)配置，那么您应该非常熟悉它。[无论如何，OpenResty](https://openresty.org/cn/openresty.html)只是一个增强版的 [Nginx](https://openresty.org/cn/nginx.html)。您可以充分利用[Nginx](https://openresty.org/cn/nginx.html)世界中所有现有的好东西。

### 启动[Nginx](https://openresty.org/cn/nginx.html)服务器

> 假设你已经安装了[OpenResty](https://openresty.org/cn/openresty.html)到`/usr/local/openresty`（这是默认值），我们使我们的`nginx`我们的可执行[OpenResty](https://openresty.org/cn/openresty.html)我们可用的安装`PATH`环境：

```bash
PATH=/usr/local/openresty/nginx/sbin:$PATH
export PATH
```

> 执行步骤如下

```bash
[root@centos7 conf]# ls /usr/local/openresty
bin  COPYRIGHT  luajit  lualib  nginx  openssl  pcre  site  zlib
[root@centos7 conf]# 
[root@centos7 conf]# ls /usr/local/openresty/nginx/
conf  html  logs  sbin  tapset
[root@centos7 conf]# ls /usr/local/openresty/nginx/sbin/
nginx  stap-nginx
[root@centos7 conf]# 
[root@centos7 conf]# PATH=/usr/local/openresty/nginx/sbin:$PATH
[root@centos7 conf]# export PATH
[root@centos7 conf]# env | grep open
PATH=/usr/local/openresty/nginx/sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@centos7 conf]# 
[root@centos7 conf]# nginx -V
nginx version: openresty/1.13.6.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) 
built with OpenSSL 1.1.0h  27 Mar 2018

```

> 然后我们以这种方式用我们的配置文件启动nginx服务器：

```bash
[root@centos7 conf]# cd ..
[root@centos7 work]# ls
conf  logs
[root@centos7 work]# pwd
/root/work
[root@centos7 work]# 
[root@centos7 work]# nginx -p `pwd`/ -c conf/nginx.conf 
[root@centos7 work]# ps -ef | grep nginx
root       2199   2181  0 14:57 ?        00:00:00 nginx: master process nginx -g daemon off;
104        2227   2199  0 14:57 ?        00:00:00 nginx: worker process
root       2489      1  0 15:41 ?        00:00:00 nginx: master process nginx -p /root/work/ -c conf/nginx.conf
nobody     2490   2489  0 15:41 ?        00:00:00 nginx: worker process
root       2504   1933  0 15:42 pts/0    00:00:00 grep --color=auto nginx
[root@centos7 work]# 
复制代码
```

错误消息将转到stderr设备或`logs/error.log`(当前工作目录中的默认错误日志文件)。

### 访问我们的HelloWorld Web服务

> 我们可以使用curl访问我们的HelloWorld新Web服务：

```bash
[root@centos7 work]# curl http://localhost:8080/

<p>hello, world</p>
[root@centos7 work]# 

```

如果一切正常，我们应该得到输出

```json
<p>hello, world</p>
```