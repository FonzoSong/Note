# Zabbix安装手册

# 一. 环境准备

|     IP地址      |  主机名   |
| :-------------: | :-------: |
| 192.168.162.128 | prototype |
|  192.168.162.9  |  zabbix   |
| 192.168.162.80  |   nginx   |

## 1. 本地hosts文件映射域名

![image-20231221194244233](./assets/image-20231221194244233.png)

## 2. yum源与epel源配置

```shell
rm -rfv /etc/yum.repos.d/*
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
```

![image-20231221195207126](./assets/image-20231221195207126.png)

# 二.yum部署LNMP环境



**本文仅需安装MySQL!!!**



## 1.NGINX安装及配置

### <1>安装

```shell
yum -y install nginx 
```

### <2>配置

1. 启动服务并加入开机自启动

```shell
systemctl enable  nginx --now
```



2. 开放nginx所需要的80端口

```shell
firewall-cmd --add-port=80/tcp --permanent
firewall-cmd --reload
```

## 2.MySQL安装及配置

### <1>卸载CentOS自带的MySQL

```shell
yum -y remove mari*
rm -rf /var/lib/mysql/*
rpm -qa | grep mariadb
yum remove mysql-community-serve
rpm -qa |grep mysql
yum remove mysql-*
find / -name mysql
rm -rf /var/lib/mysql 
```

### <2>环境清理

1. 查看glibc版本

```shell
ldd --version
```



![image-20231221202911823](./assets/image-20231221202911823.png)

**这里MySQL5.7需要glibc版本大于等于2.12**

2. 拉取MySQL官方yum源

```shell
wget http://repo.mysql.com/mysql57-community-release-el7-8.noarch.rpm
```



3. 安装yum源

```shell
yum install -y mysql57-community-release-el7-8.noarch.rpm
```



4. 禁用GPG验证

```shell
vim /etc/yum.repos.d/mysql-community.repo
```



![image-20231221205224573](./assets/image-20231221205224573.png)

### <3>安装MySQL

```shell
yum instatll -y mysql-server
```



### <4>配置MySQL

1. 启动MySQL服务

```shell
systemctl enable  mysqld --now
```



2. 查看MySQL密码

```shell
grep "password" /var/log/mysqld.log
```



3. 登入MySQL

```shell
mysql -uroot -p
```




4. 修改MySQL密码

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY '000000';
```

**若出现了以下错误则需要更改MySQL密码策略**

![image-20231221205947365](./assets/image-20231221205947365.png)

```mysql
set global validate_password_policy=0;
set global validate_password_length=1;
```



## 3.PHP安装及配置

### <1>环境部署

1. 安装yum源及yum管理工具
```shell
 yum install epel-release
 yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
 yum install yum-utils
```
### <2>安装PHP

```shell
 yum install php74-php-gd  php74-php-pdo php74-php-mbstring php74-php-cli php74-php-fpm php74-php-mysqlnd
```

### <3>配置PHP

1. 启动服务并添加到自启动

```shell
systemctl enable --now php74-php-fpm
```



## 4.配置nginx解析php

1. 编写nginx配置文件

```shell
vim /etc/nginx/conf.d/nginx-php.conf
```

```/etc/nginx/conf.d/nginx-php.conf

server {
 listen       80 default_server;

 server_name  localhost;
 root         /usr/share/nginx/html;

# Load configuration files for the default server block.

 location / {
     index index.php index.html index.htm;
 }

 error_page 404 /404.html;
     location = /40x.html {
 }

 error_page 500 502 503 504 /50x.html;
     location = /50x.html {
 }

 location ~ \.php$ {
     fastcgi_pass   127.0.0.1:9000;
     fastcgi_index  index.php;
     fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
     include        fastcgi_params;
 }
}

2. 编写php测试文件

```shell
vim /usr/share/nginx/html/test-php.php
```

> <?php
> echo "Hello World!";
> ?>

3. 重启nginx

```shell
systemctl restart nginx
```

4. 测试是否成功

```shell
curl 192.168.162.9/test-php.php
```

附成功截图

![image-20231221214048999](./assets/image-20231221214048999.png)

# 三.Zabbix部署

## 1.环境部署

### <1>更新所有包

```shell
yum update -y
```

### <2>创建MySQL数据库和用户

```mysql
create database zabbix character set utf8 collate utf8_bin;
create user zabbix@localhost identified by 'zabbix';
grant all privileges on zabbix.* to zabbix@localhost;
flush privileges;
quit;
```

如果报了以下的错误

![image-20231222170154264](./assets/image-20231222170154264.png)

