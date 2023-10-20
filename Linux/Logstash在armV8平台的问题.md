# Logstash在armV8平台的问题


> 测试armV8架构CPU的国产Linux操作系统遇到Logstash启动报错，报错信息如下：

```bash
[2019-10-14T10:49:50,592][ERROR][org.logstash.Logstash    ] java.lang.IllegalStateException: Logstash stopped processing because of an error: (LoadError) load error: ffi/ffi -- java.lang.NullPointerException: null
```

首先百度遇到的问题并没有合适的答案，灵光一现logstash的源代码托管在github上，在 https://github.com/elastic/logstash/issues/10888[logstash的 issue 10888]中找到了解决办法:  


```bash
# jar包地址
/var/T/plug/logstash/logstash-core/lib/jars/jruby-complete-9.2.7.0.jar
# jar包中存在的路径
jruby-complete-9.2.7.0.jar\META-INF\jruby.home\lib\ruby\stdlib\ffi\platform\aarch64-linux\types.conf
# 将types.conf复制到相同目录下，名字改为platform.conf

# 注意文件是aarch64-linux平台而不是arm-linux平台
# 虽然CPU是ARM架构的，但是通过uname查看系统是aarch64架构
root@Kylin:/# uname -a
Linux Kylin 4.4.131-20190108.kylin.server.YUN+-generic #kylin SMP Wed Jan 9 16:18:51 CST 2019 aarch64 aarch64 aarch64 GNU/Linux
```

这样 *修改jar包中的types.conf文件* 即可，测试过程中遇到了一些问题：  
一开始以为是替换logstash中的文件，然后搜索了一下系统中的platform.conf文件，其实是修改jar中的文件即可

## Logstash命令

```bash
# 测试
bin/logstash -f conf/alarm-kafka.conf --config.test_and_exit
# 启动
bin/logstash -f conf/file-logstash-kafka-alarm.conf --config.reload.automatic --path.data=/var/T/plug/logstash/data/
```