
- [ ] 构建LNMP环境

- [ ] 获取WordPress包

  

```shell

wget https://cn.wordpress.org/latest-zh_CN.tar.gz

```

  
  
  

- [ ] 解压WordPress包

  

```shell

tar -zxvf wordpress-6.4.2-zh_CN.tar.gz  -C /var/www/wordpress

```

  
  
  

- [ ] 加权

  

```shell

chmod 777 wordpress/ -R

```

  
  
  

- [ ] 更改NGINX的配置文件

  

```.conf

server

        {

        listen 80;

        server_name localhost;

        index index.html index.htm index.php;

        root /var/www/wordpress;

        location ~ \.php$

                        {

                        include fastcgi_params;

                        fastcgi_pass unix:/tmp/php-fcgi.sock;

                        fastcgi_index index.php;

                        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

                        }

        }

```

  
  
  

- [ ] 测试配置文件是否正确

  

```shell

nginx -t

```

  
  
  

- [ ] 重启NGINX服务

  

systemctl restart nginx

  

- [ ] 开放配置文件中指定的端口

  

```shell

  

firewall-cmd --add-port=80/tcp --permanent

```

  
  
  

- [ ] 浏览器访问

  

> localhost/wp-admin/install.php

  

![image-20231210124813004](./WordPress_deploy.assets/image-20231210124813004.png)

  

- [ ] 点击现在就开始

  

![image-20231210124942074](./WordPress_deploy.assets/image-20231210124942074.png)

  

- [ ] 填写信息

  

![image-20231210131037795](./WordPress_deploy.assets/image-20231210131037795.png)

  

- [ ] 创建所填写的数据库

  

```mysql

CREATE DATABASE wordpress;

```

  
  
  

- [ ] next

  

![image-20231210131600482](./WordPress_deploy.assets/image-20231210131600482.png)

  

- [ ] next

  

![image-20231210131701151](./WordPress_deploy.assets/image-20231210131701151.png)

  

- [ ] 搭建完成

  

![image-20231210131747890](./WordPress_deploy.assets/image-20231210131747890.png)