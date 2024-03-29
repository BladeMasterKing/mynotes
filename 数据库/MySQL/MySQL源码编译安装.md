# MySQL ARM架构Linux操作系统的编译安装

== 编译(略)

## 配置
网上的配置教程都是通过使用my.cnf启动，公司的启动脚本并未使用配置文件，而是通过在启动脚本中设置环境变量来启动

```shell
# 脚本中mysql服务的启动命令
$ ./mysqld --no-defaults --log-error="/var/mysql/mysql.log" --basedir="/var/mysql" --user=topsa --datadir="/var/mysql/data" --pid_file="/var/mysql/mysql.pid" --socket="/var/mysql/mysql.sock" --port="3306" --character-set-server=utf8 --lower_case_table_names=1 --max_connections=1000 --interactive-timeout=2592000 --wait_timeout=2592000 --bind-address=0.0.0.0 --skip-grant-tables &

# 查看mysql服务是否启动
root@Kylin:/var/T/db/mysql/bin# ps -ef|grep mysql
root    60298     1  0 10月12 ?      00:01:37 ./mysqld --no-defaults --log-error="/var/mysql/mysql.log" --basedir="/var/mysql" --user=topsa --datadir="/var/mysql/data" --pid_file="/var/mysql/mysql.pid" --socket="/var/mysql/mysql.sock" --port="3306" --character-set-server=utf8 --lower_case_table_names=1 --max_connections=1000 --interactive-timeout=2592000 --wait_timeout=2592000 --bind-address=0.0.0.0
```

## 修改root密码
在mysqld启动命令加上**skip-grant-tables**可以跳过密码验证:
```shell
# mysql/bin目录
$ ./mysqld --no-defaults --log-error="/var/mysql/mysql.log" --basedir="/var/mysql" --user=topsa --datadir="/var/mysql/data" --pid_file="/var/mysql/mysql.pid" --socket="/var/mysql/mysql.sock" --port="3306" --character-set-server=utf8 --lower_case_table_names=1 --max_connections=1000 --interactive-timeout=2592000 --wait_timeout=2592000 --bind-address=0.0.0.0 --skip-grant-tables
```

### 登录mysql

```shell
$ mysql -uroot
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/develop/mysql-new/mysql.sock' (2)
```

找不到mysql.sock文件的错误，mysqld.socket文件是用来给客户端和服务端进行通信的，如果通过源码方式安装，默认情况下这个文件会被放在tmp目录下，这个文件会mysqld启动时的参数创建，在启动脚本里指定的socket连接的存放目录或在/etc/my.cnf文件指定。不通过/etc/my.cnf启动mysqld的话，客户端连接时需要指定sock文件，即

```shell
$ ./mysql -uroot --socket="../mysql.sock"
```

修改root用户密码：

```sql
# 切换数据库
> use mysql;
# 修改root用户的密码
> update mysql.user set authentication_string=password('新密码') where user='root';
# 刷新权限
> flush privileges;
```

退出mysql客户端连接，修改启动脚本去掉--skip-grant-tables参数，重新启动mysqld服务即可。
