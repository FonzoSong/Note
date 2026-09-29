# Linux NFS 配置流程

示例环境：服务端 `192.168.1.100`，客户端 `192.168.1.101`，共享目录 `/data/share`，挂载点 `/mnt/nfs`。

---

# 一、服务端

## 第一步：检查需要的包，启动需要的服务

**1. 安装/检查软件包**

```bash
# RHEL / CentOS / Rocky 安装
dnf install -y nfs-utils rpcbind      # CentOS 7 用 yum install -y nfs-utils rpcbind

# 检查（用 rpm -qa | grep 一把梭）
rpm -qa | grep nfs
rpm -qa | grep rpcbind
```

只要能看到 `nfs-utils` 和 `rpcbind` 就说明装好了。`libnfsidmap` 是 NFSv4 用户名映射的库，随 `nfs-utils` 自动装上。

```bash
# Ubuntu / Debian
apt update && apt install -y nfs-kernel-server
dpkg -l | grep -E 'nfs|rpcbind'
```

**2. 启动服务**

RHEL 系按习惯直接一条命令起这三个：

```bash
systemctl start rpcbind nfs-lock nfs
```

三个服务名与实际 unit 的对应关系：

| 你敲的名字 | 实际 unit | 作用 | 什么时候需要 |
| --- | --- | --- | --- |
| `rpcbind` | `rpcbind.service` | RPC 端口注册中心，客户端靠它找到 mountd 等服务的端口 | **NFSv3 必需**；纯 v4 可不启 |
| `nfs-lock` | `rpc-statd.service`（别名） | 文件锁状态通知（NLM），配合 v3 的 `lockd` | **NFSv3 必需**；v4 锁内建，可不启 |
| `nfs` | `nfs-server.service`（别名） | NFS 主服务，起 `nfsd`、`mountd` | 必需 |

**3. 检查服务是否起来**

```bash
# 逐个看状态
systemctl status rpcbind nfs-lock nfs
```



## 第二步：创建共享目录

```bash
mkdir -p /data/share
chmod 755 /data/share
ls -ldn /data/share              # -n 显示数字 UID/GID，方便和客户端比对
df -hT /data/share               # 确认底层文件系统和剩余空间
```

如果要让客户端普通用户能写，需要提前把属主改成对应 UID/GID（见第四步的权限说明）：

```bash
groupadd -g 1001 nfsgroup
useradd  -u 1001 -g 1001 nfsuser
chown -R 1001:1001 /data/share
chmod 775 /data/share
```

## 第三步：编写 /etc/exports

```bash
cp /etc/exports /etc/exports.bak    # 先备份
vi /etc/exports
```

**语法格式**

```
共享目录   客户端(选项1,选项2)
```

> ⚠️ 客户端和左括号之间**不能有空格**；多个客户端之间**用空格分隔**。这是最常见的写错原因。

**常用写法**

```bash
# 1. 共享给单个客户端，读写
/data/share 192.168.1.101(rw,sync,no_root_squash)

# 2. 共享给整个网段，读写，所有用户统一映射为 UID/GID 1001（生产推荐）
/data/share 192.168.1.0/24(rw,sync,all_squash,anonuid=1001,anongid=1001,no_subtree_check)

# 3. 只读共享
/data/share 192.168.1.0/24(ro,sync,no_subtree_check)

# 4. 不同客户端给不同权限
/data/share 192.168.1.101(rw,sync) 192.168.1.102(ro,sync)

# 5. 共享多个目录
/data/share  192.168.1.0/24(rw,sync,no_subtree_check)
/data/backup 192.168.1.0/24(ro,sync,no_subtree_check)

```

## 第四步：检查配置有没有写对

```bash
# 1. 最关键：查看内核里实际生效的导出表和选项
exportfs -rv
```

临时关闭防火墙与selinux

```
systemctl stop firewall
getenforce 0
```

逐项核对四件事：目录路径对不对、客户端网段对不对、是 `rw` 还是 `ro`、squash 策略是不是你要的。



## 第五步：权限选项详解

NFS 能不能读写由**两层权限**共同决定，任何一层拒绝都会失败：

