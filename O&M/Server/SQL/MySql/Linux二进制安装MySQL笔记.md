---
title: "二进制包安装 MySQL 七步走：从解压到改掉初始密码"
slug: linux-binary-install-mysql
summary: "官方预编译包省掉 configure 与 make，七步完成部署：解压、建目录、建用户、写 my.cnf、初始化、启动、改临时密码，并理清 /etc/my.cnf 为什么要删。"
description: "以 MySQL 通用二进制 tar 包为例走通安装全流程，逐步给出可直接复制的命令。重点讲清三处易错点：初始化前必须清掉残留的 /etc/my.cnf、临时密码只能从 mysqld.log 里取、以及默认口令强度校验会拒绝简单密码。附开机自启、远程授权、8.0 与 5.7 语法差异和故障排查表。"
---

> **先搞清概念**：这里说的"二进制安装"指官方 **generic binary tarball**（预编译包，文件名形如 `mysql-5.7.xx-linux-glibc2.12-x86_64.tar.gz`）。它**不需要 `./configure` 和 `make`**，编译工作官方已经做完，你要做的只是解压、配目录、初始化数据、启动、改密码。这也是它比源码编译快得多的原因。

## 〇、开工前的清理与依赖（原文未列，但必做）

```bash
rpm -qa | grep -i mariadb            # CentOS 自带 MariaDB 库，会和 MySQL 冲突
yum remove -y mariadb mariadb-libs   # 删不掉时可用 rpm -e --nodeps mariadb-libs
rpm -qa | grep -i mysql              # 确认没有残留的 yum 版 MySQL

yum install -y libaio                # 二进制包运行依赖，缺了初始化会报错
getenforce                           # SELinux 若为 Enforcing，排障阶段建议先设 Permissive
```

## 一、将 MySQL 解压到 `/usr/local` 目录下

```bash
tar -zxf mysql-5.7.xx-linux-glibc2.12-x86_64.tar.gz -C /usr/local/
cd /usr/local/
mv mysql-5.7.xx-linux-glibc2.12-x86_64 mysql     # 统一改名，后续路径全按 /usr/local/mysql 写
```

`/usr/local` 是发行版"本机自装软件"的约定位置，与 `yum` 装的 `/usr` 分区隔离，卸载时整目录删掉即可，不会误伤系统包。

## 二、在解压的 mysql 目录下创建 `etc`、`data`、`logs` 三个文件夹

```bash
cd /usr/local/mysql
mkdir -p etc data logs
```

三个目录各司其职，这也是面试"你自建的目录分别放什么"的答案：

| 目录     | 放什么                                | 由谁读写              |
| ------ | ---------------------------------- | ----------------- |
| `etc`  | `my.cnf` 配置文件                      | 启动脚本读取            |
| `data` | 数据库文件、表空间、`mysql` 系统库              | `mysqld` 初始化并持续写入 |
| `logs` | `mysqld.log` 错误日志、`mysqld.pid` 进程号 | **初始密码就藏在错误日志里**  |

## 三、创建 mysql 用户，并把所属人与所属组改成 mysql 用户

```bash
groupadd -r mysql
useradd -r -g mysql -s /sbin/nologin mysql     # -r 系统账号，-s nologin 禁止登录
chown -R mysql:mysql /usr/local/mysql
chmod 750 /usr/local/mysql/data                # 数据目录不对外开放
```

为什么不能用 root 跑 MySQL：`mysqld` 以 root 运行会被直接拒绝启动（除非显式加 `--user=root`），设计上就要求降权到专用账号；同时限定属主也防止其他本地用户读取数据文件。

## 四、在 `etc` 目录下创建 `my.cnf`

```ini
# /usr/local/mysql/etc/my.cnf
[mysqld]
datadir=/usr/local/mysql/data
socket=/tmp/mysql.sock
log-error=/usr/local/mysql/logs/mysqld.log
pid-file=/usr/local/mysql/logs/mysqld.pid

# 以下为建议补充项，按需开启
basedir=/usr/local/mysql
port=3306
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci
max_connections=500
default_authentication_plugin=mysql_native_password
```

四个必填参数各管一件事：

