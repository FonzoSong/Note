**云基础架构平台软件（私有云平台）**

# 1.    简介

openstack-train.tar.gz包含OpenStack T版本私有云平台搭建的各项软件包、依赖包、安装脚本等，同时还提供了CentOS7.2、CentOS7.5、CentOS7.9等云主机qcow2镜像，可满足私有云平台的搭建、云平台的使用、各组件的运维操作等。

openstack-train.tar.gz包含的具体内容如下：

|   |   |   |
|---|---|---|
|编号|软件包|详细信息|
|1|openstack|提供安装脚本，可用安装脚本快捷部署OpenStack私有云平台|
|可用于安装KeyStone服务，以及对keystone认证服务进行创建用户、租户、管理权限等操作|
|可用于安装Glance服务，以及对glance服务进行上传镜像、删除镜像、创建快照等操作|
|可用于安装Nova服务，以及对nova服务进行启动云主机、创建云主机类型、删除云主机等操作|
|可用于安装Neutron服务，以及对neutron服务进行创建网络、删除网络、编辑网络等操作|
|可用于安装Horzion服务，可以通过Horzion Dashboard界面对OpenStack平台进行管理|
|可用于安装Cinder服务，以及对Cinder服务进行创建块设备、管理块设备连接、删除块设备等操作|
|可用于安装Swift服务，以及对Swift服务进行创建容器、上传对象、删除对象等操作|
|可用于安装Heat服务，可通过编辑模板文件，实现Heat编排操作|
|2|images|提供CentOS_7.2_x86_64_GJ.qcow2镜像，该镜像为CentOS7.2版本的虚拟机镜像，可基于该镜像启动CentOS7.2的云主机，用于各项操作与服务搭建|
|提供CentOS_7.5_x86_64_GJ.qcow2镜像，该镜像为CentOS7.5版本的虚拟机镜像，可基于该镜像启动CentOS7.5的云主机，用于各项操作与服务搭建|
|提供CentOS_7.9_x86_64_GJ.qcow2镜像，该镜像为CentOS7.9版本的虚拟机镜像，可基于该镜像启动CentOS7.9的云主机，用于各项操作与服务搭建|

# 1基础环境配置

## 1.1. 安装centos7说明

**【****CentOS7****版本】**

CentOS7系统选择2009版本：CentOS-7-x86_64-DVD-2009.iso

**【空白分区划分】**

[root@compute ~]# fdisk /dev/md126

 5  10520893440  11569469439    100G  Linux filesyste

 6  11569469440  12618045439    100G  Linux filesyste

 7  12618045439  13676621439    100G  Linux filesyste

划分出三个100G的分区供cinder、swift、manila使用

## 1.2. 配置网络、主机名

修改和添加/etc/sysconfig/network-scripts/ifcfg-**enp***（具体的网口）文件。

### 1.2.1.   controller节点

**enp7s0****：****192.168.100.10**

TYPE=Ethernet

BOOTPROTO=static

DEFROUTE=yes

DEVICE=enp7s0

ONBOOT=yes

IPADDR=192.168.100.10

NETMASK=255.255.255.0

GATEWAY=192.168.100.1

[root@controller ~]# service network restart

Restarting network (via systemctl):                        [  OK  ]

[root@controller ~]# hostnamectl set-hostname controller

[root@controller ~]# vi /etc/hosts

192.168.100.10 controller

192.168.100.20 compute

按ctrl+d 退出  重新登陆

### 1.2.2.   compute节点

**enp7s0****：****192.168.100.20**

TYPE=Ethernet
BOOTPROTO=static

DEFROUTE=yes

DEVICE=enp7s0

ONBOOT=yes

IPADDR=192.168.100.20

NETMASK=255.255.255.0

GATEWAY=192.168.100.1

[root@compute ~]# service network restart

Restarting network (via systemctl):                        [  OK  ]

[root@compute ~]# hostnamectl set-hostname compute

[root@compute ~]# vi /etc/hosts

192.168.100.10 controller