1. **导出层**——`/etc/exports` 里的选项（下面这张表）
2. **文件系统层**——服务端共享目录本身的 Linux 属主/属组/mode

### 访问权限

| 选项 | 是否默认 | 作用 |
| --- | --- | --- |
| `rw` | 否 | 允许客户端读写 |
| `ro` | **是** | 只读。**不写 rw 就默认是只读**，这是最容易踩的坑 |

### 用户映射（决定客户端用户在服务端是什么身份）

| 选项 | 是否默认 | 作用 |
| --- | --- | --- |
| `root_squash` | **是** | 客户端的 root 被降级为服务端匿名用户（nobody/nfsnobody），防止远程 root 提权 |
| `no_root_squash` | 否 | 客户端 root 在服务端仍是 root，权限最大。**高危**，仅限完全可信的内网 |
| `all_squash` | 否 | 客户端所有用户都映射为匿名用户，配合 `anonuid/anongid` 可统一文件归属 |
| `no_all_squash` | **是** | 保留客户端原始的 UID/GID |
| `anonuid=<n>` | 65534 | 指定匿名用户映射到哪个 UID |
| `anongid=<n>` | 65534 | 指定匿名组映射到哪个 GID |

**三种典型组合的实际效果**

| exports 写法 | 客户端 root 建的文件属主 | 客户端普通用户建的文件属主 |
| --- | --- | --- |
| `(rw,sync)` 即默认 root_squash | nobody（65534） | 客户端原始 UID |
| `(rw,sync,no_root_squash)` | root（0） | 客户端原始 UID |
| `(rw,sync,all_squash,anonuid=1001,anongid=1001)` | 1001 | 1001 |

> NFSv3 完全靠 UID/GID 数字匹配身份。要让客户端普通用户能正常读写，**两端必须有相同 UID/GID 的用户**，且服务端目录属主要匹配。

### 写入方式

| 选项 | 是否默认 | 作用 |
| --- | --- | --- |
| `sync` | **是** | 同步写入磁盘，数据一致性好，性能较低。生产推荐 |
| `async` | 否 | 先写内存缓冲再落盘，性能高，但服务端断电可能丢数据 |
| `wdelay` | **是** | 合并相邻的写请求，减少磁盘访问（仅 `sync` 下生效） |
| `no_wdelay` | 否 | 立即写入 |

### 连接与目录检查

| 选项 | 是否默认 | 作用 |
| --- | --- | --- |
| `secure` | **是** | 要求客户端使用小于 1024 的特权端口连接 |
| `insecure` | 否 | 允许大于 1024 的端口。**容器、NAT、非 root 挂载场景需要加这个**，否则报 Permission denied |
| `no_subtree_check` | **是** | 不检查父目录权限，提升性能，官方推荐 |
| `subtree_check` | 否 | 检查父目录，导出子目录时更安全但更慢 |
| `crossmnt` | 否 | 允许客户端跨越该目录下的子挂载点（NFSv4 多目录共享需要） |
| `fsid=0` | 否 | 标记为 NFSv4 的伪根目录，客户端挂载路径要相对伪根来写 |
| `nohide` | 否 | 子目录被单独导出时不隐藏 |

### 安全认证

| 选项 | 作用 |
| --- | --- |
| `sec=sys` | 默认，基于 UID/GID，不加密 |
| `sec=krb5` | Kerberos 认证 |
| `sec=krb5i` | Kerberos 认证 + 完整性校验 |
| `sec=krb5p` | Kerberos 认证 + 完整性 + 加密（最安全） |

### 六个常用权限组合（可直接抄）

每个组合都给出「服务端目录准备 + exports 写法 + 实际效果 + 适用场景」，按推荐程度排列。

---

**组合 1：统一映射到专用用户（生产最推荐）**

```bash
# 服务端
groupadd -g 1001 nfsgroup
useradd  -u 1001 -g 1001 nfsuser
chown -R 1001:1001 /data/share
chmod 775 /data/share
```

```
/data/share 192.168.1.0/24(rw,sync,all_squash,anonuid=1001,anongid=1001,no_subtree_check)
```