| 参数          | 含义        | 配错的后果                                                        |
| ----------- | --------- | ------------------------------------------------------------ |
| `datadir`   | 数据文件存放目录  | 路径不对 → 初始化成功但启动时找不到库                                         |
| `socket`    | 本地套接字文件路径 | 与客户端不一致 → `mysql -uroot -p` 报 `Can't connect through socket` |
| `log-error` | 错误日志      | **取初始密码全靠它**，没配就只能从终端输出找                                     |
| `pid-file`  | 进程号文件     | 路径不可写 → 启动脚本停不掉服务                                            |

> `default_authentication_plugin=mysql_native_password` 用来兼容老版本客户端与驱动；MySQL 8.0 默认认证插件是 `caching_sha2_password`，很多老应用连不上就是栽在这里。

## 五、回到 `bin` 目录初始化数据目录

```bash
cd /usr/local/mysql/bin
./mysqld --initialize --user=mysql \
         --basedir=/usr/local/mysql \
         --datadir=/usr/local/mysql/data
```

**运行后检查系统目录 `/etc` 下是否有 `my.cnf` 文件，有则删除。**

这一步是二进制安装最隐蔽的坑：MySQL 会按顺序读取多个选项文件，`/etc/my.cnf` 优先级很高，往往是被 MariaDB 或旧的 yum 安装遗留下来的，里面的 `datadir`、`socket` 和你第 4 步写的配置不一致。结果就是初始化正常但启动连不上、甚至连到别人的数据目录。运维上建议**先备份再移除**，而不是直接删：

```bash
[ -f /etc/my.cnf ] && cp /etc/my.cnf /etc/my.cnf.bak.$(date +%F) && rm -f /etc/my.cnf
```

另外两个必须知道的细节：

* **临时密码只出现一次**，写在 `log-error` 指定的文件里：

  ```bash
  grep -i 'temporary password' /usr/local/mysql/logs/mysqld.log
  # 2026-09-29T05:12:33.123456s [Note] ... generated temporary password: root@localhost: aB3dEf!4xy5q
  ```

  冒号后那串就是初始密码，注意它可能包含 `!`、`@` 等特殊字符，复制时别漏首位字符。

* `--initialize` 与 `--initialize-insecure` 的区别：前者生成随机临时密码（安全，生产用）；后者 root **空密码**，仅适合本地开发测试。

* `data` 目录**必须为空**才能初始化，重复初始化前要先清空：`rm -rf /usr/local/mysql/data/*`。

## 六、切换到 `support-files`，用启动脚本管理 MySQL

```bash
cd /usr/local/mysql/support-files
./mysql.server start      # 开启
./mysql.server stop       # 关闭
./mysql.server restart    # 重启
./mysql.server status     # 查看运行状态
```

**装完先设开机自启**，否则机器重启服务就没了：

```bash
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
chkconfig --add mysqld && chkconfig mysqld on     # CentOS 6 风格
systemctl enable mysqld                            # CentOS 7+ 兼容 init.d 脚本
service mysqld start                               # 之后就能用服务名管理
```

顺手把客户端命令加入 `PATH`，免得每次都进 `bin` 目录：

```bash
echo 'export PATH=$PATH:/usr/local/mysql/bin' >> /etc/profile
source /etc/profile && mysql -V
```

## 七、从错误日志取初始密码，登录并修改

去 `logs` 文件夹查看 `mysqld.log` 文件里面 mysql 的初始化密码，用该密码登录后进去修改初始密码。

```bash
mysql -uroot -p          # 粘贴第五步 grep 出来的临时密码
```

登录后执行（**未改密码前，服务器处于受限模式，只能执行改密类语句**）：

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root123456';
FLUSH PRIVILEGES;
```

关于这两句的三个考点：

1. `ALTER USER` 本身会立即生效，**`FLUSH PRIVILEGES` 在这里并非必需**；它是给"直接 `UPDATE mysql.user` 改表"这种手工操作准备的——改完系统表必须刷新权限缓存才生效。
2. 默认口令强度校验开启时，`root123456` 这类只有小写字母加数字的密码会被拒绝：

   ```text
   ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
   ```

   两种解法：直接用满足强度的密码（大小写 + 数字 + 符号、长度 ≥ 8，如 `Root@123456`）；或临时放宽策略再改。

   ```sql
   SHOW VARIABLES LIKE 'validate_password%';
   SET GLOBAL validate_password.policy = LOW;      -- 8.0；5.7 为 validate_password_policy
   SET GLOBAL validate_password.length = 4;        -- 8.0；5.7 为 validate_password_length
   ```
3. **5.7 与 8.0 语法差异**：8.0 指定插件写法是 `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Root@123456';`；并且 `GRANT` 不再隐式建用户，远程访问必须先 `CREATE USER` 再 `GRANT`。

