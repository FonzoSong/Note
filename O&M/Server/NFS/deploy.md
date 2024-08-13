# NFS 服务简介

  

## 什么是 NFS？

  

NFS 就是 Network File System 的缩写，它最大的功能就是可以通过网络，让不同的机器、不同的操作系统可以共享彼此的文件。

  

 NFS 服务器可以让 PC 将网络中的 NFS 服务器共享的目录挂载到本地端的文件系统中，而在本地端的系统中来看，那个远程主机的目录就好像是自己的一个磁盘分区一样，在使用上相当便利；

  

NFS 一般用来存储共享视频，图片等静态数据。

  

## NFS 挂载原理：

  

![在这里插入图片描述](./NFSfile.assets/70.jpeg)

  
  
  

当我们在 NFS 服务器设置好一个共享目录 /home/public 后，其他的有权访问 NFS 服务器的 NFS 客户端就可以将这个目录挂载到自己文件系统的某个挂载点，这个挂载点可以自己定义，如上图客户端 A 与客户端 B 挂载的目录就不相同。并且挂载好后我们在本地能够看到服务端 /home/public 的所有数据。如果服务器端配置的客户端只读，那么客户端就只能够只读。如果配置读写，客户端就能够进行读写。挂载后，NFS 客户端查看磁盘信息命令：#df –h。

  

既然 NFS 是通过网络来进行服务器端和客户端之间的数据传输，那么两者之间要传输数据就要有想对应的网络端口，NFS 服务器到底使用哪个端口来进行数据传输呢？基本上 NFS 这个服务器的端口开在 2049, 但由于文件系统非常复杂。因此 NFS 还有其他的程序去启动额外的端口，这些额外的用来传输数据的端口是随机选择的，是小于 1024 的端口；既然是随机的那么客户端又是如何知道 NFS 服务器端到底使用的是哪个端口呢？这时就需要通过远程过程调用（Remote Procedure Call,RPC）协议来实现了！

  

## RPC 与 NFS 通讯原理：

  

 因为 NFS 支持的功能相当多，而不同的功能都会使用不同的程序来启动，每启动一个功能就会启用一些端口来传输数据，因此 NFS 的功能对应的端口并不固定，客户端要知道 NFS 服务器端的相关端口才能建立连接进行数据传输，而 RPC 就是用来统一管理 NFS 端口的服务，并且统一对外的端口是 111，RPC 会记录 NFS 端口的信息，如此我们就能够通过 RPC 实现服务端和客户端沟通端口信息。PRC 最主要的功能就是指定每个 NFS 功能所对应的 port number, 并且通知客户端，记客户端可以连接到正常端口上去。

  

**那么 RPC 又是如何知道每个 NFS 功能的端口呢？**

  

首先当 NFS 启动后，就会随机的使用一些端口，然后 NFS 就会向 RPC 去注册这些端口，RPC 就会记录下这些端口，并且 RPC 会开启 111 端口，等待客户端 RPC 的请求，如果客户端有请求，那么服务器端的 RPC 就会将之前记录的 NFS 端口信息告知客户端。如此客户端就会获取 NFS 服务器端的端口信息，就会以实际端口进行数据的传输了。

  

> 注意：在启动 NFS SERVER 之前，首先要启动 RPC 服务（即 portmap 服务，下同）否则 NFS SERVER 就无法向 RPC 服务区注册，另外，如果 RPC 服务重新启动，原来已经注册好的 NFS 端口数据就会全部丢失。因此此时 RPC 服务管理的 NFS 程序也要重新启动以重新向 RPC 注册。特别注意：一般修改 NFS 配置文档后，是不需要重启 NFS 的，直接在命令执行 systemctl reload nfs 或 exportfs –rv 即可使修改的 /etc/exports 生效

>

  

## NFS 客户端和 NFS 服务器通讯过程：

  

![在这里插入图片描述](./NFSfile.assets/70-1702465457840-3.jpeg)

首先服务器端启动 RPC 服务，并开启 111 端口

  

服务器端启动 NFS 服务，并向 RPC 注册端口信息

  