请看[更改MySQL密码策略](#<4>配置MySQL)中的第4步

### <3>安装zabbix官方yum源

```shell
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm 
```

### <4>创建zabbix用户

```shell
useradd -M -s /sbin/nologin zabbix
```

### <5>同步时间

```shell
yum install ntpdate -y
ntpdate -u ntp.aliyun.com
```



## 2.安装zabbix

### <1>**安装存储库并启用软件合集**

```shell
yum install zabbix-server-mysql zabbix-agent -y
yum install centos-release-scl -y
```

### <2>更改yum源文件启用前端源

```shell
vim /etc/yum.repos.d/zabbix.repo
```

![image-20231222175537769](./assets/image-20231222175537769.png)

### <3>安装前端包

```shell
yum install zabbix-web-mysql-scl zabbix-nginx-conf-scl -y
```

### <4>导入数据和初始架构

```shell
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

### <5>禁用禁用log_bin_trust_function_creators选项。

```mysql
set global log_bin_trust_function_creators = 0;
```

### <6>更改zabbix_server配置文件

```shell
vim /etc/zabbix/zabbix_server.conf
```

下列参数为注释状态记得删除**#**解除注释

```/etc/zabbix/zabbix_server.conf
DBPassword=zabbix
```

### <7>更改PHP配置文件

```shell
vim /etc/opt/rh/rh-nginx116/nginx/conf.d/zabbix.conf
```

下列参数为注释状态记得删除**#**解除注释

```/etc/opt/rh/rh-nginx116/nginx/conf.d/zabbix.conf
listen 80;
server_name 192.168.162.9;
```

附上正确图片:

![image-20231222183201953](./assets/image-20231222183201953.png)

```shell
vim /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
```

```/etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
listen.acl_users = apache,nginx
php_value[date.timezone] = Asia/Shanghai
```

附上正确图片:

![image-20231222183116915](./assets/image-20231222183116915.png)

### <8>启动zabbix_server和agent进程

```shell
systemctl enable zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm --now
```

## 3.zabbix的Web页面配置

### <1>浏览器访问192.168.162.9

![image-20231222183811537](./assets/image-20231222183811537.png)![image-20231222183844334](./assets/image-20231222183844334.png)

获取mysql所监听的端口

```shell
netstat -tunlp |grep mysql
```

![image-20231222184044184](./assets/image-20231222184044184.png)

![image-20231222184345476](./assets/image-20231222184345476.png)

![image-20231222184656175](./assets/image-20231222184656175.png)

![image-20231222184740474](./assets/image-20231222184740474.png)

## 4.zabbix客户端部署

### <1>yum源配置

```shell
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm 
```

### <2>安装与配置

1. 时间同步

```shell
yum install ntpdate -y
ntpdate -u ntp.aliyun.com
```

2. 安装zabbix-agent2

```shell
yum install zabbix-agent2 -y
```

3. 编辑配置文件

```shell
vim /etc/zabbix/zabbix_agent2.conf
```

```/etc/zabbix/zabbix_agent2.conf
Server=192.168.162.9 			#修改为zabbix_server的ip
ServerActive=192.168.162.9	 	#修改为zabbix的ip
Hostname=nginx 					#修改为当前主机的名称
```

4. 启动服务并加入启动项

```shell
systemctl enable zabbix-agent2 --now
```

## 5.web页面添加主机

![image-20231224114909367](./assets/image-20231224114909367.png)

![image-20231224115323002](./assets/image-20231224115323002.png)



## 6.nginx服务监控（自定义监控项）

### <1>安装nginx

```shell
yum install nginx -y
```



### <2>安装zabbix-agent2

```shell
yum install -y zabbix-agent2
```



### <3>编辑nginx配置文件

```shell
vim /etc/nginx/conf.d/nginx-status.conf
```

```/etc/nginx/conf.d/nginx-status.conf
server {
        listen 80;
        server_name localhost;
        location /nginx-status {
                stub_status     on;
                access_log      on;
                }
}
```



### <4>编辑zabbix-agent2配置文件

```shell
vim /etc/zabbix/zabbix_agent2.conf
```

```/etc/zabbix/zabbix_agent2.conf
Server=192.168.162.9 			#修改为zabbix_server的ip
ServerActive=192.168.162.9	 	#修改为zabbix的ip
Hostname=nginx 					#修改为当前主机的名称
```



### <5>启动nginx和zabbix-agent2服务

```shell
systemctl enable zabbix-agent2 nginx --now
```

### <6>创建监控脚本

```shell
mkdir /etc/zabbix/scripts/
vim /etc/zabbix/scripts/nginx-status
```

```shell
#!/bin/bash
 
function active {
 
/usr/bin/curl "http://192.168.162.80/nginx-status" 2>/dev/null| grep 'Act ive' | awk '{print $NF}'
 
}
 
function reading {  /usr/bin/curl "http://192.168.162.80/nginx-status" 2>/dev/null| grep 'Rea ding' | awk '{print $2}'
 
}
 
function writing {  /usr/bin/curl "http://192.168.162.80/nginx-status" 2>/dev/null| grep 'Wri ting' | awk '{print $4}'
 
}
 
function waiting {  /usr/bin/curl "http://192.168.162.80/nginx-status" 2>/dev/null| grep 'Wai ting' | awk '{print $6}'
 
}
 
function accepts {  /usr/bin/curl "http://192.168.162.80/nginx-status" 2>/dev/null| awk NR==3 | awk '{print $1}'
 
}
 
function handled {  /usr/bin/curl "http://192.168.162.80/nginx-status" 2>/dev/null| awk NR==3 | awk '{print $2}'
 
}
 
function requests {  /usr/bin/curl "http://192.168.162.80/nginx-status" 2>/dev/null| awk NR==3 | awk '{print $3}'
 
}
 
$1
```

### <7>添加到zabbix监控关键字

```/etc/zabbix/zabbix_agent2.d/nginx_zabbix.conf
vim /etc/zabbix/zabbix_agent2.d/nginx-zabbix.conf
```

```shell
UserParameter=nginx[*],/etc/zabbix/scripts/nginx-status $1
```

```shell
chmod 755 /etc/zabbix/scripts/nginx-status
systemctl restart zabbix-agent2
```

## <8>创建监控模板

![image-20231224125601683](./assets/image-20231224125601683.png)

![image-20231224125656790](./assets/image-20231224125656790.png)

![image-20231224125730920](./assets/image-20231224125730920.png)

![image-20231224125756058](./assets/image-20231224125756058.png)

![image-20231224132202885](./assets/image-20231224132202885.png)

![image-20231224132230380](./assets/image-20231224132230380.png)

![image-20231224132312852](./assets/image-20231224132312852.png)

![image-20231224132337779](./assets/image-20231224132337779.png)

![image-20231224132557082](./assets/image-20231224132557082.png)

## 7.邮件报警
```
yum -y install sendmail mailx
```

```
vi /etc/mail.rc
```

```
set from=
set smtp=smtp-mail.outlook.com:587
set nss-config-dir=/etc/pki/nssdb/
set ssl-verify=ignore
set smtp-auth-user=
set smtp-auth-password=
set smtp-auth=login
set smtp-use-starttls=yes
```

```
systemctl restart sendmail
```
![img](./assets/image-20231218210425579.png)

![image-20231224132632363](./assets/image-20231224132632363.png)

![image-20231218211148373.png](./assets/1702906385919-aa8044db-432d-4daf-b06a-7181e9aa7499.png)

![image-20231218211339799.png](./assets/1702906385946-a4c6fe9d-3825-4855-be65-1bb95fc1c395.png)

![image-20231218211405515.png](./assets/1702906385945-1624120c-afd5-4fac-a615-314e03a31976.png)

![image-20231218211432751.png](./assets/1702906385947-f2c8585c-653b-4df3-861c-12876b7528b2.png)

![image-20231218210425579.png](./assets/1702906419058-e0bfa782-d490-41e2-9a82-9c7a7f7ae5a5.png)

## 8.MySQL主从与keepalive监控

从

```/etc/zabbix/zabbix_agent2.d/script/check_keepalived.sh
#!/bin/bash

if [ `ip a show ens32 |grep 192.168.162.38|wc -l` -ne 0 ]   
then
    echo "1"   
else
    echo "0"   
fi
```

zabbix配置文件

```
UserParameter=check_keepalived,/etc/zabbix/zabbix_agent2.d/script/check_keepalived.sh
```



从

```
   #!/bin/bash
   #Desc：用于获取主从同步信息，判断主从是否出现异常，然后提交给zabbix
   #Date: 2019-06-06
   #by：Lee-YJ
   
     USER="root"
     PASSWD="000000"
     NAME=$1
   
     function IO {
         Slave_IO_Running=`mysql -u $USER -p$PASSWD -e "show slave status\G;" 2> /dev/null |grep Slave_IO_Running |awk '{print $2}'`
         if [ $Slave_IO_Running == "Yes" ];then
             echo 0 
         else
             echo 1 
         fi
     }
   
     function SQL {
         Slave_SQL_Running=`mysql -u $USER -p$PASSWD -e "show slave status\G;" 2> /dev/null |grep Slave_SQL_Running: |awk '{print $2}'`
         if [ $Slave_SQL_Running == "Yes" ];then
             echo 0 
         else
             echo 1 
         fi
   
     }
   
     case $NAME in
        io)
            IO
        ;;
        sql)
            SQL
        ;;
        *)
             echo -e "Usage: $0 [io | sql]"
     esac
```

监控配置

```
UserParameter=mysql.slave[*],/etc/zabbix/script/mysql_slvae_status.sh $1
```







# 附录

## zabbix官网安装教程

[下载Zabbix 5.0 LTS for CentOS 7, MySQL, Nginx](https://www.zabbix.com/cn/download?zabbix=5.0&os_distribution=centos&os_version=7&components=server_frontend_agent&db=mysql&ws=nginx)

**选项**![image-20231222182027759](./assets/image-20231222182027759.png)

![image-20231222193323015](./assets/image-20231222193323015.png)



## NGINX操作命令

```shell
service nginx start		|		systemctl start nginx
service nginx stop		|		systemctl stop nginx
service nginx status	|		systemctl status nginx
systemctl enable nginx	#取消自启动就用disable
```

## MySQL操作命令

```shell
service mysqld start
service mysqld stop
service mysqld restart
service mysqld status
systemctl enable mysqld   #取消自启动就用disable
```

## PHP操作命令

```shell
service php74-php-fpm start
service php74-php-fpm stop
service php74-php-fpm restart
service php74-php-fpm status
systemctl enable php74-php-fpm #取消自启动就用disable
```

