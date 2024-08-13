# MySQL主从同步

# MySQL同步

## master：

### 1.修改my.cnf

```my.cof
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

log_bin
server_id=1

symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

### 2.重启MySQL服务

```shell
systemctl restart mysqld
```

### 3.创建同步用户

```shell
create user 'sync'@'%' identified by '111111';
grant replication slave,replication client on *.* to 'sync'@'%';
```

### 4.查看master二进制文件

```mysql
show master status \G
```



## backup:

### 1.修改my.cnf

```my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

server_id=2

symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

### 2.重启mysql服务

```shell
systemctl restart mysqld
```

### 3.测试可链接master

```shell
mysql -h'192.162.162.30' -u'sync' -p'111111'
```

### 4.设置主从关系

```mysql
change master to master_host='192.168.162.30',master_user='sync',master_password='111111',master_log_file='mysql-master-bin.000001',master_log_pos=615;
```

### 5.启动从服务器

```MySQL
start slave;
```

### 6.检查同步状态

```mysql
show slave status\G
```

## 自动指定二进制文件配置

在master和backup的my.cnf中加入

```shell
gtid_mode=ON
enforce_gtid_consistency=1
```

设置主从关系

```mysql
change master to master_host='192.168.162.30',master_user='sync',master_password='111111',master_auto_position=1;
```

