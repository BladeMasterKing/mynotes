# Redis设置密码有两种方式，通过客户端设置或配置文件设置

## 客户端设置

```bash
root@Kylin:/var/T/plug/redis/bin# redis-cli -p 6379
127.0.0.1:6379> config get requirepass
(error) NOAUTH Authentication required.
# 说明已经设置了密码，需要验证密码
# 设置密码
127.0.0.1:6379> config set requirepass talent
```

## 配置文件设置

```bash
# 修改redis.conf 文件
# requirepass foobared
requirepass talent   指定密码talent
```

## 验证密码登录

```bash
# 先登录后验证
$ redis-cli -p 6379
127.0.0.1:6379> auth talent
OK
# 先验证后登录
$ redis-cli -p 6379 -a talent
```