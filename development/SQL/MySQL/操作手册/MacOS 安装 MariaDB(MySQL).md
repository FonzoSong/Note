# MacOS 安装 MariaDB(MySQL)

本教程采用Homebrew工具安装MariaDB，如果没有安装Homebrew工具，请先安装Homebrew。

## 1 关于MariaDB

MySQL隶属当时的SUN公司，SUN被Oracle收购以后，MySQL也随之变成Oracle的产品，开源社区对MySQL进行了分支。由于MySQL品牌属于Oracle公司，开源社区开启了一个新的产品名称MariaDB。MariaDB是MySQL开源精神的继承者。目前Linux系统默认的数据库都是MariaDB。

**简单的说 MariaDB就是 MySQL 数据库！**

在Mac上可以使用Homebrew轻松的安装MariaDB数据库。

## 2 安装MairaDB

在MacOS的终端上执行如下命令就可以安装MariaDB：

```
brew install mariadb
```

如果希望启动MariaDB服务器:

```
brew services start mariadb
```

这个命令也设置每次重新启动电脑， 自动启动MariaDB服务。

如果希望关闭MariaDB服务器，可以使用命令：

```
brew services stop mariadb
```

如果希望检查MariaDB服务的运行状态，可以使用命令：

```
brew services info mariadb
```

## 3 登陆到MySQL

在MariaDB服务启动以后，利用命令可以登录到MySQL

```
mysql
```

或者使用root用户登陆：

```
sudo mysql -u root
```

## 4 设置root用户登陆密码

默认安装后不能直接使用root用户登录到localhost，设置密码后才能登录。设置root用户到密码，首先使用mysql登陆到数据库，然后使用SQL命令设置root用户到密码，这里设置密码为“root”：

```
grant all privileges on *.* to root@localhost identified by 'root';
```

设置以后就可以在Java程序中使用root用户连接到mariadb了。