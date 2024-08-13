# 索引

|           命令            |                   解释                   | 快捷键 |                           详解链接                           |          备注           |
| :-----------------------: | :--------------------------------------: | ------ | :----------------------------------------------------------: | :---------------------: |
|      echo [content]       |                 打印输出                 |        |                                                              |                         |
|           clear           |                   清屏                   | ctrl+L |                                                              |                         |
|       man [command]       |                   帮助                   |        |                                                              |                         |
|         cd [path]         |              切换到指定路径              |        |                                                              |                         |
|         hostname          |                打印主机名                |        |                                                              |                         |
|           date            |              打印时间和日期              |  | [](#date) |                         |
|            ls             |             列出当前目录内容             |        |                           [](#ls)                            |                         |
|            cal            |                 打印日历                 |        |                                                              |                         |
|            bc             |                  计算器                  |        |                                                              | 使用scale来定义小数位数 |
|          reboot           |                   重启                   |        |                                                              |       仅限于root        |
|        shutdown -r        |                   重启                   |        |                                                              |       仅限于root        |
|          init 6           |                   重启                   |        |                                                              |       仅限于root        |
|          logout           |                   关机                   |        |                                                              |                         |
|           halt            |                   关机                   |        |                                                              |                         |
|        shutdown -h        |                   关机                   |        |                                                              |       仅限于root        |
|          init 0           |                   关机                   |        |                                                              |       仅限于root        |
|           uname           |       打印当前操作系统和机器的信息       |        |                                                              |                         |
|            pwd            |               打印当前路径               |        |                                                              |                         |
|          history          |               打印历史命令               |        |                                                              |                         |
|            cat            |               打印文件内容               |        |                           [](#cat)                           |                         |
| tac | 反着打印文件内容 | |  | |
| more | 滚屏打印文件内容 | |  | 空格翻页,ctrl+D向上翻页 |
| less | 同more | |  | 同more,/向下搜索,?向上搜索 |
| head | 显示文件前十行 | |  | -n [打印的行数] |
| tall | 显示文件的后十行 | |  | -f 动态,实时查看 |
|          vi\vim           |             使用vi\vim编辑器             |        |                         [](#vi\vim)                          |                         |
|            yum            |             前端软件包管理器             |        |                           [](#yum)                           |                         |
|          netstat          |               显示网络状态               |        |                         [](#netstat)                         |                         |
|         systemctl         |            systemd的管理命令             |        | [网页链接](https://blog.csdn.net/skh2015java/article/details/94012643) |                         |
|           touch           |                 新建文件                 |        |                          [](#touch)                          |                         |
|            rm             |                   删除                   |        |                           [](#rm)                            |                         |
|            mv             |                移动,改名                 |        |                           [](#mv)                            |                         |
|           stat            | 查看[inode](..\磁盘\inode和block.md)信息 |        |  |                         |
|           mkdir           |                 创建目录                 |        | [](#mkdir) |                         |
|           rmdir           |                删除空目录                |        | [](#rmdir) |                         |
|          useradd          |                 添加用户                 |        | [](#useradd) |                         |
|          userdel          |                 删除用户                 |        | [](#userdel) |                         |
|          usermod          |               修改用户属性               |        | [](#usermod) |                         |
|          passwd           |                 设置密码                 |        | [](#passwd) |                         |
|           chmod           |             修改文件访问权限             |        | [](#chmod) |                         |
|            ln             |                   链接                   |        | [](#ln) |                         |
|          getfacl          |               查看ACL权限                |        | [](#getfacl) |                         |
|          setfacl          |               设置ACL权限                |        | [](#setfacl) |                         |
|          lsattr           |                 隐藏权限                 |        | [](#lsattr) |                         |
|           fdisk           |                 分区创建                 |        | [](#fdisk) |                         |
|           mkfs            |                格式化磁盘                |        | [](#mkfs) |                         |
| mk[swap](..\磁盘\swap.md) |             格式化磁盘为swap             |        | [](#mkswap) |                         |
|           blkid           |                 查看uuid                 |        | [](#blkid) |                         |
|           mount           |                   挂载                   |        | [](#mount) |                         |
|          swapon           |               挂载swap分区               |        | [](#swapon) |                         |
|          umount           | 卸载 |        | [](#umount) |                         |
|            df             | 产看磁盘大小 |        | [](#df) |                         |
|            du             | 查看文件大小 ||[](#du)||
| systemctl | sysd操作命令 ||[](..\系统服务\sysd.md\#systemctl)||
| groupadd | 添加组 ||[](#groupadd)||
| groupdel | 删除组 ||||
| groupmod | 修改组 ||[](#groupmod)||
| tar | 压缩解压缩 ||[](#tar)||
| ps | 静态任务管理器 ||[](#ps)||
| top | 动态任务管理器 ||[](#top)||
| kill | 杀死单个进程 ||[](#kill)||
| killall | 杀死一个进程树 ||[](#killall)||
| uptime | 系统运行时间 ||||
| jobs | 查看当前终端放入后台的工作 ||||
| fg | 将后台命令恢复前台执行 ||||
| bg | 恢复后台被挂起的命令 ||||
| grep | 查找文件里符合条件的字符串或正则表达式 ||[](#grep)||
| egrep | 文件中查找指定字符串 ||[](#egrep)||
| lass | 输出 ||||
| file | 查看文件类型及编码类型 ||||
| find | 查找文件和目录 ||[](#find)||
| tree | 树状显示目录 ||||
| alias | 显示别名 ||||
| netstat | 网络端口 ||[](#netstat)||
| paste | 合并文件的列 ||[](#paste)||
| chown | 更改文件的所属 ||[](#chown)||
| umask | 查看文件默认权限 |||可以在/etc/bashrc中修改,一般为0022前面的那个'0'表示umask为8进制,意为u=7-0,g=7-2,o=7-2|
| chattr | 更改特殊属性 ||[](#chattr)||
| whereis | 查找文件 ||[](#whereis)||
| locate | 查找文件 |||yum install -y mlocate|
| sudo | 以管理员的身份执行指令 ||[](#sudo)||
| blkid | 获取各个fen ||||
| fiewall-cmd | 防火墙操作 ||[](#firewall-cmd)||











## ls 

​	英文全拼： list directory contents

​	命令用于显示指定工作目录下之内容（列出目前工作目录所含的文件及子目录)。

### 语法

```
 ls [-alrtAFR] [name...]
```

**参数** :

- -a 显示所有文件及目录 (**.** 开头的隐藏文件也会列出)
- -d 只列出目录（不递归列出目录内的文件）。
- -l 以长格式显示文件和目录信息，包括权限、所有者、大小、创建时间等。
- -r 倒序显示文件和目录。
- -t 将按照修改时间排序，最新的文件在最前面。
- -A 同 -a ，但不列出 "." (目前目录) 及 ".." (父目录)
- -F 在列出的文件名称后加一符号；例如可执行档则加 "*", 目录则加 "/"
- -R 递归显示目录中的所有文件和子目录。



[返回](#索引)





## cat	

英文全拼：concatenate

命令用于连接文件并打印到标准输出设备上。

### 使用权限

所有使用者

### 语法格式

```
cat [-AbeEnstTuv] [--help] [--version] fileName
```

### 参数说明：

**-n 或 --number**：由 1 开始对所有输出的行数编号。

**-b 或 --number-nonblank**：和 -n 相似，只不过对于空白行不编号。

**-s 或 --squeeze-blank**：当遇到有连续两行以上的空白行，就代换为一行的空白行。

**-v 或 --show-nonprinting**：使用 ^ 和 M- 符号，除了 LFD 和 TAB 之外。

**-E 或 --show-ends** : 在每行结束处显示 $。

**-T 或 --show-tabs**: 将 TAB 字符显示为 ^I。

**-A, --show-all**：等价于 -vET。

**-e：**等价于"-vE"选项；

**-t：**等价于"-vT"选项；

​	

[返回](#索引)



​	

## yum

yum（ Yellow dog Updater, Modified）是一个在 Fedora 和 RedHat 以及 SUSE 中的 Shell 前端软件包管理器。

基于 RPM 包管理，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。

yum 提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。

### yum 语法

```
yum [options] [command] [package ...]
```

- **options：**可选，选项包括-h（帮助），-y（当安装过程提示选择全部为 "yes"），-q（不显示安装的过程）等等。
- **command：**要进行的操作。
- **package：**安装的包名。

------

## yum常用命令

- 1. 列出所有可更新的软件清单命令：**yum check-update**

- 2. 更新所有软件命令：**yum update**

- 3. 仅安装指定的软件命令：**yum install <package_name>**

- 4. 仅更新指定的软件命令：**yum update <package_name>**

- 5. 列出所有可安裝的软件清单命令：**yum list**

- 6. 删除软件包命令：**yum remove <package_name>**

- 7. 查找软件包命令：**yum search <keyword>**

- 8. 清除缓存命令:

  - **yum clean packages**: 清除缓存目录下的软件包
  - **yum clean headers**: 清除缓存目录下的 headers
  - **yum clean oldheaders**: 清除缓存目录下旧的 headers
  - **yum clean, yum clean all (= yum clean packages; yum clean oldheaders)** :清除缓存目录下的软件包及旧的 headers



[返回](#索引)





## netstat

Linux netstat 命令用于显示网络状态。

利用 netstat 指令可让你得知整个 Linux 系统的网络情况。

### 语法

```
netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]
```

**参数说明**：

- -a或--all 显示所有连线中的Socket。
- -A<网络类型>或--<网络类型> 列出该网络类型连线中的相关地址。
- -c或--continuous 持续列出网络状态。
- -C或--cache 显示路由器配置的快取信息。
- -e或--extend 显示网络其他相关信息。
- -F或--fib 显示路由缓存。
- -g或--groups 显示多重广播功能群组组员名单。
- -h或--help 在线帮助。
- -i或--interfaces 显示网络界面信息表单。
- -l或--listening 显示监控中的服务器的Socket。
- -M或--masquerade 显示伪装的网络连线。
- -n或--numeric 直接使用IP地址，而不通过域名服务器。
- -N或--netlink或--symbolic 显示网络硬件外围设备的符号连接名称。
- -o或--timers 显示计时器。
- -p或--programs 显示正在使用Socket的程序识别码和程序名称。
- -r或--route 显示Routing Table。
- -s或--statistics 显示网络工作信息统计表。
- -t或--tcp 显示TCP传输协议的连线状况。
- -u或--udp 显示UDP传输协议的连线状况。
- -v或--verbose 显示指令执行过程。
- -V或--version 显示版本信息。
- -w或--raw 显示RAW传输协议的连线状况。
- -x或--unix 此参数的效果和指定"-A unix"参数相同。
- --ip或--inet 此参数的效果和指定"-A inet"参数相同。

[返回](#索引)



## touch

Linux touch命令用于修改文件或者目录的时间属性，包括存取时间和更改时间。若文件不存在，系统会建立一个新的文件。

ls -l 可以显示档案的时间记录。

### 语法

```
touch [-acfm][-d<日期时间>][-r<参考文件或目录>] [-t<日期时间>][--help][--version][文件或目录…]
```

- **参数说明**：
- a 改变档案的读取时间记录。
- m 改变档案的修改时间记录。
- c 假如目的档案不存在，不会建立新的档案。与 --no-create 的效果一样。
- f 不使用，是为了与其他 unix 系统的相容性而保留。
- r 使用参考档的时间记录，与 --file 的效果一样。
- d 设定时间与日期，可以使用各种不同的格式。
- t 设定档案的时间记录，格式与 date 指令相同。
- --no-create 不会建立新档案。
- --help 列出指令格式。
- --version 列出版本讯息。

[返回](#索引)



## rm

Linux rm（英文全拼：remove）命令用于删除一个文件或者目录。

### 语法

```
rm [options] name...
```

**参数**：

- -i 删除前逐一询问确认。
- -f 即使原档案属性设为唯读，亦直接删除，无需逐一确认。
- -r 将目录及以下之档案亦逐一删除。

[返回](#索引)















## mv

Linux mv（英文全拼：move file）命令用来为文件或目录改名、或将文件或目录移入其它位置。

### 语法

```
mv [options] source dest
mv [options] source... directory
```

**参数说明**：

- **-b**: 当目标文件或目录存在时，在执行覆盖前，会为其创建一个备份。
- **-i**: 如果指定移动的源目录或文件与目标的目录或文件同名，则会先询问是否覆盖旧文件，输入 y 表示直接覆盖，输入 n 表示取消该操作。
- **-f**: 如果指定移动的源目录或文件与目标的目录或文件同名，不会询问，直接覆盖旧文件。
- **-n**: 不要覆盖任何已存在的文件或目录。
- **-u**：当源文件比目标文件新或者目标文件不存在时，才执行移动操作。



[返回](#索引)



## mkdir

Linux mkdir（英文全拼：make directory）命令用于创建目录。

### 语法

```
mkdir [-p] dirName
```

**参数说明**：

- -p 确保目录名称存在，不存在的就建一个。

[返回](#索引)



## rmdir

Linux rmdir（英文全拼：remove directory）命令删除空的目录。

### 语法

```
rmdir [-p] dirName
```

**参数**：

- -p 是当子目录被删除后使它也成为空目录的话，则顺便一并删除。





## useradd

Linux useradd 命令用于建立用户帐号。

useradd 可用来建立用户帐号。帐号建好之后，再用 passwd 设定帐号的密码。而可用 userdel 删除帐号。使用 useradd 指令所建立的帐号，实际上是保存在 /etc/passwd 文本文件中。

### 语法

```
useradd [-mMnr][-c <备注>][-d <登入目录>][-e <有效期限>][-f <缓冲天数>][-g <群组>][-G <群组>][-s <shell>][-u <uid>][用户帐号]
```

或

```
useradd -D [-b][-e <有效期限>][-f <缓冲天数>][-g <群组>][-G <群组>][-s <shell>]
```

**参数说明**： 

- -c<备注> 　加上备注文字。备注文字会保存在passwd的备注栏位中。
- -d<登入目录> 　指定用户登入时的起始目录。
- -D 　变更预设值．
- -e<有效期限> 　指定帐号的有效期限。
- -f<缓冲天数> 　指定在密码过期后多少天即关闭该帐号。
- -g<群组> 　指定用户所属的群组。
- -G<群组> 　指定用户所属的附加群组。
- -m 　制定用户的登入目录。
- -M 　不要自动建立用户的登入目录。
- -n 　取消建立以用户名称为名的群组．
- -r 　建立系统帐号。
- -s<shell>　 　指定用户登入后所使用的shell。
- -u<uid> 　指定用户ID。





## userdel

Linux userdel命令用于删除用户帐号。

userdel可删除用户帐号与相关的文件。若不加参数，则仅删除用户帐号，而不删除相关文件。

### 语法

```
userdel [-r][用户帐号]
```

**参数说明**：

- -r 　删除用户登入目录以及目录中所有文件。





## usermod

Linux usermod命令用于修改用户帐号。

usermod可用来修改用户帐号的各项设定。

### 语法

```
usermod [-LU][-c <备注>][-d <登入目录>][-e <有效期限>][-f <缓冲天数>][-g <群组>][-G <群组>][-l <帐号名称>][-s <shell>][-u <uid>][用户帐号]
```

**参数说明**：

- -c<备注> 　修改用户帐号的备注文字。
- -d登入目录> 　修改用户登入时的目录。
- -e<有效期限> 　修改帐号的有效期限。
- -f<缓冲天数> 　修改在密码过期后多少天即关闭该帐号。
- -g<群组> 　修改用户所属的群组。
- -G<群组> 　修改用户所属的附加群组。
- -l<帐号名称> 　修改用户帐号名称。
- -L 　锁定用户密码，使密码无效。
- -s<shell> 　修改用户登入后所使用的shell。
- -u<uid> 　修改用户ID。
- -U 　解除密码锁定。





## passwd

Linux passwd命令用来更改使用者的密码

### 语法

```
passwd [-k] [-l] [-u [-f]] [-d] [-S] [username]
```

**必要参数**：

- -d 删除密码
- -f 强迫用户下次登录时必须修改口令
- -w 口令要到期提前警告的天数
- -k 更新只能发送在过期之后
- -l 停止账号使用
- -S 显示密码信息
- -u 启用已被停止的账户
- -x 指定口令最长存活期
- -g 修改群组密码
- 指定口令最短存活期
- -i 口令过期后多少天停用账户

**选择参数**：

- --help 显示帮助信息
- --version 显示版本信息





## chmod

**使用权限** : 所有使用者

### 语法

```
chmod [-cfvR] [--help] [--version] mode file...
```

### 参数说明

mode : 权限设定字串，格式如下 :

```
[ugoa...][[+-=][rwxX]...][,...]
```

其中：

- u 表示该文件的拥有者，g 表示与该文件的拥有者属于同一个群体(group)者，o 表示其他以外的人，a 表示这三者皆是。
- \+ 表示增加权限、- 表示取消权限、= 表示唯一设定权限。
- r 表示可读取，w 表示可写入，x 表示可执行，X 表示只有当该文件是个子目录或者该文件已经被设定过为可执行。

其他参数说明：

- -c : 若该文件权限确实已经更改，才显示其更改动作
- -f : 若该文件权限无法被更改也不要显示错误讯息
- -v : 显示权限变更的详细资料
- -R : 对目前目录下的所有文件与子目录进行相同的权限变更(即以递归的方式逐个变更)
- --help : 显示辅助说明
- --version : 显示版本







### ln

Linux ln（英文全拼：link files）命令是一个非常重要命令，它的功能是为某一个文件在另外一个位置建立一个同步的链接。

当我们需要在不同的目录，用到相同的文件时，我们不需要在每一个需要的目录下都放一个必须相同的文件，我们只要在某个固定的目录，放上该文件，然后在 其它的目录下用ln命令链接（link）它就可以，不必重复的占用磁盘空间。

### 语法

```
 ln [参数][源文件或目录][目标文件或目录]
```

其中参数的格式为

```
[-bdfinsvF] [-S backup-suffix] [-V {numbered,existing,simple}]
[--help] [--version] [--]
```

**命令功能** :
Linux文件系统中，有所谓的链接(link)，我们可以将其视为档案的别名，而链接又可分为两种 : 硬链接(hard link)与软链接(symbolic link)，硬链接的意思是一个档案可以有多个名称，而软链接的方式则是产生一个特殊的档案，该档案的内容是指向另一个档案的位置。硬链接是存在同一个文件系统中，而软链接却可以跨越不同的文件系统。

不论是硬链接或软链接都不会将原本的档案复制一份，只会占用非常少量的磁碟空间。

**软链接**：

- 1.软链接，以路径的形式存在。类似于Windows操作系统中的快捷方式
- 2.软链接可以 跨文件系统 ，硬链接不可以
- 3.软链接可以对一个不存在的文件名进行链接
- 4.软链接可以对目录进行链接

**硬链接**：

- 1.硬链接，以文件副本的形式存在。但不占用实际空间。
- 2.不允许给目录创建硬链接
- 3.硬链接只有在同一个文件系统中才能创建

#### 命令参数

**必要参数**：

- --backup[=CONTROL] 备份已存在的目标文件
- -b 类似 **--backup** ，但不接受参数
- -d 允许超级用户制作目录的硬链接
- -f 强制执行
- -i 交互模式，文件存在则提示用户是否覆盖
- -n 把符号链接视为一般目录
- -s 软链接(符号链接)
- -v 显示详细的处理过程

**选择参数**：

- -S "-S<字尾备份字符串> "或 "--suffix=<字尾备份字符串>"
- -V "-V<备份方式>"或"--version-control=<备份方式>"
- --help 显示帮助信息
- --version 显示版本信息







## getfacl

getfacl命令来自英文词组“get file access control list”的缩写，其功能是用于显示文件或目录的ACL策略。 

语法格式：getfacl [参数] 文件或目录名 常用参数：

- -a 显示文件的ACL策略 

- -c 不显示注释标题 

- d 显示目录的ACL策略 

- -e 显示所有的有效权限 

- -h 显示帮助信息 

- -L 找到符号链接对应的文件

- -n 显示用户UID和组群GID

- -P 不找符号链接对应的文件 

- -R 递归处理所有子文件 

- -t 设置表格输出格式 

- -v 显示版本信息





## setfacl

1、setfacl的用途
setfacl命令可以用来细分linux下的文件权限。
chmod命令可以把文件权限分为u,g,o三个组，而setfacl可以对每一个文件或目录设置更精确的文件权限。
换句话说，setfacl可以更精确的控制权限的分配。
比如：让某一个用户对某一个文件具有某种权限。

这种独立于传统的u,g,o的rwx权限之外的具体权限设置叫ACL（Access Control List）
ACL可以针对单一用户、单一文件或目录来进行r,w,x的权限控制，对于需要特殊权限的使用状况有一定帮助。
如，某一个文件，不让单一的某个用户访问。

### 2、setfacl的用法
```
用法: setfacl [-bkndRLP] { -m|-M|-x|-X ... } file ...
```
- -m,       --modify-acl 更改文件的访问控制列表
-  -M,       --modify-file=file 从文件读取访问控制列表条目更改
-  -x,       --remove=acl 根据文件中访问控制列表移除条目
-  -X,       --remove-file=file 从文件读取访问控制列表条目并删除
-  -b,       --remove-all 删除所有扩展访问控制列表条目
-  -k,       --remove-default 移除默认访问控制列表
            --set=acl 设定替换当前的文件访问控制列表
            --set-file=file 从文件中读取访问控制列表条目设定
            --mask 重新计算有效权限掩码
-  -n,       --no-mask 不重新计算有效权限掩码
-  -d,       --default 应用到默认访问控制列表的操作
-  -R,       --recursive 递归操作子目录
-  -L,       --logical 依照系统逻辑，跟随符号链接
-  -P,       --physical 依照自然逻辑，不跟随符号链接
            --restore=file 恢复访问控制列表，和“getfacl -R”作用相反
            --test 测试模式，并不真正修改访问控制列表属性
 - -v,       --version           显示版本并退出
 - -h,       --help              显示本帮助信息







## lsattr

Linux lsattr命令用于显示文件属性。

用chattr执行改变文件或目录的属性，可执行lsattr指令查询其属性。

### 语法

```
lsattr [-adlRvV][文件或目录...]
```

**参数**：

- -a 　显示所有文件和目录，包括以"."为名称开头字符的额外内建，现行目录"."与上层目录".."。
- -d 　显示，目录名称，而非其内容。
- -l 　此参数目前没有任何作用。
- -R 　递归处理，将指定目录下的所有文件及子目录一并处理。
- -v 　显示文件或目录版本。
- -V 　显示版本信息。





## fdisk

Linux fdisk 是一个创建和维护分区表的程序，它兼容 DOS 类型的分区表、BSD 或者 SUN 类型的磁盘列表。

### 语法

```
fdisk [必要参数][选择参数]
```

**必要参数：**

- -l 列出素所有分区表
- -u 与 **-l** 搭配使用，显示分区数目

**选择参数：**

- -s<分区编号> 指定分区
- -v 版本信息

**菜单操作说明**



- m ：显示菜单和帮助信息
- a ：活动分区标记/引导分区
- d ：删除分区
- l ：显示分区类型
- n ：新建分区
- p ：显示分区信息
- q ：退出不保存
- t ：设置分区号
- v ：进行分区检查
- w ：保存修改
- x ：扩展应用，高级功能







## mkfs

Linux mkfs（英文全拼：make file system）命令用于在特定的分区上建立 linux 文件系统。

**使用方式** :

```
mkfs [-V] [-t fstype] [fs-options] filesys [blocks]
```

**参数** ：

- device ： 预备检查的硬盘分区，例如：/dev/sda1
- -V : 详细显示模式
- -t : 给定档案系统的型式，Linux 的预设值为 ext2
- -c : 在制做档案系统前，检查该partition 是否有坏轨
- -l bad_blocks_file : 将有坏轨的block资料加到 bad_blocks_file 里面
- block : 给定 block 的大小







## mkswap

Linux mkswap命令用于设置交换区(swap area)。

mkswap可将磁盘分区或文件设为Linux的交换区。

### 语法

```
mkswap [-cf][-v0][-v1][设备名称或文件][交换区大小]
```

**参数**：

- -c 建立交换区前，先检查是否有损坏的区块。
- -f 在SPARC电脑上建立交换区时，要加上此参数。
- -v0 建立旧式交换区，此为预设值。
- -v1 建立新式交换区。
- [交换区大小] 指定交换区的大小，单位为1024字节。









## blkid

在Linux下可以使用 blkid命令 对查询设备上所采用文件系统类型进行查询。blkid主要用来对系统的块设备（包括交换分区）所使用的文件系统类型、LABEL、[UUID](https://so.csdn.net/so/search?q=UUID&spm=1001.2101.3001.7020)等信息进行查询。要使用这个命令必须安装e2fsprogs软件包。

#### 语法

```bash
blkid -L | -U



blkid [-c ] [-ghlLv] [-o] [-s ][-t ] -[w ] [ ...]



blkid -p [-s ] [-O ] [-S ][-o] ...



blkid -i [-s ] [-o] ...
```

#### 选项

```bash
-c <file>   指定cache文件(default: /etc/blkid.tab, /dev/null = none)



-d          don't encode non-printing characters



-h          显示帮助信息



-g          garbage collect the blkid cache



-o <format> 指定输出格式



-k          list all known filesystems/RAIDs and exit



-s <tag>    显示指定信息，默认显示所有信息



-t <token>  find device with a specific token (NAME=value pair)



-l          look up only first device with token specified by -t



-L <label>  convert LABEL to device name



-U <uuid>   convert UUID to device name



-v          显示版本信息



-w <file>   write cache to different file (/dev/null = no write)



<dev>       specify device(s) to probe (default: all devices)



Low-level probing options:



-p          low-level superblocks probing (bypass cache)



-i          gather information about I/O limits



-S <size>   overwrite device size



-O <offset> probe at the given offset



-u <list>   filter by "usage" (e.g. -u filesystem,raid)



-n <list>   filter by filesystem type (e.g. -n vfat,ext3)
```











## mount

Linux mount命令是经常会使用到的命令，它用于挂载Linux系统外的文件。

### 语法

```
mount [-hV]
mount -a [-fFnrsvw] [-t vfstype]
mount [-fnrsvw] [-o options [,...]] device | dir
mount [-fnrsvw] [-t vfstype] [-o options] device dir
```

**参数说明：**

- -V：显示程序版本
- -h：显示辅助讯息
- -v：显示较讯息，通常和 -f 用来除错。
- -a：将 /etc/fstab 中定义的所有档案系统挂上。
- -F：这个命令通常和 -a 一起使用，它会为每一个 mount 的动作产生一个行程负责执行。在系统需要挂上大量 NFS 档案系统时可以加快挂上的动作。
- -f：通常用在除错的用途。它会使 mount 并不执行实际挂上的动作，而是模拟整个挂上的过程。通常会和 -v 一起使用。
- -n：一般而言，mount 在挂上后会在 /etc/mtab 中写入一笔资料。但在系统中没有可写入档案系统存在的情况下可以用这个选项取消这个动作。
- -s-r：等于 -o ro
- -w：等于 -o rw
- -L：将含有特定标签的硬盘分割挂上。
- -U：将档案分割序号为 的档案系统挂下。-L 和 -U 必须在/proc/partition 这种档案存在时才有意义。
- -t：指定档案系统的型态，通常不必指定。mount 会自动选择正确的型态。
- -o async：打开非同步模式，所有的档案读写动作都会用非同步模式执行。
- -o sync：在同步模式下执行。
- -o atime、-o noatime：当 atime 打开时，系统会在每次读取档案时更新档案的『上一次调用时间』。当我们使用 flash 档案系统时可能会选项把这个选项关闭以减少写入的次数。
- -o auto、-o noauto：打开/关闭自动挂上模式。
- -o defaults:使用预设的选项 rw, suid, dev, exec, auto, nouser, and async.
- -o dev、-o nodev-o exec、-o noexec允许执行档被执行。
- -o suid、-o nosuid：
- 允许执行档在 root 权限下执行。
- -o user、-o nouser：使用者可以执行 mount/umount 的动作。
- -o remount：将一个已经挂下的档案系统重新用不同的方式挂上。例如原先是唯读的系统，现在用可读写的模式重新挂上。
- -o ro：用唯读模式挂上。
- -o rw：用可读写模式挂上。
- -o loop=：使用 loop 模式用来将一个档案当成硬盘分割挂上系统。











## swapon

Linux swapon命令用于激活Linux系统中交换空间，Linux系统的内存管理必须使用交换区来建立虚拟内存。

### 语法

```
/sbin/swapon -a [-v]
/sbin/swapon [-v] [-p priority] specialfile ...
/sbin/swapon [-s]
```

**参数说明：**

- -h 请帮帮我
- -V 显示版本讯息
- -s 显示简短的装置讯息
- -a 自动启动所有SWAP装置
- -p 设定优先权，你可以在0到32767中间选一个数字给他。或是在 /etc/fstab 里面加上 pri=[value] ([value]就是0~32767中间一个数字)，然后你就可以很方便的直接使用 swapon -a 来启动他们，而且有优先权设定。

swapon 是开启swap.

相对的,便有一个关闭swap的指令,swapoff.









## umount

Linux umount（英文全拼：unmount）命令用于卸除文件系统。

umount可卸除目前挂在Linux目录中的文件系统。

### 语法

```
umount [-ahnrvV][-t <文件系统类型>][文件系统]
```

**参数**：

- -a 卸除/etc/mtab中记录的所有文件系统。
- -h 显示帮助。
- -n 卸除时不要将信息存入/etc/mtab文件中。
- -r 若无法成功卸除，则尝试以只读的方式重新挂入文件系统。
- -t<文件系统类型> 仅卸除选项中所指定的文件系统。
- -v 执行时显示详细的信息。
- -V 显示版本信息。
- [文件系统] 除了直接指定文件系统外，也可以用设备名称或挂入点来表示文件系统。







## df

Linux df（英文全拼：disk free） 命令用于显示目前在 Linux 系统上的文件系统磁盘使用情况统计。

### 语法

```
df [选项]... [FILE]...
```

- 文件-a, --all 包含所有的具有 0 Blocks 的文件系统
- 文件--block-size={SIZE} 使用 {SIZE} 大小的 Blocks
- 文件-h, --human-readable 使用人类可读的格式(预设值是不加这个选项的...)
- 文件-H, --si 很像 -h, 但是用 1000 为单位而不是用 1024
- 文件-i, --inodes 列出 inode 资讯，不列出已使用 block
- 文件-k, --kilobytes 就像是 --block-size=1024
- 文件-l, --local 限制列出的文件结构
- 文件-m, --megabytes 就像 --block-size=1048576
- 文件--no-sync 取得资讯前不 sync (预设值)
- 文件-P, --portability 使用 POSIX 输出格式
- 文件--sync 在取得资讯前 sync
- 文件-t, --type=TYPE 限制列出文件系统的 TYPE
- 文件-T, --print-type 显示文件系统的形式
- 文件-x, --exclude-type=TYPE 限制列出文件系统不要显示 TYPE
- 文件-v (忽略)
- 文件--help 显示这个帮手并且离开
- 文件--version 输出版本资讯并且离开









## du

Linux du （英文全拼：disk usage）命令用于显示目录或文件的大小。

du 会显示指定的目录或文件所占用的磁盘空间。

### 语法

```
du [-abcDhHklmsSx][-L <符号连接>][-X <文件>][--block-size][--exclude=<目录或文件>][--max-depth=<目录层数>][--help][--version][目录或文件]
```

**参数说明**：

- -a或-all 显示目录中个别文件的大小。
- -b或-bytes 显示目录或文件大小时，以byte为单位。
- -c或--total 除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。
- -D或--dereference-args 显示指定符号连接的源文件大小。
- -h或--human-readable 以K，M，G为单位，提高信息的可读性。
- -H或--si 与-h参数相同，但是K，M，G是以1000为换算单位。
- -k或--kilobytes 以1024 bytes为单位。
- -l或--count-links 重复计算硬件连接的文件。
- -L<符号连接>或--dereference<符号连接> 显示选项中所指定符号连接的源文件大小。
- -m或--megabytes 以1MB为单位。
- -s或--summarize 仅显示指定目录或文件的总大小，而不显示其子目录的大小。
- -S或--separate-dirs 显示个别目录的大小时，并不含其子目录的大小。
- -x或--one-file-xystem 以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。
- -X<文件>或--exclude-from=<文件> 在<文件>指定目录或文件。
- --exclude=<目录或文件> 略过指定的目录或文件。
- --max-depth=<目录层数> 超过指定层数的目录后，予以忽略。
- --help 显示帮助。
- --version 显示版本信息。





## groupadd

groupadd 命令用于创建一个新的工作组，新工作组的信息将被添加到系统文件中。

相关文件:

- /etc/group 组账户信息。
- /etc/gshadow 安全组账户信息。
- /etc/login.defs Shadow密码套件配置。

### 语法

groupadd 命令 语法格式如下：

```
groupadd [-g gid [-o]] [-r] [-f] group
```

**参数说明：**

- -g：指定新建工作组的 **id**；
- -r：创建系统工作组，系统工作组的组 ID 小于 500；
- -K：覆盖配置文件 **/etc/login.defs**；
- -o：允许添加组 ID 号不唯一的工作组。
- -f,--force: 如果指定的组已经存在，此选项将失明了仅以成功状态退出。当与 -g 一起使用，并且指定的 GID_MIN 已经存在时，选择另一个唯一的 GID（即 -g 关闭）。





## groupmod

Linux groupmod命令用于更改群组识别码或名称。

需要更改群组的识别码或名称时，可用groupmod指令来完成这项工作。

### 语法

```
groupmod [-g <群组识别码> <-o>][-n <新群组名称>][群组名称]
```

**参数**：

- -g <群组识别码> 　设置欲使用的群组识别码。
- -o 　重复使用群组识别码。
- -n <新群组名称> 　设置欲使用的群组名称





## tar

Linux tar（英文全拼：tape archive ）命令用于备份文件。

tar 是用来建立，还原备份文件的工具程序，它可以加入，解开备份文件内的文件。

### 语法

```
tar [-ABcdgGhiklmMoOpPrRsStuUvwWxzZ][-b <区块数目>][-C <目的目录>][-f <备份文件>][-F <Script文件>][-K <文件>][-L <媒体容量>][-N <日期时间>][-T <范本文件>][-V <卷册名称>][-X <范本文件>][-<设备编号><存储密度>][--after-date=<日期时间>][--atime-preserve][--backuup=<备份方式>][--checkpoint][--concatenate][--confirmation][--delete][--exclude=<范本样式>][--force-local][--group=<群组名称>][--help][--ignore-failed-read][--new-volume-script=<Script文件>][--newer-mtime][--no-recursion][--null][--numeric-owner][--owner=<用户名称>][--posix][--erve][--preserve-order][--preserve-permissions][--record-size=<区块数目>][--recursive-unlink][--remove-files][--rsh-command=<执行指令>][--same-owner][--suffix=<备份字尾字符串>][--totals][--use-compress-program=<执行指令>][--version][--volno-file=<编号文件>][文件或目录...]
```

**参数**：

- -A或--catenate 新增文件到已存在的备份文件。
- -b<区块数目>或--blocking-factor=<区块数目> 设置每笔记录的区块数目，每个区块大小为12Bytes。
- -B或--read-full-records 读取数据时重设区块大小。
- -c或--create 建立新的备份文件。
- -C<目的目录>或--directory=<目的目录> 切换到指定的目录。
- -d或--diff或--compare 对比备份文件内和文件系统上的文件的差异。
- -f<备份文件>或--file=<备份文件> 指定备份文件。
- -F<Script文件>或--info-script=<Script文件> 每次更换磁带时，就执行指定的Script文件。
- -g或--listed-incremental 处理GNU格式的大量备份。
- -G或--incremental 处理旧的GNU格式的大量备份。
- -h或--dereference 不建立符号连接，直接复制该连接所指向的原始文件。
- -i或--ignore-zeros 忽略备份文件中的0 Byte区块，也就是EOF。
- -k或--keep-old-files 解开备份文件时，不覆盖已有的文件。
- -K<文件>或--starting-file=<文件> 从指定的文件开始还原。
- -l或--one-file-system 复制的文件或目录存放的文件系统，必须与tar指令执行时所处的文件系统相同，否则不予复制。
- -L<媒体容量>或-tape-length=<媒体容量> 设置存放每体的容量，单位以1024 Bytes计算。
- -m或--modification-time 还原文件时，不变更文件的更改时间。
- -M或--multi-volume 在建立，还原备份文件或列出其中的内容时，采用多卷册模式。
- -N<日期格式>或--newer=<日期时间> 只将较指定日期更新的文件保存到备份文件里。
- -o或--old-archive或--portability 将资料写入备份文件时使用V7格式。
- -O或--stdout 把从备份文件里还原的文件输出到标准输出设备。
- -p或--same-permissions 用原来的文件权限还原文件。
- -P或--absolute-names 文件名使用绝对名称，不移除文件名称前的"/"号。
- -r或--append 新增文件到已存在的备份文件的结尾部分。
- -R或--block-number 列出每个信息在备份文件中的区块编号。
- -s或--same-order 还原文件的顺序和备份文件内的存放顺序相同。
- -S或--sparse 倘若一个文件内含大量的连续0字节，则将此文件存成稀疏文件。
- -t或--list 列出备份文件的内容。
- -T<范本文件>或--files-from=<范本文件> 指定范本文件，其内含有一个或多个范本样式，让tar解开或建立符合设置条件的文件。
- -u或--update 仅置换较备份文件内的文件更新的文件。
- -U或--unlink-first 解开压缩文件还原文件之前，先解除文件的连接。
- -v或--verbose 显示指令执行过程。
- -V<卷册名称>或--label=<卷册名称> 建立使用指定的卷册名称的备份文件。
- -w或--interactive 遭遇问题时先询问用户。
- -W或--verify 写入备份文件后，确认文件正确无误。
- -x或--extract或--get 从备份文件中还原文件。
- -X<范本文件>或--exclude-from=<范本文件> 指定范本文件，其内含有一个或多个范本样式，让ar排除符合设置条件的文件。
- -z或--gzip或--ungzip 通过gzip指令处理备份文件。
- -Z或--compress或--uncompress 通过compress指令处理备份文件。
- -<设备编号><存储密度> 设置备份用的外围设备编号及存放数据的密度。
- --after-date=<日期时间> 此参数的效果和指定"-N"参数相同。
- --atime-preserve 不变更文件的存取时间。
- --backup=<备份方式>或--backup 移除文件前先进行备份。
- --checkpoint 读取备份文件时列出目录名称。
- --concatenate 此参数的效果和指定"-A"参数相同。
- --confirmation 此参数的效果和指定"-w"参数相同。
- --delete 从备份文件中删除指定的文件。
- --exclude=<范本样式> 排除符合范本样式的文件。
- --group=<群组名称> 把加入设备文件中的文件的所属群组设成指定的群组。
- --help 在线帮助。
- --ignore-failed-read 忽略数据读取错误，不中断程序的执行。
- --new-volume-script=<Script文件> 此参数的效果和指定"-F"参数相同。
- --newer-mtime 只保存更改过的文件。
- --no-recursion 不做递归处理，也就是指定目录下的所有文件及子目录不予处理。
- --null 从null设备读取文件名称。
- --numeric-owner 以用户识别码及群组识别码取代用户名称和群组名称。
- --owner=<用户名称> 把加入备份文件中的文件的拥有者设成指定的用户。
- --posix 将数据写入备份文件时使用POSIX格式。
- --preserve 此参数的效果和指定"-ps"参数相同。
- --preserve-order 此参数的效果和指定"-A"参数相同。
- --preserve-permissions 此参数的效果和指定"-p"参数相同。
- --record-size=<区块数目> 此参数的效果和指定"-b"参数相同。
- --recursive-unlink 解开压缩文件还原目录之前，先解除整个目录下所有文件的连接。
- --remove-files 文件加入备份文件后，就将其删除。
- --rsh-command=<执行指令> 设置要在远端主机上执行的指令，以取代rsh指令。
- --same-owner 尝试以相同的文件拥有者还原文件。
- --suffix=<备份字尾字符串> 移除文件前先行备份。
- --totals 备份文件建立后，列出文件大小。
- --use-compress-program=<执行指令> 通过指定的指令处理备份文件。
- --version 显示版本信息。
- --volno-file=<编号文件> 使用指定文件内的编号取代预设的卷册编号。

### 常用

压缩文件 非打包

```
# touch a.c       
# tar -czvf test.tar.gz    a.c   //压缩 a.c文件为test.tar.gz
a.c
```

列出压缩文件内容

```
# tar -tzvf test.tar.gz 
-rw-r--r-- root/root     0 2010-05-24 16:51:59 a.c
```

解压文件

```
# tar -xzvf test.tar.gz 
a.c
```





## ps

静态任务管理器

```
ps - report a snapshot of the current processes
语法
ps [options)]
常用的参数 :
	a: 显示跟当前终端关联的所有进程
	u:基于用户的格式显示
	U: 显示某用户ID所有的进程
	x: 显示所有进程，不以终端机来区分
	A: 显示所有程序
	e:此参数的效果和指定”A”参数相同
	f: 用ASCII字符显示树状结构，表达程序间的相互关系
	o:自定义输出格式
```

输出:

```
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND

root          2  0.0  0.0      0     0 ?        S    11:57   0:00 [kthreadd]
```

详解:

```
USER : 进程拥有者
PID : pid
%CPU : 占用的CPU使用率
%MEM : 占用的内存使用率
VSZ : 占用的虚拟内存大小
RSS : 占用的内存大小
TTY : 终端号码
STAT : 该进程的状态
START : 进程开始时间
TIME : 执行时间
COMMAND : 所执行的指令
```

```
进程状态:
D;		不可中断的禁止
R;		正在执行中
```



## top

动态任务管理器

```
top [选项]
命令选项
	-d number	number代表秒数，表示top命令显示的页面更新一次的间隔 (default=5s)
	-b	以批次的方式执行top
	-n	与-b配合使用，表示需要进行几次top命令的输出结果
	-p	指定特定的pid进程号进行观察
```

输出

```
top - 18:15:55 up  6:18,  2 users,  load average: 0.00, 0.01, 0.05
Tasks: 110 total,   1 running, 109 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.2 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  2027896 total,  1435364 free,   240620 used,   351912 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.  1619768 avail Mem

PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND

  1 root      20   0  193672   6812   4184 S   0.3  0.3   0:08.93 systemd
  
 14 root      20   0       0      0      0 S   0.3  0.0   0:00.34 ksoftirqd/1
```



## kill

杀死单个进程

```
kill - terminate a process

语法
kill [选项] [进程号]

选项
	-l 打印信号编号,如果不加信号的编号参数,则使用'-l'参数会列出全部的信号名称
	-a 当处理当前进程时,不限制命令名和进程号的对应关系
	-p 只打印相关进程的进程号,而不发送任何信号
	-s 指定发送信号
	-u 指定用户,kill某个用户的所有进程
```

kill命令工作原理:

```
向linux内核发送一个系统操作信号和某个程序的的进程标识号,然后系统内核就可以对进程标识号指定的进程进行操作
```

[信号编号:](https://blog.csdn.net/qq_23080741/article/details/119205621)

```
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
 
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM

16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP

21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ

26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR

31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3

38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8

43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13

48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12

53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7

58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2

63) SIGRTMAX-1  64) SIGRTMAX

```

懒得翻译啦!!!





## killall

杀死一个进程树

```
Linux系统中的killall命令用于杀死指定名字的进程( kill processes by name)
命令格式:
killal [选项] [进程名]
命令选项:
	-l			忽略小写
	-g 			杀死进程组而不是进程
	-i 			交互模式，杀死进程前先询问用户
	-1 			列出所有的已知信号名称
	-q 			不输出警告信息
	-s 			发送指定的信号
	-v 			报告信号是否成功发送
	-w 			等待进程死亡
	--help		显示帮助信息
	-version 	显示版本显示
	-u:			杀死指定用户的进程
	-r:			使用正规表达式匹配要杀死的进程名称
	-p:			杀死进程所属的进程组
```





## egrep

Linux egrep命令用于在文件内查找指定的字符串。

egrep执行效果与"grep-E"相似，使用的语法及参数可参照grep指令，与grep的不同点在于解读字符串的方法。

egrep是用extended regular expression语法来解读的，而grep则用basic regular expression 语法解读，extended regular expression比basic regular expression的表达更规范。

### 语法

```
egrep [范本模式] [文件或目录] 
```

**参数说明：**

- [范本模式] ：查找的字符串规则。
- [文件或目录] ：查找的目标文件或目录。





## find

Linux find 命令用于在指定目录下查找文件和目录。

它可以使用不同的选项来过滤和限制查找的结果。

### 语法

```
find [path] [expression]
```

**参数说明** :

**path** 是要查找的目录路径，可以是一个目录或文件名，也可以是多个路径，多个路径之间用空格分隔，如果未指定路径，则默认为当前目录。

**expression** 是可选参数，用于指定查找的条件，可以是文件名、文件类型、文件大小等等。

expression 中可使用的选项有二三十个之多，以下列出最常用的部份：

- `-name pattern`：按文件名查找，支持使用通配符 `*` 和 `?`。
- `-type type`：按文件类型查找，可以是 `f`（普通文件）、`d`（目录）、`l`（符号链接）等。
- `-size [+-]size[cwbkMG]`：按文件大小查找，支持使用 `+` 或 `-` 表示大于或小于指定大小，单位可以是 `c`（字节）、`w`（字数）、`b`（块数）、`k`（KB）、`M`（MB）或 `G`（GB）。
- `-mtime days`：按修改时间查找，支持使用 `+` 或 `-` 表示在指定天数前或后，days 是一个整数表示天数。
- `-user username`：按文件所有者查找。
- `-group groupname`：按文件所属组查找。

find 命令中用于时间的参数如下：

- `-amin n`：查找在 n 分钟内被访问过的文件。
- `-atime n`：查找在 n*24 小时内被访问过的文件。
- `-cmin n`：查找在 n 分钟内状态发生变化的文件（例如权限）。
- `-ctime n`：查找在 n*24 小时内状态发生变化的文件（例如权限）。
- `-mmin n`：查找在 n 分钟内被修改过的文件。
- `-mtime n`：查找在 n*24 小时内被修改过的文件。

在这些参数中，n 可以是一个正数、负数或零。正数表示在指定的时间内修改或访问过的文件，负数表示在指定的时间之前修改或访问过的文件，零表示在当前时间点上修改或访问过的文件。

例如：**-mtime 0** 表示查找今天修改过的文件，**-mtime -7** 表示查找一周以前修改过的文件。

关于时间 n 参数的说明：

- **+n**：查找比 n 天前更早的文件或目录。
- **-n**：查找在 n 天内更改过属性的文件或目录。
- **n**：查找在 n 天前（指定那一天）更改过属性的文件或目录。









## date

Linux **date** 命令可以用来显示或设定系统的日期与时间。

### 语法

```
date [OPTION]... [+FORMAT]
date [-u] [-d datestr] [-s datestr] [--utc] [--universal] [--date=datestr] [--set=datestr] [--help] [--version] [+FORMAT] [MMDDhhmm[[CC]YY][.ss]]
```

#### 可选参数

- **-d, --date=STRING**：通过字符串显示时间格式，字符串不能是'now'。
- **-f, --file=DATEFILE**：类似于--date; 一次从DATEFILE处理一行。
- **-I[FMT], --iso-8601[=FMT]**：按照 ISO 8601 格式输出时间，FMT 可以为'date'(默认)，'hours'，'minutes'，'seconds'，'ns'。 可用于设置日期和时间的精度，例如：2006-08-14T02:34:56-0600。
- **-R, --rfc-2822** ： 按照 RFC 5322 格式输出时间和日期，例如: Mon, 14 Aug 2006 02:34:56 -0600。
- **--rfc-3339=FMT**：按照 RFC 3339 格式输出，FMT 可以为'date', 'seconds','ns'中的一个，可用于设置日期和时间的精度， 例如：2006-08-14 02:34:56-06:00。
- **-r, --reference=FILE**：显示文件的上次修改时间。
- **-s, --set=STRING**：根据字符串设置系统时间。
- **-u, --utc, --universal**：显示或设置协调世界时(UTC)。
- **--help**：显示帮助信息。
- **--version**：输出版本信息。

#### FORMAT 参数

在显示方面，使用者可以设定欲显示的格式 ，格式设定为一个加号后接数个标记，其中可用的标记列表如下：

```
%%   输出字符 %
%a   星期几的缩写 (Sun..Sat)
%A   星期的完整名称(Sunday..Saturday)。 
%b   缩写的月份名称（例如，Jan）
%B   完整的月份名称（例如，January）
%c   本地日期和时间（例如，Thu Mar  3 23:05:25 2005）
%C   世纪，和%Y类似，但是省略后两位（例如，20）
%d   日 (01..31)
%D   日期，等价于%m/%d/%y
%e   一月中的一天，格式使用空格填充，等价于%_d
%F   完整的日期；等价于 %Y-%m-%d
%g   ISO 标准计数周的年份的最后两位数字
%G   ISO 标准计数周的年份，通常只对%V有用
%h   等价于 %b
%H   小时 (00..23)
%I   小时 (01..12)
%j   一年中的第几天 (001..366)
%k   小时，使用空格填充 ( 0..23); 等价于 %_H
%l   小时, 使用空格填充 ( 1..12); 等价于 %_I
%m   月份 (01..12)
%M   分钟 (00..59)
%n   新的一行，换行符
%N   纳秒 (000000000..999999999)
%p   用于表示当地的AM或PM，如果未知则为空白
%P   类似 %p, 但是是小写的
%r   本地的 12 小时制时间(例如 11:11:04 PM)
%R   24 小时制 的小时与分钟; 等价于 %H:%M
%s   自 1970-01-01 00:00:00 UTC 到现在的秒数
%S   秒 (00..60)
%t   插入水平制表符 tab
%T   时间; 等价于 %H:%M:%S
%u   一周中的一天 (1..7); 1 表示星期一
%U   一年中的第几周，周日作为一周的起始 (00..53)
%V   ISO 标准计数周，该方法将周一作为一周的起始 (01..53)
%w   一周中的一天（0..6），0代表星期天
%W   一年中的第几周，周一作为一周的起始（00..53）
%x   本地的日期格式（例如，12/31/99）
%X   本地的日期格式（例如，23:13:48）
%y   年份后两位数字 (00..99)
%Y   年
%z   +hhmm 格式的数值化时区格式（例如，-0400）
%:z  +hh:mm 格式的数值化时区格式（例如，-04:00）
%::z  +hh:mm:ss格式的数值化时区格式（例如，-04:00:00）
%:::z  数值化时区格式，相比上一个格式增加':'以显示必要的精度（例如，-04，+05:30）
%Z  时区缩写 （如 EDT）
```

若是不以加号作为开头，则表示要设定时间，而时间格式为 **MMDDhhmm[[CC]YY][.ss]**，其中 **MM** 为月份，**DD** 为日，**hh** 为小时，**mm** 为分钟，**CC** 为年份前两位数字，**YY** 为年份后两位数字，**ss** 为秒数。

使用权限：所有使用者。

当您不希望出现无意义的 0 时(比如说 1999/03/07)，则可以在标记中插入 - 符号，比如说 **date '+%-H:%-M:%-S'** 会把时分秒中无意义的 0 给去掉，像是原本的 08:09:04 会变为 8:9:4。另外，只有取得权限者(比如说 root)才能设定系统时间。

当您以 root 身分更改了系统时间之后，请记得以 **clock -w** 来将系统时间写入 **CMOS** 中，这样下次重新开机时系统时间才会持续保持最新的正确值。





## grep

Linux grep (global regular expression) 命令用于查找文件里符合条件的字符串或正则表达式。

grep 指令用于查找内容包含指定的范本样式的文件，如果发现某文件的内容符合所指定的范本样式，预设 grep 指令会把含有范本样式的那一列显示出来。若不指定任何文件名称，或是所给予的文件名为 **-**，则 grep 指令会从标准输入设备读取数据。

### 语法

```
grep [options] pattern [files]
或
grep [-abcEFGhHilLnqrsvVwxy][-A<显示行数>][-B<显示列数>][-C<显示列数>][-d<进行动作>][-e<范本样式>][-f<范本文件>][--help][范本样式][文件或目录...]
```

- pattern - 表示要查找的字符串或正则表达式。
- files - 表示要查找的文件名，可以同时查找多个文件，如果省略 files 参数，则默认从标准输入中读取数据。

**常用选项：**：

- `-i`：忽略大小写进行匹配。
- `-v`：反向查找，只打印不匹配的行。
- `-n`：显示匹配行的行号。
- `-r`：递归查找子目录中的文件。
- `-l`：只打印匹配的文件名。
- `-c`：只打印匹配的行数。

**更多参数说明**：

- **-a 或 --text** : 不要忽略二进制的数据。
- **-A<显示行数> 或 --after-context=<显示行数>** : 除了显示符合范本样式的那一列之外，并显示该行之后的内容。
- **-b 或 --byte-offset** : 在显示符合样式的那一行之前，标示出该行第一个字符的编号。
- **-B<显示行数> 或 --before-context=<显示行数>** : 除了显示符合样式的那一行之外，并显示该行之前的内容。
- **-c 或 --count** : 计算符合样式的列数。
- **-C<显示行数> 或 --context=<显示行数>或-<显示行数>** : 除了显示符合样式的那一行之外，并显示该行之前后的内容。
- **-d <动作> 或 --directories=<动作>** : 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep指令将回报信息并停止动作。
- **-e<范本样式> 或 --regexp=<范本样式>** : 指定字符串做为查找文件内容的样式。
- **-E 或 --extended-regexp** : 将样式为延伸的正则表达式来使用。
- **-f<规则文件> 或 --file=<规则文件>** : 指定规则文件，其内容含有一个或多个规则样式，让grep查找符合规则条件的文件内容，格式为每行一个规则样式。
- **-F 或 --fixed-regexp** : 将样式视为固定字符串的列表。
- **-G 或 --basic-regexp** : 将样式视为普通的表示法来使用。
- **-h 或 --no-filename** : 在显示符合样式的那一行之前，不标示该行所属的文件名称。
- **-H 或 --with-filename** : 在显示符合样式的那一行之前，表示该行所属的文件名称。
- **-i 或 --ignore-case** : 忽略字符大小写的差别。
- **-l 或 --file-with-matches** : 列出文件内容符合指定的样式的文件名称。
- **-L 或 --files-without-match** : 列出文件内容不符合指定的样式的文件名称。
- **-n 或 --line-number** : 在显示符合样式的那一行之前，标示出该行的列数编号。
- **-o 或 --only-matching** : 只显示匹配PATTERN 部分。
- **-q 或 --quiet或--silent** : 不显示任何信息。
- **-r 或 --recursive** : 此参数的效果和指定"-d recurse"参数相同。
- **-s 或 --no-messages** : 不显示错误信息。
- **-v 或 --invert-match** : 显示不包含匹配文本的所有行。
- **-V 或 --version** : 显示版本信息。
- **-w 或 --word-regexp** : 只显示全字符合的列。
- **-x --line-regexp** : 只显示全列符合的列。
- **-y** : 此参数的效果和指定"-i"参数相同。





## paste

Linux paste 命令用于合并文件的列。

paste 指令会把每个文件以列对列的方式，一列列地加以合并。

### 语法

```
paste [-s][-d <间隔字符>][--help][--version][文件...]
```

**参数**：

- -d<间隔字符>或--delimiters=<间隔字符> 　用指定的间隔字符取代跳格字符。
- -s或--serial 　串列进行而非平行处理。
- --help 　在线帮助。
- --version 　显示帮助信息。
- [文件…] 指定操作的文件路径





## chown

Linux chown（英文全拼：**change owner**）命令用于设置文件所有者和文件关联组的命令。

Linux/Unix 是多人多工操作系统，所有的文件皆有拥有者。利用 chown 将指定文件的拥有者改为指定的用户或组，用户可以是用户名或者用户 ID，组可以是组名或者组 ID，文件是以空格分开的要改变权限的文件列表，支持通配符。 。

chown 需要超级用户 **root** 的权限才能执行此命令。

只有超级用户和属于组的文件所有者才能变更文件关联组。非超级用户如需要设置关联组可能需要使用 chgrp 命令。

**使用权限** : root

### 语法

```
chown [-cfhvR] [--help] [--version] user[:group] file...
```

**参数** :

- user : 新的文件拥有者的使用者 ID
- group : 新的文件拥有者的使用者组(group)
- -c : 显示更改的部分的信息
- -f : 忽略错误信息
- -h :修复符号链接
- -v : 显示详细的处理信息
- -R : 处理指定目录以及其子目录下的所有文件
- --help : 显示辅助说明
- --version : 显示版本

### chgrp

Linux chgrp（英文全拼：change group）命令用于变更文件或目录的所属群组。

与 [chown](https://www.runoob.com/linux/linux-comm-chown.html) 命令不同，chgrp 允许普通用户改变文件所属的组，只要该用户是该组的一员。

在 UNIX 系统家族里，文件或目录权限的掌控以拥有者及所属群组来管理。您可以使用 chgrp 指令去变更文件与目录的所属群组，设置方式采用群组名称或群组识别码皆可。

### 语法

```
chgrp [-cfhRv][--help][--version][所属群组][文件或目录...] 或 chgrp [-cfhRv][--help][--reference=<参考文件或目录>][--version][文件或目录...]
```

### 参数说明

**-c 或 --changes**：效果类似"-v"参数，但仅回报更改的部分。

**-f 或 --quiet 或 --silent**： 　不显示错误信息。

**-h 或 --no-dereference**： 　只对符号连接的文件作修改，而不改动其他任何相关文件。

**-R 或 --recursive**： 　递归处理，将指定目录下的所有文件及子目录一并处理。

**-v 或 --verbose**： 　显示指令执行过程。

**--help**： 　在线帮助。

**--reference=<参考文件或目录>**： 　把指定文件或目录的所属群组全部设成和参考文件或目录的所属群组相同。

**--version**： 　显示版本信息。





## chattr

Linux chattr命令用于改变文件属性。

这项指令可改变存放在ext2文件系统上的文件或目录属性，这些属性共有以下9种模式：

1. a：让文件或目录仅供附加用途。
2. b：不更新文件或目录的最后存取时间。
3. c：将文件或目录压缩后存放。
4. d：将文件或目录排除在倾倒操作之外。
5. i：不得任意更动文件或目录。
6. s：保密性删除文件或目录。
7. S：即时更新文件或目录。
8. u：预防意外删除。
9. A : 文件或目录的Atime将不可修改

### 语法

```
chattr [-RV][-v<版本编号>][+/-/=<属性>][文件或目录...]
```

### 参数

　　-R 递归处理，将指定目录下的所有文件及子目录一并处理。

　　-v<版本编号> 设置文件或目录版本。

　　-V 显示指令执行过程。

　　+<属性> 开启文件或目录的该项属性。

　　-<属性> 关闭文件或目录的该项属性。

　　=<属性> 指定文件或目录的该项属性。





## whereis

Linux whereis命令用于查找文件。

该指令会在特定目录中查找符合条件的文件。这些文件应属于原始代码、二进制文件，或是帮助文件。

该指令只能用于查找二进制文件、源代码文件和man手册页，一般文件的定位需使用locate命令。

### 语法

```
whereis [-bfmsu][-B <目录>...][-M <目录>...][-S <目录>...][文件...]
```

**参数**：



-b 　只查找二进制文件。

-B<目录> 　只在设置的目录下查找二进制文件。

-f 　不显示文件名前的路径名称。

-m 　只查找说明文件。

-M<目录> 　只在设置的目录下查找说明文件。

-s 　只查找原始代码文件。

-S<目录> 　只在设置的目录下查找原始代码文件。

-u 　查找不包含指定类型的文件。





## sudo

在**/etc/sudoers**中定义可使用者

 ### 语法

 ```
 sudo -V
 sudo -h
 sudo -l
 sudo -v
 sudo -k
 sudo -s
 sudo -H
 sudo [ -b ] [ -p prompt ] [ -u username/#uid] -s
 sudo command
 ```

 **参数说明**：

 - -V 显示版本编号
 - -h 会显示版本编号及指令的使用方式说明
 - -l 显示出自己（执行 sudo 的使用者）的权限
 - -v 因为 sudo 在第一次执行时或是在 N 分钟内没有执行（N 预设为五）会问密码，这个参数是重新做一次确认，如果超过 N 分钟，也会问密码
 - -k 将会强迫使用者在下一次执行 sudo 时问密码（不论有没有超过 N 分钟）
 - -b 将要执行的指令放在背景执行
 - -p prompt 可以更改问密码的提示语，其中 %u 会代换为使用者的帐号名称， %h 会显示主机名称
 - -u username/#uid 不加此参数，代表要以 root 的身份执行指令，而加了此参数，可以以 username 的身份执行指令（#uid 为该 username 的使用者号码）
 - -s 执行环境变数中的 SHELL 所指定的 shell ，或是 /etc/passwd 里所指定的 shell
 - -H 将环境变数中的 HOME （家目录）指定为要变更身份的使用者家目录（如不加 -u 参数就是系统管理者 root ）
 - command 要以系统管理者身份（或以 -u 更改为其他人）执行的指令

## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
test    ALL=(ALL)       ALL






## firewall-cmd

```fiewall
firewall-cmd --version  # 查看版本
firewall-cmd --help     # 查看帮助

# 查看设置：
firewall-cmd --state  # 显示状态
firewall-cmd --get-active-zones  # 查看区域信息
firewall-cmd --get-zone-of-interface=eth0  # 查看指定接口所属区域
firewall-cmd --panic-on  # 拒绝所有包
firewall-cmd --panic-off  # 取消拒绝状态
firewall-cmd --query-panic  # 查看是否拒绝

firewall-cmd --reload # 更新防火墙规则
firewall-cmd --complete-reload
# 两者的区别就是第一个无需断开连接，就是firewalld特性之一动态添加规则，第二个需要断开连接，类似重启服务


# 将接口添加到区域，默认接口都在public
firewall-cmd --zone=public --add-interface=eth0
# 永久生效再加上 --permanent 然后reload防火墙

# 设置默认接口区域，立即生效无需重启
firewall-cmd --set-default-zone=public

# 查看所有打开的端口：
firewall-cmd --zone=dmz --list-ports

# 加入一个端口到区域：
firewall-cmd --zone=dmz --add-port=8080/tcp
# 若要永久生效方法同上

# 打开一个服务，类似于将端口可视化，服务需要在配置文件中添加，/etc/firewalld 目录下有services文件夹，这个不详细说了，详情参考文档
firewall-cmd --zone=work --add-service=smtp

# 移除服务
firewall-cmd --zone=work --remove-service=smtp

# 显示支持的区域列表
firewall-cmd --get-zones

# 设置为家庭区域
firewall-cmd --set-default-zone=home

# 查看当前区域
firewall-cmd --get-active-zones

# 设置当前区域的接口
firewall-cmd --get-zone-of-interface=enp03s

# 显示所有公共区域（public）
firewall-cmd --zone=public --list-all

# 临时修改网络接口（enp0s3）为内部区域（internal）
firewall-cmd --zone=internal --change-interface=enp03s

# 永久修改网络接口enp03s为内部区域（internal）
firewall-cmd --permanent --zone=internal --change-interface=enp03s
```