192.168.100.20 compute

按ctrl+d 退出  重新登陆

## 1.3. 配置yum源

### 1.3.1.   controller节点

**(1)**      **删除centos****源**

[root@controller ~]# rm -rf /etc/yum.repos.d/*

**(2)**      **编写repo****文件**

[root@controller ~]# vi /etc/yum.repos.d/local.repo

[centos]

name=centos

baseurl=file:///opt/centos

gpgcheck=0

enabled=1

[openstack]

name=openstack

baseurl=file:///opt/openstack

gpgcheck=0

enabled=1

**(3)**      **将iso****文件配置为yum****源**

[root@controller ~]# mkdir /opt/centos/

[root@controller ~]# mount -o loop CentOS-7-x86_64-DVD-2009.iso /mnt/

[root@controller ~]# cp -rvf /mnt/* /opt/centos/

[root@controller ~]# umount /mnt/

将openstack-train.tar.gz文件上传至/root目录下

[root@controller ~]# tar -zxvf openstack-train.tar.gz -C /opt/

**(4)**      **清除缓存、验证yum****源**

[root@controller ~]# yum clean all

[root@controller ~]# yum repolist

**(5)**      **配置防火墙和selinux**

[root@controller ~]# systemctl stop firewalld

[root@controller ~]# systemctl disable firewalld

[root@controller ~]# vi /etc/selinux/config

SELINUX=permissive

[root@controller ~]# setenforce 0  #临时生效

**(6)**      **搭建ftp****服务器并设置开机自启**

[root@controller ~]# yum install -y vsftpd

[root@controller ~]# vi /etc/vsftpd/vsftpd.conf

anon_root=/opt

保存退出

[root@controller ~]# systemctl restart vsftpd

[root@controller ~]# systemctl enable vsftpd

### 1.3.2.   compute节点

**(1)**      **删除centos****源**

[root@compute ~]# rm -rf /etc/yum.repos.d/*

**(2)**      **编写repo****文件**

[root@compute ~]# vi /etc/yum.repos.d/ftp.repo

[centos]

name=centos

baseurl=ftp://controller/centos

gpgcheck=0

enabled=1

[openstack]

name=openstack

baseurl=ftp://controller/openstack

gpgcheck=0

enabled=1

**(3)**      **配置防火墙和selinux**

[root@compute ~]# systemctl stop firewalld

[root@compute ~]# systemctl disable firewalld

[root@compute ~]# vi /etc/selinux/config

SELINUX=permissive

[root@compute ~]# setenforce 0  #临时生效

**(4)**      **清除缓存、验证yum****源**

[root@compute ~]# yum clean all

[root@compute ~]# yum repolist

## 1.4. 配置环境变量

### 1.4.1.   安装

**# controller****和****compute****节点执行**yum install -y openstack-shell

### 1.4.2.   controller节点

variable.sh 是安装时的各项参数，需根据服务器实际情况配置

[root@controller ~]# vi /root/variable.sh

HOST_IP=192.168.100.10

HOST_PASS=000000

HOST_NAME=controller

HOST_IP_NODE=192.168.100.20

HOST_PASS_NODE=000000

HOST_NAME_NODE=compute

network_segment_IP=192.168.100.0/24

RABBIT_USER=openstack

RABBIT_PASS=000000

DB_PASS=000000

DOMAIN_NAME=demo

ADMIN_PASS=000000

DEMO_PASS=000000

KEYSTONE_DBPASS=000000

GLANCE_DBPASS=000000

GLANCE_PASS=000000

NOVA_DBPASS=000000

NOVA_PASS=000000

NEUTRON_DBPASS=000000

NEUTRON_PASS=000000

METADATA_SECRET=000000

INTERFACE_IP_HOST=192.168.100.10（controller节点ip）

INTERFACE_IP_NODE=192.168.100.20（compute节点ip）

INTERFACE_NAME_HOST=enp8s0（controller节点外部网络网卡名称）