- **效果**：客户端无论谁（含 root）写文件，属主都是 1001:1001，权限整齐可控
- **适用**：多台客户端共享、K8s/容器存储、备份归档
- **要点**：客户端要有 UID=1001 的用户才能以自己的身份读写；否则只有目录的 other 权限（775 下可读不可写）

---

**组合 2：默认 root_squash + 两端 UID 对齐（多用户协作推荐）**

```bash
# 服务端与客户端都执行，UID/GID 必须完全一致
groupadd -g 1001 devgroup
useradd  -u 1001 -g 1001 devuser
chown -R 1001:1001 /data/share
chmod 775 /data/share
```

```
/data/share 192.168.1.0/24(rw,sync,no_subtree_check)
```

- **效果**：客户端 root 被降级为 nobody（写不进去），普通用户 devuser 以自己的 UID 1001 正常读写，文件属主显示为 devuser
- **适用**：开发团队共享代码/资料、需要保留真实属主的场景
- **要点**：这是**默认行为**，不写 squash 选项就是它。跨机属主显示正确的前提是两端 UID/GID 一致（NFSv4 还要 `/etc/idmapd.conf` 的 Domain 一致）

---

**组合 3：完全放开 root（内网可信环境最省事）**

```bash
# 服务端
chown -R root:root /data/share
chmod 755 /data/share
```

```
/data/share 192.168.1.101(rw,sync,no_root_squash,no_subtree_check)
```

- **效果**：客户端 root 在服务端就是 root，任意读写、任意 chmod/chown
- **适用**：纯内网、少数几台可信主机、运维管理目录、ZFS/私有 NAS
- **⚠️ 风险**：客户端 root 可以改服务端文件权限甚至覆盖数据。**客户端地址必须写死具体 IP 或内网网段，绝不能用 `*`**，且防火墙必须限定来源网段

---

**组合 4：只读分发（软件仓库、配置分发、静态资源）**

```bash
# 服务端
chown -R root:root /data/repo
chmod 755 /data/repo
```

```
/data/repo 192.168.1.0/24(ro,sync,no_subtree_check)
```

- **效果**：所有客户端只能读，写入报 `Read-only file system`
- **适用**：yum/apt 本地仓库、ISO 镜像库、共享配置、模型文件分发
- **要点**：`ro` 其实就是默认值，但**建议显式写出来**，避免以后有人误以为漏写了 `rw`

---

**组合 5：读写分离（同一目录，不同客户端不同权限）**

```
/data/share 192.168.1.101(rw,sync,no_root_squash,no_subtree_check) 192.168.1.0/24(ro,sync,no_subtree_check)
```

- **效果**：101 这台可读写，网段内其他机器只能读
- **适用**：一台写入服务器 + 多台只读消费节点（如日志收集、内容发布）
- **要点**：多个客户端之间用**空格**分隔，各自的选项写在各自的括号里；LVM 的匹配规则是**精确 IP 优先于网段**

---

**组合 6：容器 / NAT 环境（非特权端口 + 匿名映射）**

```bash
# 服务端
chown -R 1000:1000 /data/share
chmod 777 /data/share        # 容器内 UID 不可控时的折中做法
```

```
/data/share 10.244.0.0/16(rw,sync,insecure,all_squash,anonuid=1000,anongid=1000,no_subtree_check)
```

- **效果**：允许来自 > 1024 端口的连接，所有请求统一映射为 1000
- **适用**：K8s PV、Docker 挂载、经过 NAT 的客户端
- **要点**：`insecure` 是必需的——容器和 NAT 环境的源端口通常是高位端口，不加会报 `mount.nfs: Permission denied`。安全性靠**网段限定 + 防火墙**兜底

---

**组合选择速查**

| 场景 | 选哪个 | 核心选项 |
| --- | --- | --- |
| 多客户端共享、要权限可控 | 组合 1 | `all_squash,anonuid,anongid` |
| 团队协作、要保留真实属主 | 组合 2 | 默认（`no_all_squash` + `root_squash`）+ 两端 UID 对齐 |
| 内网少数可信机、图省事 | 组合 3 | `no_root_squash` |
| 仓库 / 镜像 / 配置分发 | 组合 4 | `ro` |
| 一写多读 | 组合 5 | 多客户端分别写选项 |
| K8s / Docker / NAT | 组合 6 | `insecure` + `all_squash` |