**允许远程连接**（面试高频追问）：

```sql
-- 5.7
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'Root@123456' WITH GRANT OPTION;
FLUSH PRIVILEGES;

-- 8.0：建号与授权分离
CREATE USER 'root'@'%' IDENTIFIED BY 'Root@123456';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
FLUSH PRIVILEGES;
```

```bash
firewall-cmd --add-port=3306/tcp --permanent && firewall-cmd --reload
```

> 生产环境不建议开 `root@'%'`，标准做法是建独立账号并按库授权，只放行应用服务器 IP。

## 八、验证与常见故障速查

```bash
ss -lntp | grep 3306                     # 端口是否监听、进程是否为 mysqld
mysqladmin -uroot -p'Root@123456' status # 服务应答与运行变量
mysql -uroot -p -e "select version(), @@datadir, @@socket, @@port;"
ps -ef | grep mysqld | grep -v grep
```

| 现象                                               | 根因                     | 处理                                                |
| ------------------------------------------------ | ---------------------- | ------------------------------------------------- |
| 启动无报错但连不上                                        | `/etc/my.cnf` 残留覆盖配置   | 备份并移除，或启动时加 `--defaults-file` 显式指定                |
| `Can't connect through socket '/tmp/mysql.sock'` | 客户端读的 `socket` 与服务端不一致 | 用 `mysql --socket=/tmp/mysql.sock` 或统一写进 `my.cnf` |
| 改密报 `ERROR 1819`                                 | 口令强度校验                 | 换强密码或调低 `validate_password` 策略                    |
| `data directory has files in it`                 | 重复初始化                  | 清空 `data` 后重新 `--initialize`                      |
| 找不到临时密码                                          | `log-error` 未生效或已滚动    | 检查 `logs/mysqld.log`；必要时清 `data` 重新初始化            |
| 服务起来了但远程连不上                                      | `root@'%'` 未授权或防火墙     | 授权 + 放行 3306，`bind-address` 确认为 `0.0.0.0`         |

## 九、考点速记（背诵卡）

**七步口诀**：**解压 → 建目录 → 建用户 → 写配置 → 初始化 → 启服务 → 改密码**（前置先把 MariaDB 清干净）。

| 高频提问                         | 秒答                                                                  |
| ---------------------------- | ------------------------------------------------------------------- |
| 二进制安装和源码编译的区别？               | 官方已编译好，无 `configure`/`make`，只做解压、配置、初始化                             |
| 为什么必须建 mysql 用户？             | `mysqld` 拒绝以 root 运行，需降权并限定数据目录属主                                   |
| `my.cnf` 四个必配项？              | `datadir`、`socket`、`log-error`、`pid-file`                           |
| 为什么要删 `/etc/my.cnf`？         | 它优先级高且常是旧包残留，`datadir`/`socket` 会冲突                                 |
| 初始密码从哪来？                     | `grep 'temporary password' logs/mysqld.log`，只出现一次                   |
| `--initialize-insecure` 是什么？ | root 空密码初始化，仅用于测试                                                   |
| 初始化前目录要求？                    | `data` 必须为空，否则报错                                                    |
| `FLUSH PRIVILEGES` 何时必须？     | 手工改 `mysql` 库系统表之后；`ALTER USER` 无需                                  |
| 简单密码为什么改不上？                  | `validate_password` 默认策略，报 `ERROR 1819`，需强密码或降策略                    |
| 8.0 老客户端连不上？                 | 默认 `caching_sha2_password`，改 `mysql_native_password`                |
| 怎么设开机自启？                     | `mysql.server` 拷到 `/etc/init.d/mysqld`，`chkconfig/systemctl enable` |
| 怎么确认端口起来了？                   | `ss -lntp \| grep 3306` + `mysqladmin status`                       |
