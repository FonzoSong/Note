
### 环境

  

| IP             | hostname     |

| -------------- | ------------ |

| 192.168.162.80 | nginx-master |

| 192.168.162.81 | nginx-bak    |

  
  
  

## 1.配置yum源

  

```shell

rm -rfv /etc/yum.repos.d/*

curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

wget -O /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo

```

  

## 2.安装keepalived

```shell

yum install -y keepalived

```

  
  

## 3.编写配置文件

  

```shell

vim /etc/keepalived/keepalived.conf

```

  

### master

  

```/etc/keepalived/keepalived.conf

global_defs {

   notification_email {

     acassen@firewall.loc

   }

   notification_email_from Alexandre.Cassen@firewall.loc

   smtp_server 127.0.0.1

   smtp_connect_timeout 30

   router_id ngix-master

}

  
  

vrrp_instance VI_1 {

    state MASTER

    interface ens32

    virtual_router_id 51

    priority 100

    advert_int 1

    authentication {

        auth_type PASS

        auth_pass 1111

    }

  

    virtual_ipaddress {

        192.168.163.80

    }

    preempt delay 60

}

```

  

### backup

  

```/etc/keepalived/keepalived.conf

global_defs {

   notification_email {

     acassen@firewall.loc

   }

   notification_email_from Alexandre.Cassen@firewall.loc

   smtp_server 127.0.0.1

   smtp_connect_timeout 30

   router_id nginx-bak

}

  
  

vrrp_instance VI_1 {

    state BACKUP

    interface ens32

    virtual_router_id 51

    priority 90

    advert_int 1

    authentication {

        auth_type PASS

        auth_pass 1111

    }

  

    virtual_ipaddress {

        192.168.163.80

    }

    preempt delay 60

}

```

  

## 4.启动keepalived服务

  

```shell

systemctl restart keepalived

```

  

# keepalived实现nginx高可用（主备）

  

## 1.安装nginx

  

```shell

yum install nginx

```

  
  
  

## 2.编辑nginx欢迎页面

  

master

  

```shell

echo master  > /usr/share/nginx/html/index.html

```

  

backup

  

```shell

echo backup  > /usr/share/nginx/html/index.html

```

  
  
  

## 3.启动nginx服务

  

```shell

systemctl enable nginx --now

```

  
  
  

## 4.再起一台虚拟机然后拉VIP

  

```shell

curl 192.168.162.88

```

  

![image-20231224201955785](./assets/image-20231224201955785.png)

  

![image-20231224202012591](./assets/image-20231224202012591.png)

  
  
  

![image-20231224202850184](./assets/image-20231224202850184.png)

  

## 5.编辑脚本

  

```shell

vim /usr/local/sbin/check_ng.sh

```

  

```/usr/local/sbin/check_ng.sh

#!/bin/bash

d=`date --date today +%Y%m%d_%H:%M:%S`

#d=`date -d today +%Y%m%d%H%M%S`

n=`ps -C nginx --no-heading|wc -l`

if [ $n -eq 0 ]; then

        systemctl restart nginx ;echo "nginx restart" >>/var/log/ng.log

        n2=`ps -C nginx --no-heading|wc -l`

        if [ $n2 -eq 0 ]; then

                echo "$d nginx down,keepalived will stop" >>/var/log/check_ng.log

                systemctl stop keepalived

        fi

fi

```

  

## 6.将脚本添加到keepalived

```shell

vim /etc/keepalived/keepalived.conf

```

  

```/etc/keepalived/keepalived.conf

vrrp_script chk_nginx {

    script "/usr/local/sbin/check_ng.sh"

    interval 2

    weight 20

    fall 2

    rise 2

}

track_script {chk_nginx}

```

## 7.停止nginx服务以验证高可用

  

![image-20231225094652006](./assets/image-20231225094652006.png)

  

![image-20231225094657086](./assets/image-20231225094657086.png)

  

查看日志可发现nginx重启成功

  

# keepalived实现mysql高可用

  

编写脚本

  

```shell

vim /etc/keepalived/check_mysql.sh"

```

  
  
  

```/etc/keepalived/check_mysql.sh

#!/bin/bash

counter=$(netstat -na|grep "LISTEN"|grep "3306"|wc -l)

if [ "${counter}" -eq 0 ]; then

    /etc/init.d/keepalived stop

fi

```

  

更改配置文件

  

```/etc/keepalived/keepalived.conf

vim /etc/keepalived/keepalived.conf

```

  
  
  

```shell

vrrp_script chk_mysql {

    script "/etc/keepalived/check_mysql.sh"

    interval 2

    weight -20

}

track_script {chk_mysql}

```