**通用建议**：`sync` 一律保留（数据安全）；`no_subtree_check` 一律加上（性能与稳定性）；`async` 只在明确能接受断电丢数据时用。

### 防火墙放行（服务能起来 ≠ 客户端能连上）

```bash
# firewalld（RHEL 系）
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --reload
firewall-cmd --list-services

# ufw（Ubuntu 系）
ufw allow from 192.168.1.0/24 to any port 2049 proto tcp
ufw allow from 192.168.1.0/24 to any port 111  proto tcp
ufw reload
ufw status
```

需要放行的端口：**NFSv4 只要 2049/tcp**；**NFSv3 还要 111（rpcbind）、20048（mountd）、662（statd）**。

### SELinux（RHEL 系，开启状态下必做）

```bash
getenforce                       # 查看是否 Enforcing
getsebool -a | grep nfs

setsebool -P nfs_export_all_rw 1 # 允许读写共享（-P 表示永久生效）
```

## 第六步：做成开机自启与持久化

NFS 服务端的持久化分四件事，**配置文件本身是持久的，需要 `enable` 的是服务**：

| 要持久化的东西 | 做法 | 说明 |
| --- | --- | --- |
| 服务开机自启 | `systemctl enable --now nfs-server rpcbind` | `--now` 等于同时 start |
| 共享规则 | 写在 `/etc/exports` 里 | 文件本身就是持久化的，开机自动加载；临时改动必须写进文件，否则重启丢失 |
| 防火墙规则 | 加 `--permanent` 再 `firewall-cmd --reload` | 不加 `--permanent` 重启就没了 |
| SELinux 布尔值 | `setsebool -P` | 不加 `-P` 重启就没了 |

```bash
# RHEL / CentOS / Rocky（--now 等于 enable + start 一起做）
systemctl enable --now rpcbind nfs-lock nfs

# 或分两步
systemctl enable rpcbind nfs-lock nfs
systemctl start  rpcbind nfs-lock nfs

# Ubuntu / Debian
systemctl enable --now nfs-kernel-server
```

**检查自启是否配好**

```bash
systemctl is-enabled rpcbind nfs-lock nfs
systemctl list-unit-files | grep -E 'nfs|rpcbind'
```

```
enabled
enabled
enabled
```

```
nfs-server.service    enabled    ← nfs 指向它
rpc-statd.service     enabled    ← nfs-lock 指向它
rpcbind.service       enabled
```

> 用别名 `enable` 时，systemd 实际写入的是真实 unit（`nfs-server.service` / `rpc-statd.service`），所以 `list-unit-files` 里看到的是真实名字，这是正常的。

**验证持久化真正生效**

```bash
reboot
# 重启后逐项确认
systemctl is-active nfs-server rpcbind
exportfs -v                          # 导出表还在
firewall-cmd --list-services         # 防火墙规则还在
getsebool nfs_export_all_rw          # SELinux 布尔值还是 on
```

## 服务端完整流程（可直接复制）

```bash
# 1. 装包 + 起服务 + 设自启
dnf install -y nfs-utils rpcbind          # Ubuntu: apt install -y nfs-kernel-server
rpm -qa | grep -E 'nfs|rpcbind'           # 确认装好
systemctl start rpcbind nfs-lock nfs      # 启动这三个
systemctl enable rpcbind nfs-lock nfs     # 设开机自启
systemctl is-active rpcbind nfs-lock nfs

# 2. 建共享目录
mkdir -p /data/share
groupadd -g 1001 nfsgroup; useradd -u 1001 -g 1001 nfsuser
chown -R 1001:1001 /data/share && chmod 775 /data/share

# 3. 写 /etc/exports
echo '/data/share 192.168.1.0/24(rw,sync,all_squash,anonuid=1001,anongid=1001,no_subtree_check)' >> /etc/exports

# 4. 生效
exportfs -rav

# 5. 检查
exportfs -v
ss -tulnp | grep 2049
mkdir -p /mnt/selftest && mount -t nfs 127.0.0.1:/data/share /mnt/selftest && umount /mnt/selftest

# 6. 防火墙 + SELinux（持久化）
firewall-cmd --permanent --add-service={nfs,rpc-bind,mountd} && firewall-cmd --reload
setsebool -P nfs_export_all_rw 1
```

