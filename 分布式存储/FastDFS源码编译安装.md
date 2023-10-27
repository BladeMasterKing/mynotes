# FastDFS源码编译安装


## 安装libfastcommon

在 [Github余庆大神的仓库](https://github.com/happyfish100/libfastcommon/archive/V1.0.39.tar.gz) 上下载libfastcommon包

```bash
# 解压tar.gz包
$ tar -zxvf libfastcommon-1.0.39.tar.gz
# 进入根目录，编译安装
$ cd  libfastcommon-1.0.39
$ ./make.sh && ./make.sh install                     make: Nothing to be done for 'all'.

# 以下是编译安装输出的日志内容
mkdir -p /usr/lib64
mkdir -p /usr/lib
mkdir -p /usr/include/fastcommon
install -m 755 libfastcommon.so /usr/lib64
install -m 644 common_define.h hash.h chain.h logger.h base64.h shared_func.h pthread_func.h ini_file_reader.h _os_define.h sockopt.h sched_thread.h http_func.h md5.h local_ip_func.h avl_tree.h ioevent.h ioevent_loop.h fast_task_queue.h fast_timer.h process_ctrl.h fast_mblock.h connection_pool.h fast_mpool.h fast_allocator.h fast_buffer.h skiplist.h multi_skiplist.h flat_skiplist.h skiplist_common.h system_info.h fast_blocked_queue.h php7_ext_wrapper.h id_generator.h char_converter.h char_convert_loader.h common_blocked_queue.h multi_socket_client.h skiplist_set.h fc_list.h /usr/include/fastcommon
if [ ! -e /usr/lib/libfastcommon.so ]; then ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so; fi
```

## 安装FastDFS

在 [Github余庆大神的仓库](https://github.com/happyfish100/fastdfs/archive/V5.11.tar.gz) 上下载FastDFS的源码

```basu
# 解压FastDFS的tar.gz包
$ tar -zxvf fastdfs-5.11.tar.gz
$ cd fastdfs-5.11
# 编译源码并安装
$ ./make.sh && ./make.sh install

# 以下是编译输出的日志内容
mkdir -p /usr/bin
mkdir -p /etc/fdfs
cp -f fdfs_trackerd /usr/bin
if [ ! -f /etc/fdfs/tracker.conf.sample ]; then cp -f ../conf/tracker.conf /etc/fdfs/tracker.conf.sample; fi
if [ ! -f /etc/fdfs/storage_ids.conf.sample ]; then cp -f ../conf/storage_ids.conf /etc/fdfs/storage_ids.conf.sample; fimkdir -p /usr/bin
mkdir -p /etc/fdfs
cp -f fdfs_storaged  /usr/bin
if [ ! -f /etc/fdfs/storage.conf.sample ]; then cp -f ../conf/storage.conf /etc/fdfs/storage.conf.sample; fi
mkdir -p /usr/bin
mkdir -p /etc/fdfs
mkdir -p /usr/lib64
mkdir -p /usr/lib
cp -f fdfs_monitor fdfs_test fdfs_test1 fdfs_crc32 fdfs_upload_file fdfs_download_file fdfs_delete_file fdfs_file_info fdfs_appender_test fdfs_appender_test1 fdfs_append_file fdfs_upload_appender /usr/bin
if [ 0 -eq 1 ]; then cp -f libfdfsclient.a /usr/lib64; cp -f libfdfsclient.a /usr/lib/;fi
if [ 1 -eq 1 ]; then cp -f libfdfsclient.so /usr/lib64; cp -f libfdfsclient.so /usr/lib/;fi
mkdir -p /usr/include/fastdfs
cp -f ../common/fdfs_define.h ../common/fdfs_global.h ../common/mime_file_parser.h ../common/fdfs_http_shared.h ../tracker/tracker_types.h ../tracker/tracker_proto.h ../tracker/fdfs_shared_func.h ../storage/trunk_mgr/trunk_shared.h tracker_client.h storage_client.h storage_client1.h client_func.h client_global.h fdfs_client.h /usr/include/fastdfs
if [ ! -f /etc/fdfs/client.conf.sample ]; then cp -f ../conf/client.conf /etc/fdfs/client.conf.sample; fi
```

*配置文件准备:*

```bash
$ cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf
$ cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf
$ cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf #客户端文件，测试用
# nginx相关文件未找到
$ cp /usr/local/src/fastdfs/conf/http.conf /etc/fdfs/ #供nginx访问使用
$ cp /usr/local/src/fastdfs/conf/mime.types /etc/fdfs/ #供nginx访问使用
$ ls -l /etc/fdfs/
total 32
-rwxr-xr-x 1 root root 1461 Sep 29 15:55 client.conf
-rwxr-xr-x 1 root root 1461 Sep 29 15:48 client.conf.sample
-rwxr-xr-x 1 root root 7927 Sep 29 15:55 storage.conf
-rwxr-xr-x 1 root root 7927 Sep 29 15:48 storage.conf.sample
-rwxr-xr-x 1 root root  105 Sep 29 15:48 storage_ids.conf.sample
-rwxr-xr-x 1 root root 7389 Sep 29 15:54 tracker.conf
-rwxr-xr-x 1 root root 7389 Sep 29 15:48 tracker.conf.sample
```

## 安装fastdfs-nginx-module

在 [Github余庆大神的仓库](https://github.com/happyfish100/fastdfs-nginx-module/archive/V1.20.tar.gz) 上下载fastdfs-nginx-module的源码:

```bash
# 编译nginx并安装
$ tar -zxvf nginx-1.10.1.tar.gz
$ cd nginx-1.10.1
# 调用nginx的configure命令,将fastdfs模块配置到nginx
$ ./configure --add-module=/mnt/e/DevelopKit/fastdfs/fastdfs-nginx-module-1.20/src
$ make && make install
```

*make安装nginx时报错:*

```bash
# make安装fastdfs-nginx-module时报错
In file included from /mnt/e/DevelopKit/fastdfs/fastdfs-nginx-module-1.20/src//common.c:26:0,
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

确定是因为高版本出现的问题,需要修改源码:
```bash
## fastdfs-nginx-module-1.20/src/config

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

*替换以下内容:*

```bash
ngx_module_incs="/usr/include/fastdfs /usr/include/fastcommon/"
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
```

再次调用 make && make install 成功安装。

NOTE: nginx编译安装完成之后，nginx的存放路径在 /usr/local/nginx 目录下，nginx的启动脚本配置文件都在这个目录下，不再使用源码包conf目录下的文件.

## 单机部署

* tracker配置:

```bash
# /etc/fdfs/tracker.conf

$ vim /etc/fdfs/tracker.conf
#需要修改的内容如下
port=22122  # tracker服务器端口（默认22122,一般不修改）
base_path=/home/dfs  # 存储日志和数据的根目录
    
# 启动tracker
$ service fdfs_trackerd start
$ /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf
    
# ARM架构启动tracker会报错：
[2018-04-25 23:37:36] ERROR - file: pthread_func.c, line: 121, call pthread_attr_setstacksize fail, errno: 22, error info: Invalid argument
[2018-04-25 23:37:36] ERROR - file: tracker_service.c, line: 78, init_pthread_attr fail, program exit!
[2018-04-25 23:37:36] CRIT - exit abnormally!
# 需要修改配置文件 /etc/fdfs/tracker.conf :
thread_statck_size=1024KB  # 原 64KB 改为 1024KB 即可
```
    
* storage配置:
```bash
# ./etc/fdfs/storage.conf

$ vim /etc/fdfs/storage.conf
# 需要修改的内容如下
port=23000  # storage服务端口（默认23000,一般不修改）
base_path=/mnt/e/DevelopKit/fastdfs_storage  # 数据和日志文件存储根目录
store_path0=/mnt/e/DevelopKit/fastdfs_storage_data  # 第一个存储目录
tracker_server=192.167.3.8:22122  # tracker服务器IP和端口,不能是127.0.0.1
http.server_port=8888  # http访问文件的端口(默认8888,看情况修改,和nginx中保持一致)


# 检验storage是否注册到tracker中
$ /usr/bin/fdfs_monitor /etc/fdfs/storage.conf
```


# 启动storage
```bash
$ service fdfs_storaged start
$ /usr/bin/fdfs_storaged /etc/fdfs/storage.conf
```

* client测试:

```bash
# /etc/fdfs/client.conf

$ vim /etc/fdfs/client.conf
# 需要修改的内容如下
base_path=/mnt/e/DevelopKit/fastdfs_tracker   #tracker服务器文件路径
tracker_server=192.168.52.1:22122    #tracker服务器IP和端口


# 保存后测试,返回ID表示成功
$ fdfs_upload_file /etc/fdfs/client.conf /mnt/e/DevelopKit/fastdfs/fastdfs-5.11.tar.gz
group1/M00/00/00/wKcDCF2dQiuAGERCAAUkK6yqBFI.tar.gz
```

* 配置nginx访问:

```bash
# /etc/fdfs/mod_fastdfs.conf

$ vim /etc/fdfs/mod_fastdfs.conf
# 需要修改的内容如下
base_path=/mnt/e/DevelopKit/fastdfs_storage  #保存日志目录
tracker_server=192.168.52.1:22122  #tracker服务器IP和端口
url_have_group_name=true
store_path0=/mnt/e/DevelopKit/fastdfs_storage_data

# 配置nginx.config,并非源码包./conf目录里,位置在 /usr/local/ngin/conf
$ vim /usr/local/nginx/conf/nginx.conf
# 添加如下配置
server {
    listen       80;    #该端口为storage.conf中的http.server_port相同
    server_name  localhost;
    location / {
        root   html;
        index  index.html index.htm;
    } 
    
    ...
    
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
    
    ...
    
    location ~/group[0-9]/ {
        # root /mnt/e/DevelopKit/fastdfs_storage_data/data;  
        ngx_fastdfs_module;
    }
}
```


# 启动nginx,脚本在编译完自动创建在/usr/local/nginx/sbin目录

```bash
$ /usr/local/nginx/sbin/nginx
```

测试下载，用外部浏览器访问刚才已传过的nginx安装包,引用返回的ID:  
http://192.167.3.8:80/group1/M00/00/00/wKcDCF2dQiuAGERCAAUkK6yqBFI.tar.gz  
弹出下载,至此单机部署全部跑通