INTERFACE_NAME_NODE=enp8s0（compute节点外部网络网卡名称）

Physical_NAME=provider（外部网络适配器名称）

minvlan=101（vlan网络范围的第一个vlanID）

maxvlan=200（vlan网络范围的最后一个vlanID）

CINDER_DBPASS=000000

CINDER_PASS=000000

BLOCK_DISK=md126p5（空白分区）

SWIFT_PASS=000000

OBJECT_DISK=md126p6（空白分区）

STORAGE_LOCAL_NET_IP=192.168.100.20

HEAT_DBPASS=000000

HEAT_PASS=000000

CEILOMETER_DBPASS=000000

CEILOMETER_PASS=000000

MANILA_DBPASS=000000

MANILA_PASS=000000

SHARE_DISK= md126p7（空白分区）

CLOUDKITTY_DBPASS=000000

CLOUDKITTY_PASS=000000

BARBICAN_DBPASS=000000

BARBICAN_PASS=000000

**将编写好的变量文件发给****compute****节点**

[root@controller ~]# scp variable.sh compute:/root/

variable.sh        100%  748     1.1MB/s   00:00

[root@controller ~]source /root/variable.sh

[root@compute ~]source /root/variable.sh

# 2.   通过脚本安装服务

## 2.1. 执行初始化脚本

**# controller****和****compute****节点**

**执行脚本openstack-completion.sh****进行安装**

## 2.2. 安装Mysql数据库服务

**# Controller****节点**

**执行脚本openstack-controller-mysql.sh****进行安装**

## 2.3. 安装Keystone认证服务

**# Controller****节点**

**执行脚本openstack-controller-keystone.sh****进行安装**

## 2.4. 安装Glance镜像服务

**# Controller****节点**

**执行脚本****openstack-controller-glance****.sh****进行安装**

## 2.5. 安装Nova计算服务

**# Controller****节点**

**执行脚本****openstack-****controller-nova****.sh****进行安装**

**# Compute****节点**

**执行脚本****openstack-compute-nova.sh****进行安装**

## 2.6. 安装Neuron网络服务

**# Controller****节点**

**执行脚本****openstack-****controller-neutron****.sh****进行安装**

**# Compute****节点**

**执行脚本****openstack-compute-neutron.sh****进行安装**

## 2.7. 安装Dashboard服务

**# Controller****节点**

**执行脚本****openstack-****controller-dashboard****.sh****进行安装**

## 2.8. 安装Cinder块存储服务

**# Controller****节点**

**执行脚本****openstack-****controller-cinder****.sh****进行安装**

**# Compute****节点**

**执行脚本****openstack-compute****-cinder****.sh****进行安装**

## 2.9. 安装Swift块存储服务

**# Controller****节点**

**执行脚本****openstack-****controller-swift****.sh****进行安装**

**# Compute****节点**

**执行脚本****openstack-compute****-swift****.sh****进行安装**

## 2.10.   安装Heat编排服务

**# Controller****节点**

**执行脚本****openstack-****controller-heat****.sh****进行安装**

## 2.11.   安装Manila文件共享服务

**# Controller****节点**

**执行脚本****openstack-****controller-manila****.sh****进行安装**

**# Compute****节点**

**执行脚本****openstack-compute****-manila****.sh****进行安装**

## 2.12.   安装Cloudkitty计费服务

**# Controller****节点**

**执行脚本****openstack-****controller-cloudkitty****.sh****进行安装**

## 2.13.   安装Barbican密钥管理服务

**# Controller****节点**

**执行脚本****openstack-controller-barbican.sh****进行安装**

## 2.14.   添加控制节点资源到云平台

### 2.14.1.            修改variable.sh

**在控制节点把variable.sh****文件中计算节点的ip****和主机名改为控制节点ip****和主机名**

**HOST_IP_NODE=192.168.100.10**

**HOST_NAME_NODE=controller**

### 2.14.2.            运行openstack-compute-nova.sh

**在控制节点运行****openstack-compute****-nova****.sh****进行安装**