---

# 二、客户端

## 第一步：需要的包和要启动的服务

**1. 安装软件包**

```bash
# RHEL / CentOS / Rocky 安装
dnf install -y nfs-utils              # CentOS 7 用 yum install -y nfs-utils

# 检查（同样用 rpm -qa | grep）
rpm -qa | grep nfs
rpm -qa | grep -E 'nfs|rpcbind'
```

```
nfs-utils-2.5.4-21.el9.x86_64
libnfsidmap-2.5.4-21.el9.x86_64
```

客户端**只需要 `nfs-utils`**，它同时提供 `mount.nfs`、`showmount`、`rpcinfo`、`nfsstat`、`nfsidmap` 等全部工具，不需要单独装 rpcbind 包（`nfs-utils` 已依赖）。

```bash
# Ubuntu / Debian
apt update && apt install -y nfs-common
dpkg -l | grep -E 'nfs|rpcbind'

# Ubuntu 精简镜像报 mount.nfs not found 时补装内核模块
apt install -y linux-modules-extra-$(uname -r)
```

**2. 需要启动的服务**

客户端要启的服务比服务端少，按 NFS 版本区分：

```bash
# NFSv3：需要 rpcbind（找端口）+ nfs-lock（文件锁）
systemctl start rpcbind nfs-lock

# NFSv4：锁已内建、不依赖 rpcbind，理论上无需额外服务；
#        但为了让用户名映射正常（否则文件属主显示 nobody），建议起 nfs-idmapd
systemctl start nfs-idmapd

# 图省事，三个一起起也没问题
systemctl start rpcbind nfs-lock nfs-idmapd
```

| 服务 | 客户端作用 | 必需性 |
| --- | --- | --- |
| `rpcbind` | 向服务端查询 mountd 等 RPC 端口 | **v3 必需**，纯 v4 可不起 |
| `nfs-lock`（= `rpc-statd`） | v3 的文件锁状态通知 | **v3 必需**，v4 不需要 |
| `nfs-idmapd` | NFSv4 用户名 ↔ UID/GID 映射 | v4 建议起，不起则属主显示 nobody |
| `nfs` / `nfs-server` | NFS 服务端主程序 | **客户端不需要**，起了也没用 |

> 客户端**不要**启动 `nfs`（nfs-server），那是服务端才需要的。有些机器装了 `nfs-utils` 后 `nfs-server` 会被顺带 enable，可以 `systemctl disable --now nfs-server` 关掉。

**3. 检查**

```bash
which mount.nfs showmount rpcinfo    # 命令都在说明包装好了
systemctl is-active rpcbind          # 用 v3 时检查
```

## 第二步：连接到服务端的 NFS 共享

**1. 先确认网络通**

```bash
ping -c 3 192.168.1.100
nc -zv -w 5 192.168.1.100 2049       # 2049 必须通
```

```
Connection to 192.168.1.100 2049 port [tcp/nfs] succeeded!
```

**2. 查服务端共享了哪些目录**

```bash
showmount -e 192.168.1.100
```

```
Export list for 192.168.1.100:
/data/share 192.168.1.0/24
```

> 如果服务端只用 NFSv4 并关掉了 rpcbind，`showmount` 会失败，这是正常的，直接下一步挂载即可。

**3. 创建挂载点并挂载**

```bash
mkdir -p /mnt/nfs

# 最简形式（自动协商版本）
mount -t nfs 192.168.1.100:/data/share /mnt/nfs

# 指定 NFSv4.2（生产推荐）
mount -t nfs -o vers=4.2 192.168.1.100:/data/share /mnt/nfs

# 指定 NFSv3
mount -t nfs -o vers=3,proto=tcp,nolock 192.168.1.100:/data/share /mnt/nfs

# 云 NAS 推荐的完整参数
mount -t nfs -o vers=3,nolock,proto=tcp,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport \
      192.168.1.100:/data/share /mnt/nfs
```

