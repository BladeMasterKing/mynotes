# Nginx源码编译安装

> 本次编译源码安装Nginx，主要是为了应服务器国产化的需求，支持国产麒麟操作系统(Ubuntu内核)以及arm架构的CPU指令集。项目中用到nginx主要是两个方面：FastDFS的代理和HTTPS协议的代理。


## FastDFS代理

这部分内容基于已成功编译安装**FastDFS v5.11**
使用Nginx代理FastDFS，需要配置Nginx安装fastdfs_nginx_module(https://github.com/happyfish100/fastdfs-nginx-module/archive/V1.20.tar.gz[源码下载地址])。

```bash
# 解压nginx源码包
$ tar -zxvf nginx-1.10.1.tar.gz

# 解压fastdfs_nginx_module源码包
$ tar -zxvf fastdfs-nginx-module-1.20.tar.gz

# 进入nginx源码根目录，配置nginx的fastdfs模块
$ cd nginx-1.10.1
$ ./configure --add-module=/var/T/nginx/fastdfs-nginx-module-1.20/src
```

执行完上述操作之后，会在根目录生成**Makefile**文件，接下来可以编译安装了

```bash
# 在nginx的根目录make安装nginx
$ make && make install
```

执行make编译安装失败，报错如下：

```bash
In file included from /mnt/e/DevelopKit/fastdfs fastdfs-nginx-module-1.20/src//common.c:26:0,
                from /mnt/e/DevelopKit/fastdfs/fastdfs-nginx-module-1.20/src//ngx_http_fastdfs_module.c:6:
/usr/include/fastdfs/fdfs_define.h:15:27: fatal error: common_define.h: No such file or directory
compilation terminated.
objs/Makefile:1151: recipe for target
'objs/addon/src/ngx_http_fastdfs_module.o' failed
make[1]: *** [objs/addon/src/ngx_http_fastdfs_module.o] Error 1
make[1]: Leaving directory '/mnt/e/DevelopKit/nginx-1.10.1'
Makefile:8: recipe for target 'build' failed
make: *** [build] Error 2
```

这是由于fastdfs_nginx_module模块的版本与FastDFS的版本之间有出入，需要修改部分配置文件:

```bash
$ vim fastdfs-nginx-module-1.20/src/config


ngx_addon_name=ngx_http_fastdfs_module

if test -n "${ngx_module_link}"; then
ngx_module_type=HTTP
ngx_module_name=$ngx_addon_name
ngx_module_incs="/usr/include/fastdfs /usr/include/fastcommon/"
ngx_module_libs="-lfastcommon -lfdfsclient"
ngx_module_srcs="$ngx_addon_dir/ngx_http_fastdfs_module.c"
ngx_module_deps=
CFLAGS="$CFLAGS -D_FILE_OFFSET_BITS=64 -DFDFS_OUTPUT_CHUNK_SIZE='2561024' -DFDFS_MOD_CONF_FILENAME='"/etc/fdfs/mod_fastdfs.conf"'"
. auto/module
else
HTTP_MODULES="$HTTP_MODULES ngx_http_fastdfs_module"
NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_fastdfs_module.c"
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
CORE_LIBS="$CORE_LIBS -lfastcommon -lfdfsclient"
CFLAGS="$CFLAGS -D_FILE_OFFSET_BITS=64 -DFDFS_OUTPUT_CHUNK_SIZE='2561024' -DFDFS_MOD_CONF_FILENAME='"/etc/fdfs/mod_fastdfs.conf"'"
fi
```

替换以下内容:

```bash
ngx_module_incs="/usr/include/fastdfs /usr/include/fastcommon/"
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
```

再次回到nginx的根目录，执行**make && make install**，安装成功。
> nginx编译安装完成后，会将配置文件存放到/usr/local/nginx路径，可以将整个文件夹移动到其他软件整体管理的位置。

## HTTPS代理
### 安装http_ssl_module
通过nginx代理实现HTTPS协议，需要在配置nginx的时候启用http_ssl_module模块，结合fastdfs_nginx_module模块配置nginx的命令最终为：

```bash
$ ./configure --add-module=/fastdfs-nginx-module-1.20/src --with-http_ssl_module
```

在国产化服务器ARM架构CPU的系统环境下，执行**openssl version**命令查看openssl的安装版本为**OpenSSL 1.0.2g-fips 1 Mar 2016**，与Ubuntu-16.04默认安装的OpenSSL的版本一致。  
然后执行上述配置nginx的命令时报错：

```bash
checking for OpenSSL library ... not found
checking for OpenSSL library in /usr/local/ ... not found
checking for OpenSSL library in /usr/pkg/ ... not found
checking for OpenSSL library in /opt/local/ ... not found

./configure: error: SSL modules require the OpenSSL library.
You can either do not enable the modules, or install the OpenSSL library
into the system, or build the OpenSSL library statically from the source
with nginx by using --with-openssl=<path> option.
```

提示并未找到openssl类库即libssl-dev，在 [Ununtu依赖库搜索网站](https://launchpad.net/ubuntu) 查找libssl-dev以及依赖的类库：

* AMD架构版本
** libssl-dev_1.0.2g-1ubuntu4_amd64.deb
** zlib1g-dev_1.2.8.dfsg-2ubuntu4.1_amd64.deb
** libssl1.0.0_1.0.2g-1ubuntu4_amd64.deb
* ARM架构版本
** libssl-dev_1.0.2g-1ubuntu4_arm64.deb
** zlib1g-dev_1.2.8.dfsg-2ubuntu4_arm64.deb
** libssl1.0.0_1.0.2g-1ubuntu4_arm64.deb

通过Ubuntu的dpkg命令安装即可，**注意**安装顺序libssl-dev在最后

```bash
$ dpkg -i libssl1.0.0_1.0.2g-1ubuntu4_arm64.deb
$ dpkg -i zlib1g-dev_1.2.8.dfsg-2ubuntu4_arm64.deb
$ dpkg -i libssl-dev_1.0.2g-1ubuntu4_arm64.deb
```

重新编译安装nginx成功安装

### 使用 OpenSSL 生成 SSL Key 和 CSR 文件

配置HTTPS需要私钥 xxx.key 和 xxx.crt 证书文件，申请证书文件需要用到 xxx.csr 文件，OpenSSL 命令可以生成 xxx.key 和 xxx.csr 文件。
**CSR ( Cerificate Signing Request )**: 证书签署请求文件，里面包含申请者的DN( Distinguished Name, 标识名) 和公钥信息, 在第三方证书颁发机构签署证书的时候需要提供. 证书颁发机构拿到CSR后使用其根证书私钥对证书进行加密并生成CRT证书文件, 里面包含证书加密信息以及申请者的DN及公钥信息.
**Key**: 证书申请者私钥文件, 和证书里面的公钥配对使用, 在HTTPS"握手"通讯过程需要使用私钥去解密客户端发来的经过证书公钥加密的随机数信息, 是HTTPS加密通讯过程非常重要的文件, 在配置HTTPS的是够使用.
OpenSSL生成 xxx.key 和 xxx.csr 文件的命令:

```bash
openssl req -new -newkey rsa:2048 -sha256 -nodes -out tsm-aps.csr -keyout private.key -subj "/C=CN/ST=Beijing/L=Beijing/O=Topsec/OU=aps/CN=topsec.com.cn"
```

命令中相关字段含义:

* C : Country, 单位所在国家, 两个字母的国家英文缩写, CN为中国
* ST : State/Province, 单位所在州/省
* L : Locality, 单位所在城市
* O : Organization, 组织名
* OU : Organization Unit, 下属部门名称, 常常用于产品名
* CN : Common Name, 域名

生成csr文件后, 提供给CA机构, 签署成功后,会得到一个 xxx.crt 证书文件, 得到SSL证书文件可以在nginx中配置HTTPS.
签署证书命令:

```bash
openssl x509 -req -in tsm-aps.csr -extensions v3_ca -signkey private.key -out tsm-aps.crt
```

### 配置HTTPS
要启用HTTPS服务, 必须使用监听命令**listen**的**ssl**参数和定义服务器证书文件和私钥文件:

```bash
server {
    #ssl参数
    listen              0.0.0.0:443 ssl;
    server_name         example.com;
    ssl_certificate     example.com.crt;    #证书文件
    ssl_certificate_key example.com.key;    #私钥文件
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    #...
}
```


## 官方对configure的介绍
使用configure命令(源码根目录)配置构建。它定义了系统的各个方面，包括允许nginx用于连接处理的方法。最后，它会创建一个Makefile。
该configure命令支持以下参数：

* **--help** +
打印帮助信息。
* **--prefix=path** +
定义将保留服务器文件的目录。此相同目录还将用于设置的所有相对路径configure(库源路径除外)和nginx.conf配置文件中。/usr/local/nginx默认情况下设置为目录。
* **--sbin-path=path** +
设置nginx可执行文件的名称。此名称仅在安装期间使用。默认情况下，文件名为prefix/sbin/nginx。
* **--modules-path=path** +
定义将在其中安装nginx动态模块的目录。默认情况下使用prefix/modules目录。
* **--conf-path=path** +
设置nginx.conf配置文件的名称。如果需要，可以通过在命令行参数中指定nginx来始终使用其他配置文件来启动它 。默认情况下，文件名为 。 -c fileprefix/conf/nginx.conf
* **--error-log-path=path** +
设置主要错误，警告和诊断文件的名称。安装后，可以始终nginx.conf使用error_log伪指令在配置文件中 更改文件名 。默认情况下，文件名为prefix/logs/error.log。
* **--pid-path=path** +
设置nginx.pid将存储主进程的进程ID的文件名。安装后，可以始终nginx.conf使用pid伪指令在配置文件中 更改文件名 。默认情况下，文件名为 prefix/logs/nginx.pid。
* **--lock-path=path** +
为锁定文件的名称设置前缀。安装后，可以始终nginx.conf使用lock_file伪指令在配置文件中 更改该值 。默认情况下，值为 prefix/logs/nginx.lock。
* **--user=name** +
设置一个非特权用户的名称，其凭据将由工作进程使用。安装后，可以始终nginx.conf使用用户指令在配置文件中 更改名称 。默认用户名是nobody。
* **--group=name** +
设置其凭据将由工作进程使用的组的名称。安装后，可以始终nginx.conf使用用户指令在配置文件中 更改名称 。默认情况下，组名称设置为非特权用户的名称。
* **--build=name** +
设置一个可选的nginx构建名称。
* **--builddir=path** +
设置构建目录。
* **--with-select_module**  
* **--without-select_module** +
启用或禁用构建允许服务器使用该select()方法的模块。如果平台似乎不支持kqueue，epoll或/ dev / poll等更合适的方法，则会自动构建此模块。
* **--with-poll_module**  
* **--without-poll_module** +
启用或禁用构建允许服务器使用该poll()方法的模块。如果平台似乎不支持kqueue，epoll或/ dev / poll等更合适的方法，则会自动构建此模块。
* **--with-threads** +
启用线程池的使用 。
* **--with-file-aio** +
支持 在FreeBSD和Linux上使用 异步文件I / O（AIO）。
* **--with-http_ssl_module**  
启用构建将HTTPS协议支持添加 到HTTP服务器的模块的功能。默认情况下未构建此模块。需要OpenSSL库来构建和运行此模块。
* **--with-http_v2_module**  
支持构建一个模块，该模块提供对HTTP / 2的支持 。默认情况下未构建此模块。
* **--with-http_realip_module**  
启用构建ngx_http_realip_module 模块的功能，该 模块将客户端地址更改为在指定的标头字段中发送的地址。默认情况下未构建此模块。
* **--with-http_addition_module**  
允许构建ngx_http_addition_module模块，该 模块在响应之前和之后添加文本。默认情况下未构建此模块。
* **--with-http_xslt_module**
* **--with-http_xslt_module=dynamic**  
支持构建ngx_http_xslt_module模块，该模块使用一个或多个XSLT样式表转换XML响应。默认情况下未构建此模块。该libxml2的和 的libxslt库需要构建和运行此模块。
* **--with-http_image_filter_module**
* **--with-http_image_filter_module=dynamic**  
支持构建ngx_http_image_filter_module模块，该模块可以转换JPEG，GIF，PNG和WebP格式的图像。默认情况下未构建此模块。
* **--with-http_geoip_module**
* **--with-http_geoip_module=dynamic**  
支持构建ngx_http_geoip_module模块，该模块根据客户端IP地址和预编译的MaxMind数据库创建变量 。默认情况下未构建此模块。
* **--with-http_sub_module**  
支持构建ngx_http_sub_module 模块，该 模块通过将一个指定的字符串替换为另一个指定的字符串来修改响应。默认情况下未构建此模块。
* **--with-http_dav_module**  
支持构建ngx_http_dav_module 模块，该 模块通过WebDAV协议提供文件管理自动化。默认情况下未构建此模块。
* **--with-http_flv_module**  
支持构建ngx_http_flv_module 模块，该 模块为Flash Video（FLV）文件提供伪流服务器端支持。默认情况下未构建此模块。
* **--with-http_mp4_module**  
支持构建ngx_http_mp4_module模块，该模块为MP4文件提供伪流服务器端支持。默认情况下未构建此模块。
* **--with-http_gunzip_module**  
支持为不支持"gzip"编码方法的客户端构建ngx_http_gunzip_module 模块，该模块使用" Content-Encoding: gzip"解压缩响应。默认情况下未构建此模块。
* **--with-http_gzip_static_module**  
支持构建ngx_http_gzip_static_module模块，该模块支持发送扩展名为".gz"的预压缩文件，而不是常规文件。默认情况下未构建此模块。
* **--with-http_auth_request_module**  
允许构建ngx_http_auth_request_module模块，该模块基于子请求的结果实现客户端授权。默认情况下未构建此模块。
* **--with-http_random_index_module**  
支持构建ngx_http_random_index_module模块，该模块处理以斜杠（'/'）结尾的请求，并从目录中选择一个随机文件作为索引文件。默认情况下未构建此模块。
* **--with-http_secure_link_module**  
启用构建 ngx_http_secure_link_module模块。默认情况下未构建此模块。
* **--with-http_degradation_module**  
启用构建 ngx_http_degradation_module模块。默认情况下未构建此模块。
* **--with-http_slice_module**  
支持构建ngx_http_slice_module 模块，该 模块将请求拆分为子请求，每个子请求返回一定范围的响应。该模块提供了更有效的大响应缓存。默认情况下未构建此模块。
* **--with-http_stub_status_module**  
支持构建ngx_http_stub_status_module模块，该模块提供对基本状态信息的访问。默认情况下未构建此模块。
* **--without-http_charset_module**  
禁用构建ngx_http_charset_module模块，该模块将指定的字符集添加到"Content-Type"响应头字段中，并且可以将数据从一个字符集转换为另一个字符集。
* **--without-http_gzip_module**  
禁用构建可压缩HTTP服务器响应的模块。zlib库是构建和运行此模块所必需的。
* **--without-http_ssi_module**  
禁用构建处理通过SSI(服务器端包含)命令的ngx_http_ssi_module模块的响应。
* **--without-http_userid_module**  
禁用构建ngx_http_userid_module模块，该模块设置适用于客户端标识的cookie。
* **--without-http_access_module**  
禁用构建ngx_http_access_module模块，该模块允许限制对某些客户端地址的访问。
* **--without-http_auth_basic_module**  
禁用构建ngx_http_auth_basic_module模块，该模块允许通过使用"HTTP基本身份验证"协议验证用户名和密码来限制对资源的访问。
* **--without-http_mirror_module**  
禁用构建ngx_http_mirror_module模块，该模块通过创建后台镜像子请求来实现原始请求的镜像。
* **--without-http_autoindex_module**  
禁用构建ngx_http_autoindex_module模块，以处理以斜杠('/')结尾的请求，并在ngx_http_index_module模块找不到索引文件的情况下生成目录列表 。
* **--without-http_geo_module**  
禁用构建ngx_http_geo_module模块，该模块创建的变量的值取决于客户端IP地址。
* **--without-http_map_module**  
禁用构建ngx_http_map_module模块，该模块创建的变量的值取决于其他变量的值。
* **--without-http_split_clients_module**  
禁用构建ngx_http_split_clients_module模块，该模块创建用于A/B测试的变量。
* **--without-http_referer_module**  
禁用构建ngx_http_referer_module模块，该模块可以阻止对"Referer"标头字段中具有无效值的请求的站点访问。
* **--without-http_rewrite_module**  
禁用构建允许HTTP服务器重定向请求和更改请求URI的模块。构建和运行此模块需要PCRE库。
* **--without-http_proxy_module**  
禁用构建HTTP服务器代理模块。
* **--without-http_fastcgi_module**  
禁用构建将请求传递到FastCGI服务器的ngx_http_fastcgi_module模块。
* **--without-http_uwsgi_module**  
禁用构建将请求传递到uwsgi服务器的ngx_http_uwsgi_module模块。
* **--without-http_scgi_module**  
禁用构建将请求传递到SCGI服务器的ngx_http_scgi_module模块。
* **--without-http_grpc_module**  
禁用构建将请求传递到GRPC服务器的ngx_http_grpc_module模块。
* **--without-http_memcached_module**  
禁用构建ngx_http_memcached_module模块，该模块从memcached服务器获取响应。
* **--without-http_limit_conn_module**  
禁用构建ngx_http_limit_conn_module模块，该模块限制每个键的连接数，例如，单个IP地址的连接数。
* **--without-http_limit_req_module**  
禁用构建ngx_http_limit_req_module模块，该模块限制每个密钥的请求处理速率，例如，来自单个IP地址的请求的处理速率。
* **--without-http_empty_gif_module**  
禁用构建发出单像素透明GIF的模块。
* **--without-http_browser_module**  
禁用构建ngx_http_browser_module模块，该模块创建的变量的值取决于"User-Agent"请求头字段的值。
* **--without-http_upstream_hash_module**  
禁用构建实现哈希负载平衡方法的模块 。
* **--without-http_upstream_ip_hash_module**  
禁用构建实现ip_hash负载平衡方法的模块 。
* **--without-http_upstream_least_conn_module**  
禁用构建实现了minimum_conn 负载平衡方法的模块。
* **--without-http_upstream_keepalive_module**  
禁用构建一个模块来提供对上游服务器连接的缓存。
* **--without-http_upstream_zone_module**  
禁用构建模块，该模块可以将上游组的运行时状态存储在共享内存区域中。
* **--with-http_perl_module**  
* **--with-http_perl_module=dynamic**  
支持构建嵌入式Perl模块。默认情况下未构建此模块。
* **--with-perl_modules_path=path**  
定义一个目录，该目录将保留Perl模块。
* **--with-perl=path**  
设置Perl二进制文件的名称。
* **--http-log-path=path**  
设置HTTP服务器的主请求日志文件的名称。安装后，可以始终nginx.conf使用access_log伪指令在配置文件中 更改文件名 。默认情况下，文件名为 prefix/logs/access.log。
* **--http-client-body-temp-path=path**  
定义用于存储包含客户端请求正文的临时文件的目录。安装后，可以始终nginx.conf使用client_body_temp_path指令在配置文件中更改目录。默认情况下，目录名为prefix/client_body_temp。
* **--http-proxy-temp-path=path**  
定义一个目录，用于存储包含从代理服务器接收到的数据的临时文件。安装后，可以始终nginx.conf使用proxy_temp_path 指令在配置文件中 更改目录 。默认情况下，目录名为 prefix/proxy_temp。
* **--http-fastcgi-temp-path=path**  
定义一个目录，用于存储包含从FastCGI服务器接收到的数据的临时文件。安装后，可以始终nginx.conf使用fastcgi_temp_path指令在配置文件中更改目录。默认情况下，目录名为 prefix/fastcgi_temp。
* **--http-uwsgi-temp-path=path**  
定义一个目录，用于存储带有从uwsgi服务器接收到的数据的临时文件。安装后，可以始终nginx.conf使用uwsgi_temp_path指令在配置文件中更改目录。默认情况下，目录名为prefix/uwsgi_temp。
* **--http-scgi-temp-path=path**  
定义一个目录，用于存储带有从SCGI服务器接收到的数据的临时文件。安装后，可以始终nginx.conf使用scgi_temp_path指令在配置文件中更改目录。默认情况下，目录名为prefix/scgi_temp。
* **--without-http**
禁用HTTP服务器。
* **--without-http-cache**  
禁用HTTP缓存。
* **--with-mail**
* **--with-mail=dynamic**  
启用POP3/IMAP4/SMTP邮件代理服务器。
* **--with-mail_ssl_module**  
支持构建一个模块，该模块 向邮件代理服务器添加 SSL / TLS协议支持。默认情况下未构建此模块。需要OpenSSL库来构建和运行此模块。
* **--without-mail_pop3_module**  
在邮件代理服务器中 禁用POP3协议。
* **--without-mail_imap_module**  
在邮件代理服务器中 禁用IMAP协议。
* **--without-mail_smtp_module**  
在邮件代理服务器中 禁用SMTP协议。
* **--with-stream**  
* **--with-stream=dynamic**  
支持构建 用于通用TCP / UDP代理和负载平衡的 流模块。默认情况下未构建此模块。
* **--with-stream_ssl_module**  
支持构建一个模块，该模块 向流模块添加 SSL / TLS协议支持。默认情况下未构建此模块。需要OpenSSL库来构建和运行此模块。
* **--with-stream_realip_module**  
启用构建ngx_stream_realip_module模块的功能，该模块将客户端地址更改为PROXY协议标头中发送的地址。默认情况下未构建此模块。--with-stream_geoip_module
* **--with-stream_geoip_module=dynamic**  
支持构建ngx_stream_geoip_module 模块，该 模块根据客户端IP地址和预编译的MaxMind数据库创建变量 。默认情况下未构建此模块。
* **--with-stream_ssl_preread_module**  
支持构建ngx_stream_ssl_preread_module 模块，该 模块允许从ClientHello 消息中提取信息， 而无需终止SSL / TLS。默认情况下未构建此模块。
* **--without-stream_limit_conn_module**  
禁用构建ngx_stream_limit_conn_module 模块，该 模块限制每个键的连接数，例如，单个IP地址的连接数。
* **--without-stream_access_module**  
禁用构建ngx_stream_access_module 模块，该 模块允许限制对某些客户端地址的访问。
* **--without-stream_geo_module**  
禁用构建ngx_stream_geo_module 模块，该 模块创建的变量值取决于客户端IP地址。
* **--without-stream_map_module**  
禁用构建ngx_stream_map_module 模块，该 模块创建的变量值取决于其他变量的值。
* **--without-stream_split_clients_module**  
禁用构建ngx_stream_split_clients_module 模块，该 模块创建用于A / B测试的变量。
* **--without-stream_return_module**  
禁用构建ngx_stream_return_module 模块，该 模块向客户端发送一些指定的值，然后关闭连接。
* **--without-stream_upstream_hash_module**  
禁用构建实现哈希 负载平衡方法的模块 。
* **--without-stream_upstream_least_conn_module**  
禁用构建实现了minimum_conn 负载平衡方法的模块 。
* **--without-stream_upstream_zone_module**  
禁用构建模块，该模块可以将上游组的运行时状态存储在共享内存 区域中。
* **--with-google_perftools_module**  
允许构建ngx_google_perftools_module 模块，该 模块可以使用Google Performance Tools对nginx工作进程进行 性能分析。该模块适用于Nginx开发人员，默认情况下未构建。
* **--with-cpp_test_module**  
启用构建 ngx_cpp_test_module模块。
* **--add-module=path**  
启用外部模块。
* **--add-dynamic-module=path**  
启用外部动态模块。
* **--with-compat**  
启用动态模块兼容性。
* **--with-cc=path**  
设置C编译器的名称。
* **--with-cpp=path**  
设置C预处理器的名称。
* **--with-cc-opt=parameters**  
设置将添加到CFLAGS变量的其他参数。在FreeBSD下使用系统PCRE库时,--with-cc-opt="-I /usr/local/include" 应指定。如果select()需要增加支持的文件数量，也可以在此处指定，例如:--with-cc-opt="-D FD_SETSIZE=2048"。
* **--with-ld-opt=parameters**  
设置将在链接期间使用的其他参数。在FreeBSD下使用系统PCRE库时， --with-ld-opt="-L /usr/local/lib" 应指定。
* **--with-cpu-opt=cpu**  
支持按指定的CPU建设： pentium，pentiumpro， pentium3，pentium4， athlon，opteron， sparc32，sparc64， ppc64。
* **--without-pcre**  
禁用PCRE库的使用。
* **--with-pcre**  
强制使用PCRE库。
* **--with-pcre=path**  
设置PCRE库源的路径。需要从PCRE站点下载并分发库分发（版本4.4 — 8.43） 。其余的由nginx的./configure和完成 make。该库对于location指令中的正则表达式支持和 ngx_http_rewrite_module 模块是必需的 。
* **--with-pcre-opt=parameters**  
为PCRE设置其他构建选项。
* **--with-pcre-jit**  
使用“及时编译”支持（1.1.12，pcre_jit指令）构建PCRE库 。
* **--with-zlib=path**  
设置zlib库源的路径。需要从zlib站点下载库发行版（版本1.1.3-1.2.11） 并解压缩。其余的由nginx的./configure和完成 make。ngx_http_gzip_module模块需要该库 。
* **--with-zlib-opt=parameters**  
为zlib设置其他构建选项。
* **--with-zlib-asm=cpu**  
使得能够使用指定的CPU中的一个优化的zlib汇编源程序： pentium，pentiumpro。
* **--with-libatomic**  
强制使用libatomic_ops库。
* **--with-libatomic=path**  
设置libatomic_ops库源的路径。
* **--with-openssl=path**  
设置OpenSSL源码的路径。
* **--with-openssl-opt=parameters**  
为OpenSSL设置其他构建选项。
* **--with-debug**  
启用调试日志。

参数用法示例（所有这些都需要在一行中键入）：

```bash
./configure 
    --sbin-path= /usr/local/nginx/nginx
    --conf-path= /usr/local/nginx/nginx.conf 
    --pid-path= /usr/local/nginx/nginx.pid 
    --with-http_ssl_module
    --with-pcre=../pcre-8.43 
    --with-zlib=../zlib-1.2.11
```

执行完./configure配置后，nginx源码已经编译完成并生成Makefile文件，之后可以通过make命令来安装

```bash
make && make install
```