# 安装手册

  

- [ ] 安装samba

  

```shell

yum install samba samba-client samba-swat

```

  
  
  

- [ ] 启动服务

  

```shell

systemctl start smb

systemctl start nmb

```

  
  
  

- [ ] 更改配置文件

  

```配置文件

[global]

        workgroup = SAMBA                  #表示设置工作组名称

        security = user                    #表示设置安全级别，其值可为 share、user、server、domain

  

        passdb backend = tdbsam           #表示设置共享帐户文件的类型，其值可为 tdbsam(tdb数据库文件)、ldapsam(LDAP目录                                            认证)、smbpasswd(兼容旧版本 samba 密码文件)

        printing = cups

        printcap name = cups

        load printers = yes

        cups options = raw

        disable spoolss=yes                #控制是否禁用 Samba 的打印支持功能

[homes]

        comment = Home Directories

        valid users = %S, %D%w%S           #允许访问该共享的用户

        browseable = No                    #共享是否可被查看

        read only = No

        inherit acls = Yes

  

[printers]

        comment = All Printers              #共享描述

        path = /var/tmp                     #共享路径

        printable = Yes                     #是否可以打印

        create mask = 0600

        browseable = No                     #共享是否可被查看

  

[print$]

        comment = Printer Drivers              #共享描述

        path = /var/lib/samba/drivers          #表示共享目录的路径

        write list = @printadmin root          #表示设置允许写的用户和组，组要用 @ 表示

        force group = @printadmin

        create mask = 0664

        directory mask = 0775

[samba]

       path=/var/sambafile       # 将要共享的目录

       browseable=yes        # 操作权限

       public=yes            # 访问权限

       writable=yes          # 对文件的操作权限

```

  
  
  

- [ ] 创建共享目录并赋予权限

  

```shell

mkdir /var/sambafile

chmod 777 /var/sambafile

```

  
  
  

- [ ] 创建samba用户

  

```shell

smbpasswd -a root

```

  
  
  

- [ ] 重建samba服务

  

```shell

service smb restart

```

  
  
  

- [ ] 修改[selinux](https://zhuanlan.zhihu.com/p/165974960)设置

  

文件位置

  

> /etc/sysconfig/selinux

  

```shell

SELINUX=disabled

```

  

# 介绍

  

**Samba服务器**是一个开源的网络文件共享服务，其主要功能是在不同操作系统之间实现文件和打印机共享。它最常用于将Linux/Unix系统与Windows系统互联，但也支持其他操作系统。后来微软又把 SMB 改名为 **CIFS**（Common Internet File System），即公共 Internet 文件系统，并且加入了许多新的功能，这样一来，使得Samba具有了更强大的功能。

  

主要功能：

  

-  文件共享：Samba允许用户在网络上共享文件和目录，使其可从不同计算机和操作系统访问。这使得在混合环境中（例如Linux、Windows和macOS）共享文件变得容易。

-  打印机共享：Samba还可以用于共享网络打印机，以便多台计算机可以通过网络访问共享的打印设备。

-  身份验证和授权：Samba支持不同类型的身份验证和授权机制，包括基于用户的、基于域的和Kerberos身份验证。这使得用户可以通过其用户名和密码访问共享资源，并且管理员可以对访问进行更精细的控制。

-  集成Windows域控制器：Samba还可以用作Windows域控制器的替代品，允许管理Windows域内的用户、组和计算机。这使得在没有Windows服务器的情况下构建和管理Windows域成为可能。

-  数据备份：Samba可以用于将文件从不同计算机备份到中央存储位置，以确保数据的可用性和冗余。

  

## Samba服务组成

  

Samba服务器提供smbd、nmbd两个服务程序，分别完成不同的功能

  

`smbd`：其主要功能就是用来管理Samba服务器上的共享目录、打印机等，主要是针对网络上的共享资源进行管理的服务。当要访问服务器时，要查找共享文件，这时我们就要依靠smbd这个进程来管理数据传输。

`nmbd`：其功能是进行NetBIOS名解析，并提供浏览服务显示网络上的共享资源列表。