**4. 检查挂载是否成功**

```bash
df -hT /mnt/nfs
findmnt /mnt/nfs
nfsstat -m                 # 查看实际协商到的版本和参数，比 mount 输出更准
```

```
/mnt/nfs from 192.168.1.100:/data/share
 Flags: rw,vers=4.2,rsize=1048576,wsize=1048576,hard,proto=tcp,timeo=600,retrans=2,sec=sys,...
```

**5. 读写测试**

```bash
echo "hello nfs" > /mnt/nfs/test.txt
cat /mnt/nfs/test.txt
ls -ln /mnt/nfs/test.txt     # -n 看数字属主，判断 squash 是否按预期生效
rm -f /mnt/nfs/test.txt
```

**6. 常用挂载参数**

| 参数 | 作用 |
| --- | --- |
| `vers=3` / `vers=4` / `vers=4.2` | 指定 NFS 版本（`nfsvers=` 同义） |
| `proto=tcp` | 传输协议，建议 tcp |
| `rsize=1048576` / `wsize=1048576` | 读写块大小，建议用满 1MiB |
| `hard` | 硬挂载（默认），服务端无响应时无限重试，不丢数据。**推荐** |
| `soft` | 软挂载，超时后向应用返回错误，可能丢数据 |
| `timeo=600` | 超时时间，单位 0.1 秒（600 = 60 秒） |
| `retrans=2` | 超时后重试次数 |
| `nolock` | 禁用文件锁（v3）。单客户端提速；多客户端写同一文件时不要用 |
| `noresvport` | 断网重连时使用新的源端口。**云 NAS 必加** |
| `ro` / `rw` | 只读 / 读写挂载 |
| `_netdev` | 声明为网络设备，fstab 里必加，等网络就绪再挂 |
| `nofail` | 服务端不可达时不阻塞开机 |

**7. 卸载**

```bash
umount /mnt/nfs

# 提示 target is busy 时，先看谁占用
lsof +D /mnt/nfs
fuser -vm /mnt/nfs
cd / && umount /mnt/nfs

# 服务端已宕机导致卸载不掉（Stale file handle）
umount -l /mnt/nfs           # 懒卸载
```

## 第三步：做持久化（开机自动挂载）

**1. 写入 /etc/fstab**

```bash
cp /etc/fstab /etc/fstab.bak
vi /etc/fstab
```

```
# 基本写法
192.168.1.100:/data/share  /mnt/nfs  nfs  defaults,_netdev  0 0

# 生产推荐（NFSv4 + 完整参数）
192.168.1.100:/data/share  /mnt/nfs  nfs  vers=4.2,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport,_netdev  0 0

# NFSv3
192.168.1.100:/data/share  /mnt/nfs  nfs  vers=3,nolock,proto=tcp,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport,_netdev  0 0

# 服务端可能不可达时，避免卡开机（交给 systemd 按需挂载）
192.168.1.100:/data/share  /mnt/nfs  nfs  defaults,_netdev,noauto,x-systemd.automount,x-systemd.mount-timeout=30  0 0
```

六个字段含义：`设备  挂载点  文件系统类型  挂载选项  dump备份(填0)  fsck检查顺序(填0)`

> ⚠️ `_netdev` 必须加，否则开机会在网络就绪前尝试挂载而卡住；服务端不稳定时再加 `nofail` 或 `x-systemd.automount`。

**2. 校验 fstab（必做，写错下次开机进应急模式）**

```bash
umount /mnt/nfs
mount -a                     # 无报错说明语法正确
df -hT /mnt/nfs              # 能看到条目说明挂载成功
findmnt /mnt/nfs
```

**3. 用 v3 时确保 rpcbind 自启**

```bash
systemctl enable rpcbind nfs-lock
systemctl enable remote-fs.target
```

**4. 验证持久化真正生效**

