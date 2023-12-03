# RocketMQ

## 安装运行

### 1.解压安装

```bash
$ unzip rocketmq-all-5.1.4-source-release.zip
$ cd rocketmq-all-5.1.4-source-release/
$ mvn -Prelease-all -DskipTest -Dspotbugs.skip=true clean install -U
$ cd distribution/target/rocketmq-5.1.4/rocketmq-5.1.4
```

### 2.启动NameServer

```bash
## 启动namesrv
$ nohup sh bin/mqnamesrv &

## 验证namesrv是否成功启动

$ tail -f ~/logs/rocketmqlogs/namesrc.log
2023-11-17 09:28:16 INFO main - The Name Server boot success. serializeType=JSON
```

### 3.启动Broker+Proxy

NameServer成功启动后，启动Broker和Proxy

```bash
## 先启动broker
$ nohup sh bin/mqbroker -n localhost:9876 --enable-proxy &

## 验证broker启动成功，
$ tail -f ~/logs/rocketmqlogs/proxy.log
```

### 4.消息收发

```bash
$ export NAMESRV_ADDR=localhost:9876
$ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
 SendResult [sendStatus=SEND_OK, msgId= ...
 
$ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
 ConsumeMessageThread_%d Receive New Messages: [MessageExt...
```

### 5.关闭

```bash
# 关闭broker
$ sh bin/mqshutdown broker

# 关闭nameserver
$ sh bin/mqshutdown namesrv
```

## 消息

消息的属性包括：

Topic：消息的主题

Body：消息的存储内容

Tags：标签，每个消息只有一个；方便服务器过滤

