
本文部署的 `openstack` 版本为 `rocky` ; [docs link](https://docs.openstack.org/rocky/configuration/)

### 约定:
仅限于  `/O&M/Server/openstack/*`
- 小标题标注 `CC` 为 `control` 与 `comput` 两个 `node` 都需要进行操作;
- 小标题标注 `cn` 为 只需在 `control node` 进行操作;
- 小标题标注 `cm` 为 只需在 `comput node` 进行操作;
- 所有操作使用 `root or sudo` 权限; 
- `control node` 最少 `2GB` 内存, `comput node` 最少 `4GB` 内存; 
- 所有节点建议至少 `40G` 磁盘容量;
- 防火墙与 `selinux` 为关闭状态;
- 在修改配置文件之前备份;

#### 基础环境说明:

| OS    | HostName | eth0 IP 外部与管理  | eth1 IP 隧道     |
| ----- | -------- | -------------- | -------------- |
| RHEL9 | control  | 172.148.188.10 | 192.168.166.1  |
| RHEL9 | comput   | 172.148.188.11 | 192.168.166.11 |

本文将仅使用两张网卡;将管理网络与外部网络进行合并;

完整的网络规划:
1. 管理网络(management/API网络):
	提供系统管理相关功能，用于节点之间各服务组件内部通信以及对数据库服务的访问，所有节点都需要连接到管理网络，这里管理网络也承载了API网络的流量，将API网络和管理网络合并，OpenStack各组件通过API网络向用户暴露API服务。
2. 隧道网络(tunnel网络或self-service网络):
	提供租户虚拟网络的承载网络(VXLAN or GRE)。OpenStack里面使用gre或vxlan模式，需要有隧道网络；隧道网络采用了点到点通信协议替代了交换连接，在openstack里，这个tunnel就是虚拟机走网络数据流量的。
3. 外部网络(external网络或provider网络):
	openstack网络至少包括一个外部网络，这个网络能够访问OpenStack安装环境之外的网络，并且非openstack环境中的设备能够访问openstack外部网络的某个IP。另外外部网络为openstack环境中的虚拟机提供浮动IP，实现openstack外部网络对内部虚拟机实例的访问。

---

### Basic

#### 配置主机解析 CC
编辑 `/etc/hosts` 内容
```shell
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.166.10 control
192.168.166.11 comput
```
`copy` 到 `comput node` 
```shell
scp /etc/hosts 192.168.166.11:/etc/hosts
```
#### NTP服务 cn
安装 `chrony` 服务 `dnf install -y chrony`;
编辑 `/etc/chron.conf`
`allow 192.168.0.0/16` 去掉注释 指定网段访问
`systemctl enable chronyd --now && systemctl status chronyd`

#### NTP服务 cm
安装 `chrony` 服务 `dnf install -y chrony`;
`#`server 0.centos.pool.ntp.org iburst
`#`server 1.centos.pool.ntp.org iburst
`#`server 2.centos.pool.ntp.org iburst
`#`server 3.centos.pool.ntp.org iburst  `#` 都注释掉
server controller iburst 添加这一行
`systemctl enable chronyd --now && systemctl status chronyd`
chronyc sources 查看同步状态;