客户端启动 RPC（portmap 服务），向服务端的 RPC (portmap) 服务请求服务端的 NFS 端口

  

服务端的 RPC (portmap) 服务反馈 NFS 端口信息给客户端。

  

客户端通过获取的 NFS 端口来建立和服务端的 NFS 连接并进行数据的传输。

  
  
  
  
  

# 安装手册

  
  

curl -o /etc/yum.repos.d/centos.repo http://mirrors.aliyun.com/repo/Centos-7.repo

  

## 服务端

  

- [ ] 安装NFS包

  

```shell

yum install -y nfs-utils  rpcbind

```

  
  
  

- [ ] 启动服务并设置开机自启

  

```shell

systemctl start rpcbind

systemctl enable rpcbind

systemctl start nfs-server

systemctl enable nfs-server

```

  
  
  

- [ ] 防火墙开放

  

```shell

firewall-cmd --permanent --add-service=rpc-bind

firewall-cmd --permanent --add-service=nfs

firewall-cmd --permanent --add-service=mountd

firewall-cmd --reload

  

```

  
  
  

- [ ] 编辑配置文件

  

> vim /etc/exports

  

```shell

/data/local 192.168.162.0/24(rw,no_root_squash,async)

/data/visitor * (ro,all_squash)

```

  
  
  

- [ ] 创建共享目录

  

```shell

mkdir /data/local -p

mkdir /data/visitor -p

```

  
  
  

## 客户端

  

- [ ] 安装NFS包

  

```shell

yum install -y nfs-utils  rpcbind

```

  
  
  

- [ ] 启动服务并自启动

  

```shell

systemctl start nfs-server

systemctl enable nfs-server

```

  
  
  

- [ ] 开放防火墙

  

```shell

firewall-cmd --permanent --add-service=rpc-bind

firewall-cmd --permanent --add-service=nfs

firewall-cmd --permanent --add-service=mountd

firewall-cmd --reload

  

```

  
  
  

- [ ] 测试

  

```shell

showmount -e 192.168.162.8

```

  
  
  

- [ ] 创建挂载目录

  

```shell

mkdir /data

```

  
  
  

- [ ] 编写自动挂载配置文件

  

> vim /etc/fstab

  

```shell

192.168.162.8:/data/local  /data nfs defaults 0 0

```

  
  
  

- [ ] 挂载

  

```shell

mount -a

```

  
  
  

# 细节补充

  

## NFS 配置文件的参数：

  

|    **参数**    |                           **作用**                           |

| :------------: | :----------------------------------------------------------: |

|       ro       |                             只读                             |

|       rw       |                             读写                             |

|  root_squash   | 当 NFS 客户端以 root 管理员访问时，映射为 NFS 服务器的匿名用户 |

| no_root_squash | 当 NFS 客户端以 root 管理员访问时，映射为 NFS 服务器的 root 管理员 |

|   all_squash   | 无论 NFS 客户端使用什么账户访问，均映射为 NFS 服务器的匿名用户 |

|      sync      |         同时将数据写入到内存与硬盘中，保证不丢失数据         |

|     async      | 优先将数据保存到内存，然后再写入硬盘；这样效率更高，但可能会丢失数据 |

  

## showmount 命令的用法；

  

| 参数 | 作用                                        |

| ---- | ------------------------------------------- |

| -e   | 显示 NFS 服务器的共享列表                   |

| -a   | 显示本机挂载的文件资源的情况 NFS 资源的情况 |

| -v   | 显示版本号                                  |

  

## WIN上的NFS挂载

  

第一步：在控制面板–> 添加程序和功能–> 添加 NFS 组件。![在这里插入图片描述](./NFSfile.assets/70-1702469797947-9.png)

  
  

第二步：在此电脑，映射驱动器中添加 nfs 地址，和要共享的文件夹。

  

![在这里插入图片描述](./NFSfile.assets/70-1702469792735-6.png)

  

第三步：如果权限有问题，打开注册表：regedit, 在 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default 下新建两个 OWORD（64）位值，添加值 AnonymousGid，值默认为 0，AnonymousUid，值默认为 0。