```bash
reboot
# 重启后确认
df -hT /mnt/nfs
findmnt /mnt/nfs
touch /mnt/nfs/reboot_test && ls -ln /mnt/nfs/reboot_test && rm -f /mnt/nfs/reboot_test
```

**5.（可选）客户端权限对齐**

如果服务端用了 `all_squash,anonuid=1001` 或默认 `root_squash`，要让普通用户正常读写：

```bash
# 客户端创建与服务端相同 UID/GID 的用户
groupadd -g 1001 nfsgroup
useradd  -u 1001 -g 1001 nfsuser
id nfsuser                   # 输出必须与服务端完全一致

# NFSv4 还要保证两端域名一致，否则文件属主显示 nobody
vi /etc/idmapd.conf          # [General] 段 Domain = example.com
systemctl restart nfs-idmapd
```

## 客户端完整流程（可直接复制）

```bash
# 1. 装包
dnf install -y nfs-utils                 # Ubuntu: apt install -y nfs-common
rpm -qa | grep nfs                       # 确认装好

# 2. 服务（v3 需要 rpcbind + nfs-lock；v4 建议起 nfs-idmapd）
systemctl start rpcbind nfs-lock nfs-idmapd
systemctl enable rpcbind nfs-lock nfs-idmapd

# 3. 连通性与共享探测
ping -c 2 192.168.1.100
nc -zv -w 5 192.168.1.100 2049
showmount -e 192.168.1.100

# 4. 挂载
mkdir -p /mnt/nfs
mount -t nfs -o vers=4.2 192.168.1.100:/data/share /mnt/nfs

# 5. 检查
df -hT /mnt/nfs && nfsstat -m
echo test > /mnt/nfs/t.txt && cat /mnt/nfs/t.txt && rm -f /mnt/nfs/t.txt

# 6. 持久化
cp /etc/fstab /etc/fstab.bak
echo '192.168.1.100:/data/share  /mnt/nfs  nfs  vers=4.2,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport,_netdev  0 0' >> /etc/fstab
umount /mnt/nfs && mount -a && df -hT /mnt/nfs

# 7. 重启验证
reboot && df -hT /mnt/nfs
```

---

# 三、两端持久化对照

| 项目 | 服务端 | 客户端 |
| --- | --- | --- |
| 配置写在哪 | `/etc/exports` | `/etc/fstab` |
| 立即生效命令 | `exportfs -rav` | `mount -a` |
| 服务自启 | `systemctl enable rpcbind nfs-lock nfs` | `systemctl enable rpcbind nfs-lock`（v3）/ `nfs-idmapd`（v4） |
| 服务名对应关系 | `nfs` → `nfs-server.service`，`nfs-lock` → `rpc-statd.service` | 同左（别名一致） |
| 防火墙持久化 | `firewall-cmd --permanent ... && --reload` | 一般无需配置 |
| SELinux 持久化 | `setsebool -P nfs_export_all_rw 1` | `setsebool -P use_nfs_home_dirs 1`（用 NFS 家目录时） |
| 验证方式 | `reboot` 后 `exportfs -v` 仍在 | `reboot` 后 `df -hT` 已挂载 |

---

# 四、最容易出问题的五个点

| 现象 | 原因 | 处置 |
| --- | --- | --- |
| `access denied by server` | exports 里客户端网段不对；或括号前多打了空格；或改完没执行 `exportfs -rav` | 服务端 `exportfs -v` 核对，去掉空格后重新 `-rav` |
| `Connection timed out` | 防火墙/安全组没放行 2049（v3 还需 111、20048） | `nc -zv 服务端 2049` 定位，放行端口 |
| 挂上了但写入 `Permission denied` | 文件系统层权限不足，或 `root_squash` 把 root 降级成了 nobody | 服务端 `chown`/`chmod` 匹配映射后的 UID；或改用 `all_squash,anonuid=` |
| 挂上了但是只读 | exports 漏写 `rw`（默认就是 `ro`） | 补上 `rw`，`exportfs -rav` |
| 文件属主显示 `nobody` | NFSv4 两端 `/etc/idmapd.conf` 的 Domain 不一致 | 统一 Domain，`systemctl restart nfs-idmapd` |

