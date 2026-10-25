# LVM 命令手册：传统 LVM 与精简 LVM

分三部分：**第一部分**传统 LVM 全部命令与参数，**第二部分**精简 LVM 全部命令与参数，**第三部分**两者对比。

示例统一命名：VG=`vg0`、卷=`lv_data`、精简池=`pool0`、磁盘=`/dev/sdb`。

---

# 第一部分　传统 LVM（thick / linear）

传统逻辑卷在创建时立即从卷组划走等量物理空间（PE，默认 4MiB），空间预分配、不可超分。层次结构为 **磁盘 → PV → VG → LV → 文件系统**。

## 1.1 命令总览

| 命令 | 作用 |
| --- | --- |
| `pvcreate` | 磁盘/分区 → 物理卷 |
| `vgcreate` | PV → 卷组 |
| `vgextend` | 向 VG 添加 PV |
| `pvresize` | 底层盘变大后刷新 PV 容量 |
| `lvcreate` | 创建逻辑卷 / 快照 |
| `mkfs.xfs` / `mkfs.ext4` | 格式化 |
| `mount` | 挂载 |
| `lvextend` / `lvresize` | 扩容 |
| `xfs_growfs` / `resize2fs` | 扩文件系统 |
| `lvreduce` | 缩容 |
| `lvconvert --merge` | 快照回滚 |
| `lvchange` / `vgchange` | 激活、去激活、改属性 |
| `lvs` / `vgs` / `pvs` / `*display` | 查看 |
| `pvmove` | 迁移 extent |
| `lvremove` / `vgreduce` / `pvremove` / `vgremove` | 删除 |

## 1.2 pvcreate

```bash
pvcreate /dev/sdb                    # 整盘
pvcreate /dev/sdb1 /dev/sdc1         # 多个分区一次创建
pvcreate -ff -y /dev/sdb             # 强制覆盖已有签名
pvcreate --metadatasize 250k /dev/sdb
```

| 参数 | 含义 |
| --- | --- |
| `-ff` | 强制，覆盖已有 LVM 标签 |
| `-y` | 不提示确认 |
| `-u <uuid>` | 指定 PV 的 UUID |
| `--metadatasize <size>` | 元数据区大小 |
| `--metadatacopies 0\|1\|2` | 元数据副本数 |
| `-t` / `--test` | 演练模式 |

## 1.3 vgcreate

```bash
vgcreate vg0 /dev/sdb /dev/sdc
vgcreate -s 16M vg0 /dev/sdb         # 大容量存储建议加大 PE
vgcreate -s 4M -p 10 -l 20 vg0 /dev/sdb
```

| 参数 | 含义 |
| --- | --- |
| `-s <size>` | PE 大小，默认 4M，可选 4M/8M/16M…；PE 越大可管理容量越大 |
| `-p <n>` | VG 内最多 PV 数 |
| `-l <n>` | VG 内最多 LV 数 |
| `-A y\|n` | 是否自动备份元数据 |

## 1.4 vgextend / pvresize / pvmove / vgreduce

```bash
# 加盘扩 VG
pvcreate /dev/sdd
vgextend vg0 /dev/sdd

# 云盘在控制台扩容后（100G→200G）
echo 1 > /sys/class/block/sdb/device/rescan
growpart /dev/sdb 1            # 若用了分区
pvresize --test /dev/sdb1      # 先演练
pvresize /dev/sdb1

# 把某块盘的数据迁走后再移出 VG
pvmove /dev/sdb
vgreduce vg0 /dev/sdb
```

| 命令与参数 | 含义 |
| --- | --- |
| `vgextend VG PV...` | 添加一个或多个 PV |
| `pvresize [--test] PV` | 让 LVM 识别底层设备的新容量 |
| `pvmove [-n] PV_src [PV_dst]` | 在线迁移 extent；`-n` 仅迁移指定 LV |
| `vgreduce VG PV` | 从 VG 移除 PV（需该 PV 已无数据） |
| `vgreduce --removemissing VG` | 清除已损坏/掉线的 PV 记录 |

## 1.5 lvcreate —— 创建传统逻辑卷

```bash
lvcreate -L 100G -n lv_data vg0                 # 固定大小
lvcreate -l 100%FREE -n lv_data vg0             # 用尽 VG 剩余空间
lvcreate -l 50%FREE -n lv_data vg0              # 用剩余空间的一半
lvcreate -l 25 -n lv_data vg0                   # 按 PE 个数（25×4M=100M）
lvcreate -L 100G -n lv_data vg0 /dev/sdb        # 指定落在哪块 PV
lvcreate -L 100G -n lv_data -i 2 -I 64K vg0     # 条带化，跨 2 个 PV
lvcreate --type raid1 -m 1 -L 100G -n lv_data vg0   # RAID1 镜像
lvcreate -L 100G -n lv_data -an vg0             # 创建后不激活
```

| 参数 | 含义 |
| --- | --- |
| `-L <size>` | 卷大小，**立即占用 VG 物理空间** |
| `-l <n>[%FREE\|%VG\|%PVS\|%ORIGIN]` | 按 PE 数或百分比 |
| `-n <name>` | 卷名 |
| `-i <n>` | 条带数（跨几个 PV） |
| `-I <size>` | 条带大小，如 `64K` |
| `-m <n>` | 镜像副本数（配合 `--type raid1`/`mirror`） |
| `--type` | `linear`(默认)/`striped`/`raid1`/`raid5`/`raid6`/`raid10`/`mirror` |
| `-C y\|n` | 是否要求 extent 连续分配 |
| `-p r\|w\|rw` | 读写权限 |
| `-M y` | 固定主次设备号 |
| `-an` / `-ay` | 创建后不激活 / 激活 |
| `-Z y\|n` | 是否将前 4KiB 清零 |
| `VG [PV...]` | 末尾可指定候选 PV |

空间不足时的报错（传统模式不能超分）：

```
Insufficient free space: 25600 extents needed, 15359 free
```

## 1.6 格式化与挂载

```bash
mkfs.xfs /dev/vg0/lv_data
mkfs.ext4 /dev/vg0/lv_data
mkdir -p /data
mount /dev/vg0/lv_data /data
df -hT /data

# 持久化（推荐 UUID）
blkid /dev/vg0/lv_data
echo 'UUID=3f2a...-c91d  /data  xfs  defaults,noatime  0 0' >> /etc/fstab
mount -a
```

| 挂载/文件系统参数 | 含义 |
| --- | --- |
| `defaults` | 常规挂载 |
| `noatime` | 不更新访问时间，提升性能 |
| `nofail` | 设备缺失时不阻塞开机 |
| `ro,nouuid` | 挂载快照时用（XFS 快照 UUID 与原卷相同，必须加 `nouuid`） |

## 1.7 lvextend —— 扩容

```bash
# 一键：扩 LV 同时扩文件系统（推荐）
lvextend -r -L +50G /dev/vg0/lv_data
lvextend -r -L 200G /dev/vg0/lv_data          # 扩到绝对大小
lvextend -r -l +100%FREE /dev/vg0/lv_data     # 用尽 VG 剩余
lvextend -r -l +50%FREE /dev/vg0/lv_data

# 只扩 LV，之后手动扩文件系统
lvextend -L +50G /dev/vg0/lv_data
xfs_growfs /data                 # XFS：传【挂载点】，仅支持在线扩
resize2fs /dev/vg0/lv_data       # ext4：传【设备路径】

# 指定从哪块 PV 取空间
lvextend -L +50G /dev/vg0/lv_data /dev/sdb

# 演练
lvextend -t -L +50G /dev/vg0/lv_data
```

| 参数 | 含义 |
| --- | --- |
| `-L [+]<size>` | 加号=增量，无加号=绝对大小 |
| `-l [+]<n>[%FREE\|%VG\|%PVS]` | 按 PE 数或百分比 |
| `-r` / `--resizefs` | 同时调整文件系统，自动识别 XFS/ext4 |
| `-i <n>` / `-I <size>` | 新增段时指定条带数/条带大小 |
| `-t` / `--test` | 演练 |
| `-f` | 跳过确认 |
| 末尾 `PV...` | 限定从指定 PV 分配 |

扩容四层顺序（缺一不可）：**PV → VG → LV → 文件系统**。`df` 看不到新空间，一定是漏了最后一步。

## 1.8 lvreduce / lvresize —— 缩容（仅 ext4）

```bash
umount /data
e2fsck -f /dev/vg0/lv_data
resize2fs /dev/vg0/lv_data 80G        # ① 先缩文件系统
lvreduce -L 80G /dev/vg0/lv_data      # ② 再缩 LV

# 或用 lvresize -r 一步完成，更安全
umount /data && e2fsck -f /dev/vg0/lv_data
lvresize -r -L 80G /dev/vg0/lv_data
```

| 参数 | 含义 |
| --- | --- |
| `-L <size>` | 缩到绝对大小；`-L -20G` 表示减少 20G |
| `-l <n>` | 按 PE 数 |
| `-r` | 自动先缩文件系统 |
| `-f` | 跳过确认 |

> ⚠️ XFS 不支持缩容，只能备份后重建。顺序颠倒（先缩 LV）会直接损坏数据。

## 1.9 lvcreate -s —— 传统快照（COW）

```bash
lvcreate -L 20G -s -n lv_data_snap /dev/vg0/lv_data
lvs -o lv_name,lv_size,snap_percent,origin vg0
lvextend -L +10G /dev/vg0/lv_data_snap            # 在线扩 COW 空间
mkdir -p /mnt/snap
mount -o ro,nouuid /dev/vg0/lv_data_snap /mnt/snap
umount /mnt/snap
lvconvert --merge /dev/vg0/lv_data_snap            # 回滚，合并后快照自动消失
lvremove -y /dev/vg0/lv_data_snap                  # 只删快照，不回滚
```

| 参数 | 含义 |
| --- | --- |
| `-s` / `--snapshot` | 创建快照 |
| `-L <size>` | COW 空间大小，**必填**；写满则快照失效，原卷可能变只读 |
| `-n <name>` | 快照名 |
| `-c <size>` / `--chunksize` | COW 块大小，默认 4KiB 起 |
| `--type snapshot` | 显式声明类型 |

`lvconvert --merge` 的参数：`--mergethin`（精简快照专用）· `--merge`（通用）· 合并要求原卷与快照均处于未激活状态。

## 1.10 lvchange / vgchange

```bash
lvchange -ay vg0/lv_data
lvchange -an vg0/lv_data          # 需先 umount
vgchange -ay vg0
vgchange -an vg0
lvchange --persistent y -M y -m 253 -M 5 /dev/vg0/lv_data
```

| 参数 | 含义 |
| --- | --- |
| `-ay` / `-an` | 激活 / 去激活 |
| `-aey` | 仅本地激活（共享环境用） |
| `-p r\|w\|rw` | 改读写权限 |
| `-M y --major N --minor N` | 固定主次设备号 |
| `--refresh` | 刷新映射（内核态重载） |
| `--addtag <tag>` / `--deltag <tag>` | 打标签 |
| `-P` | 部分激活（PV 缺失时用） |

## 1.11 查看命令

```bash
pvs;  vgs;  lvs
pvs -o pv_name,vg_name,pv_size,pv_free
vgs -o vg_name,vg_size,vg_free,vg_extent_size,lv_count
lvs -o lv_name,lv_size,segtype,lv_path,devices,lv_attr
pvdisplay -m /dev/sdb          # 查看 extent 分布
vgdisplay vg0
lvdisplay /dev/vg0/lv_data
lsblk -f
findmnt -T /data -o TARGET,SOURCE,FSTYPE
```

| 通用参数 | 含义 |
| --- | --- |
| `-o <字段列表>` | 自定义输出字段 |
| `-a` | 显示隐藏/内部卷 |
| `--noheadings` | 无表头，便于脚本处理 |
| `-S <表达式>` | 过滤，如 `-S lv_size>10g` |
| `--separator ':'` | 指定分隔符 |

常用字段：`pv_name pv_size pv_free vg_name vg_size vg_free vg_extent_size lv_name lv_size segtype lv_attr lv_path devices origin snap_percent copy_percent`

## 1.12 删除

```bash
umount /data
sed -i '\|/dev/mapper/vg0-lv_data|d' /etc/fstab    # 删 fstab 行，否则开机进应急模式
lvchange -an vg0/lv_data
lvremove -y /dev/vg0/lv_data
lvremove -y vg0/lv_data_snap        # 快照需先删，否则 lvremove 卷会连带处理

# 彻底释放磁盘
pvmove /dev/sdb
vgreduce vg0 /dev/sdb
pvremove /dev/sdb
vgremove -y vg0                     # 要求 VG 内已无 LV
```

| 参数 | 含义 |
| --- | --- |
| `lvremove -y LV` | 跳过确认直接删 |
| `vgremove -y VG` | 删卷组 |
| `pvremove -ff PV` | 强制抹标签 |

## 1.13 传统 LVM 完整流程

```bash
# ── 创建 ──
pvcreate /dev/sdb /dev/sdc
vgcreate -s 16M vg0 /dev/sdb /dev/sdc
vgs vg0
lvcreate -L 100G -n lv_data vg0
lvs vg0

# ── 使用 ──
mkfs.xfs /dev/vg0/lv_data
mkdir -p /data
mount /dev/vg0/lv_data /data
UUID=$(blkid -s UUID -o value /dev/vg0/lv_data)
echo "UUID=$UUID  /data  xfs  defaults,noatime  0 0" >> /etc/fstab
df -hT /data

# ── 扩容（VG 有余量） ──
lvextend -r -L +50G /dev/vg0/lv_data
df -hT /data

# ── 扩容（VG 不足，加盘） ──
pvcreate /dev/sdd
vgextend vg0 /dev/sdd
lvextend -r -l +100%FREE /dev/vg0/lv_data

# ── 缩容（仅 ext4） ──
# umount /data && e2fsck -f /dev/vg0/lv_data && resize2fs /dev/vg0/lv_data 80G && lvreduce -L 80G /dev/vg0/lv_data

# ── 快照 ──
lvcreate -L 20G -s -n lv_data_snap /dev/vg0/lv_data
mount -o ro,nouuid /dev/vg0/lv_data_snap /mnt/snap
ls /mnt/snap && umount /mnt/snap
lvs -o lv_name,snap_percent vg0
# lvconvert --merge /dev/vg0/lv_data_snap     # 需要回滚时

# ── 删除 ──
lvremove -y /dev/vg0/lv_data_snap
umount /data
sed -i "\|$UUID|d" /etc/fstab
lvchange -an vg0/lv_data
lvremove -y /dev/vg0/lv_data
vgremove -y vg0
pvremove /dev/sdb /dev/sdc
```

---

# 第二部分　精简 LVM（thin provisioning）

精简 LVM 先建一个**精简池**（thin pool）作为物理空间容器，再从池中切出**精简卷**（thin LV）。精简卷的大小只是虚拟声明，物理块在首次写入时按 chunk 粒度分配，因此可以超分。

精简池由三部分构成，用 `lvs -a` 才看得到：

| 组件 | 名称 | 作用 |
| --- | --- | --- |
| 数据区 | `pool0_tdata` | 存放精简卷的实际数据块 |
| 元数据区 | `pool0_tmeta` | 存放 dm-thin 块映射表，**写满则整池不可用** |
| 备用元数据 | `lvol0_pmspare` | VG 内首个池创建时自动生成，用于修复 |

```bash
lvs -a -o lv_name,lv_size,segtype,data_percent,metadata_percent,chunksize vg0
```

```
  LV               LSize   Type      Data%  Meta%  Chunk
  pool0            100.00g thin-pool  0.00   0.36   64.00k
  [pool0_tdata]    100.00g linear
  [pool0_tmeta]      1.00g linear
  [lvol0_pmspare]    1.00g linear
```

## 2.1 命令总览

| 命令 | 作用 |
| --- | --- |
| `pvcreate` / `vgcreate` / `vgextend` / `pvresize` | 与传统 LVM 完全相同 |
| `lvcreate --thinpool` | 创建精简池 |
| `lvcreate --type thin-pool` | 创建精简池（新版推荐写法） |
| `lvconvert --type thin-pool` | 把已有 LV 或 data+meta 组合转成池 |
| `lvcreate -V --thin` | 创建精简卷 |
| `lvextend -L` | 扩精简卷虚拟大小 / 扩池数据区 |
| `lvextend --poolmetadatasize` | 扩池元数据区 |
| `lvextend --use-policies` | 按 lvm.conf 策略自动扩池 |
| `lvcreate -s` | 精简快照（无需指定大小） |
| `lvchange -K` / `--setactivationskip` | 处理精简快照的激活跳过属性 |
| `lvchange --errorwhenfull` | 池满行为控制 |
| `lvchange --discards` / `fstrim` | 空间回收 |
| `lvs -o data_percent,metadata_percent` | 池占用监控 |
| `lvremove` | 删精简卷 / 删池 |

## 2.2 lvcreate --thinpool —— 创建精简池

```bash
# 常规：一步创建
lvcreate -L 100G --thinpool pool0 vg0
lvcreate --type thin-pool -L 100G --thinpool pool0 vg0    # 新版推荐
lvcreate -l 100%FREE --thinpool pool0 vg0                 # 用尽 VG
lvcreate -L 100G -T vg0/pool0                             # -T 简写（配 -L 即建池）

# 带完整参数
lvcreate -L 200G --thinpool pool0 \
         --chunksize 128K \
         --poolmetadatasize 2G \
         --zero y \
         --errorwhenfull y \
         --discards passdown \
         --poolmetadataspare y \
         vg0
```

```
  Thin pool volume with chunk size 64.00 KiB can address at most 15.88 TiB of data.
  Logical volume "pool0" created.
```

| 参数 | 含义与建议 |
| --- | --- |
| `-L <size>` | **池的物理容量**，不是虚拟容量 |
| `--thinpool <name>` | 声明池名 |
| `--type thin-pool` | 显式声明类型（新版推荐，与 `--thinpool` 连用） |
| `-T <VG/池名>` | 简写形式，配 `-L` 建池 |
| `-c` / `--chunksize <64K~1G>` | 分配粒度，须为 2 的幂，默认 64K。顺序写/数据库类建议 256K~1M；快照密集、随机小 IO 用 64K |
| `--poolmetadatasize <size>` | 元数据区大小。经验值：每 1TiB 数据区 12~24MiB，每 100 个精简卷再加约 1MiB；实际建议直接给 512M~2G |
| `--zero y\|n` | 新分配块是否清零，默认 `y`（安全）；`n` 提升首次写性能但可能读到旧数据 |
| `--errorwhenfull y\|n` | 池满行为。`y`=写入报错（默认，保数据）；`n`=写入挂起排队（等扩容，但有 IO 卡死风险） |
| `--discards passdown\|nopassdown\|ignore` | 是否透传上层 TRIM，空间回收需 `passdown` |
| `--poolmetadataspare y\|n` | 是否创建备用元数据卷 `pmspare`，默认 `y` |
| `-i` / `-I` | 数据区条带化 |
| `-l <n>%FREE` | 按 VG 剩余百分比建池 |

## 2.3 lvconvert —— 组合或转换精简池

**data 与 metadata 分离创建（生产推荐，可分别做 RAID 或放不同盘）**

```bash
lvcreate -L 500G -n pool0_data vg0 /dev/sdb
lvcreate -L 2G   -n pool0_meta vg0 /dev/sdc
lvconvert --type thin-pool --poolmetadata vg0/pool0_meta vg0/pool0_data
```

```
  WARNING: Converting vg0/pool0_data and vg0/pool0_meta to thin pool's data and metadata volumes with metadata wiping.
  THIS WILL DESTROY CONTENT OF LOGICAL VOLUME (filesystem etc.)
Do you really want to convert vg0/pool0_data and vg0/pool0_meta? [y/n]: y
  Converted vg0/pool0_data to thin pool.
```

**带 RAID1 的高可用版本**

```bash
lvcreate --type raid1 -m 1 -L 2G   -n pool0_meta vg0 /dev/sdc /dev/sdd
lvcreate --type raid1 -m 1 -L 500G -n pool0_data vg0 /dev/sdb /dev/sde
lvconvert --type thin-pool --poolmetadata vg0/pool0_meta vg0/pool0_data
```

**把已有传统 LV 转成池**（卷内数据会被清空）

```bash
lvconvert --type thin-pool vg0/lv_old
lvconvert --type thin-pool --poolmetadata vg0/pool0_meta vg0/lv_old
```

**先建普通卷再转池（Proxmox 常用）**

```bash
lvcreate -L 100G -n data vg0
lvconvert --type thin-pool vg0/data
```

| 参数 | 含义 |
| --- | --- |
| `--type thin-pool` | 转换为精简池 |
| `--poolmetadata <VG/LV>` | 指定用作元数据区的 LV |
| `-c` / `--chunksize` | 转换时指定 chunk |
| `--poolmetadatasize` | 转换时指定元数据区大小 |
| `-y` | 跳过销毁数据的确认 |

> 合并后池名沿用数据区 LV 的名字。

## 2.4 lvcreate -V --thin —— 创建精简卷

```bash
lvcreate -V 200G --thin -n lv_data vg0/pool0
lvcreate -V 5GiB -T vg0/pool0 -n thin1              # -T 后直接跟 VG/池名
lvcreate --thinpool pool0 -V 5GiB vg0 -n thin1      # 新版推荐写法
lvcreate -y -V 200G --thin -n vm01 vg0/pool0        # 超分时跳过确认
lvcreate -V 50G --thin -n lv_data -an vg0/pool0     # 创建但不激活
lvcreate -n vz -V 10G vg0/pool0                     # 池路径可省略 --thin
```

| 参数 | 含义 |
| --- | --- |
| `-V <size>` / `--virtualsize` | **虚拟大小**，可远超池物理容量 |
| `--thin` | 声明为精简卷（纯开关，不带参数） |
| `-T <VG/池名>` | 指定源池（带参数） |
| `--thinpool <池名> <VG>` | 新版写法，指定源池 |
| `-n <name>` | 卷名 |
| `-y` | 超分时不再提示 |
| `-an` | 创建后不激活（批量预制虚拟机磁盘常用） |
| `-kn` | 创建后仅保留在内存中不建映射（少见） |

**超分演示**——池只有 100G，可创建虚拟总量 300G 的卷：

```bash
lvcreate -y -V 100G --thin -n vm01 vg0/pool0
lvcreate -y -V 100G --thin -n vm02 vg0/pool0
lvcreate -y -V 100G --thin -n vm03 vg0/pool0
```

超分时的告警回显：

```
WARNING: Sum of all thin volume sizes (300.00 GiB) exceeds the size of thin pool vg0/pool0 and the size of whole volume group (100.00 GiB).
WARNING: You have not turned on protection against thin pools running out of space.
WARNING: Set activation/thin_pool_autoextend_threshold below 100 to trigger automatic extension of thin pools before they get full.
  Logical volume "vm03" created.
```

**一条命令同时建池 + 建卷**

```bash
lvcreate --type thin -L 100G -V 5GiB --thinpool pool0 -n thin1 vg0
```

### -T 的三种形态（最易混淆处）

`-T` 既能建池也能建卷，**取决于搭配 `-L` 还是 `-V`**：

| 写法 | 搭配 | 结果 |
| --- | --- | --- |
| `lvcreate -L 100G -T vg0/pool0` | `-L` 物理大小 | 建**池** |
| `lvcreate -V 5GiB -T vg0/pool0 -n thin1` | `-V` 虚拟大小 | 建**精简卷** |
| `lvcreate --type thin -L 100G -V 5GiB --thinpool pool0 -n thin1 vg0` | `-L` + `-V` | **同时建池 + 建卷** |

> `--thin` 是不带参数的开关，`-T` 带参数——手册中是两个不同定义。LXC 默认存储用的就是 `lvcreate -T vg0/lxcfpool -V 5GiB -n thin1` 这种形式。

## 2.5 格式化与挂载使用

精简卷创建完成后，上层用法与传统 LV **完全一致**：

```bash
mkfs.xfs /dev/vg0/lv_data
mkdir -p /data
mount -o discard /dev/vg0/lv_data /data
echo "UUID=$(blkid -s UUID -o value /dev/vg0/lv_data)  /data  xfs  defaults,noatime,discard,nofail  0 0" >> /etc/fstab
```

激活（激活卷会连带激活其所属池）：

```bash
lvchange -ay vg0/lv_data
lvchange -ay vg0/pool0
vgchange -ay vg0
```

## 2.6 lvextend —— 精简扩容（三个对象别搞混）

| 扩容对象 | 命令 | 是否消耗 VG 物理空间 |
| --- | --- | --- |
| 精简卷的虚拟大小 | `lvextend -L +50G vg0/lv_data` | **否**，只改声明 |
| 精简池的数据区 | `lvextend -L +50G vg0/pool0` | 是 |
| 精简池的元数据区 | `lvextend --poolmetadatasize +200M vg0/pool0` | 是 |

```bash
# ① 扩精简卷（虚拟大小 + 文件系统）
lvextend -r -L +50G /dev/vg0/lv_data
lvextend -r -L 200G /dev/vg0/lv_data
lvextend -L +50G /dev/vg0/lv_data && xfs_growfs /data     # 手动两步

# ② 扩池数据区
lvextend -L +50G vg0/pool0
lvextend -l +100%FREE vg0/pool0
lvextend -L +50G vg0/pool0_tdata                          # 直接操作隐藏子卷

# ③ 扩池元数据区
lvextend --poolmetadatasize +200M vg0/pool0
lvconvert --poolmetadatasize +200M vg0/pool0              # 等价写法
lvextend --poolmetadatasize +200M vg0/pool0_tmeta

# ④ 数据区 + 元数据区一起扩
lvextend -L +50G --poolmetadatasize +200M vg0/pool0

# ⑤ 按策略自动扩
lvextend --use-policies vg0/pool0
```

| 参数 | 含义 |
| --- | --- |
| `-L [+]<size>` | 作用于卷=扩虚拟大小；作用于池=扩数据区物理容量 |
| `-r` | 同时扩文件系统 |
| `--poolmetadatasize [+]<size>` | **精简专属**，扩元数据区 |
| `--use-policies` | **精简专属**，按 lvm.conf 阈值扩池 |
| `-l +100%FREE` | 用尽 VG 剩余空间扩池 |

> 扩池的前提是 VG 有空闲 PE，不足时先 `pvcreate` + `vgextend` 或 `pvresize`。所以**建池时不要把 VG 空间用尽**，留 20% 以上给自动扩容。

## 2.7 精简池自动扩容

```bash
# /etc/lvm/lvm.conf  →  activation 段
activation {
    thin_pool_autoextend_threshold = 70   # 占用达 70% 触发；最小 50，设为 100 = 禁用
    thin_pool_autoextend_percent   = 20   # 每次扩当前大小的 20%
}

# 启用监控（dmeventd 负责接收内核通知并执行 lvextend --use-policies）
systemctl enable --now lvm2-monitor
systemctl status lvm2-monitor
```

## 2.8 lvcreate -s —— 精简快照

精简快照**不需要 `-L`**，与原卷共享同一个池，只记录差异块；支持快照的快照。

```bash
lvcreate -s -n lv_data_snap vg0/lv_data          # 无大小参数
lvcreate -s -n snap2 vg0/lv_data_snap            # 快照的快照
lvcreate -s -n lv_clone vg0/lv_data              # 可写克隆

# 精简快照默认带 skip activation，普通 -ay 激活不了
lvs -o lv_name,lv_attr,skip_activation vg0/lv_data_snap
lvchange -ay -K vg0/lv_data_snap                 # 用 -K 强制激活
lvchange --setactivationskip n vg0/lv_data_snap  # 或永久取消该属性

# 挂载（只读）
mkdir -p /mnt/snap
mount -o ro,nouuid /dev/vg0/lv_data_snap /mnt/snap
umount /mnt/snap

# 回滚：原卷与快照都需先未激活
umount /data
lvchange -an vg0/lv_data
lvconvert --merge vg0/lv_data_snap
lvchange -ay vg0/lv_data && mount /dev/vg0/lv_data /data

# 外部源快照（把已有传统 LV 当只读源，不占额外空间）
lvcreate --type thin --thinpool pool0 -s -n ext_snap vg0/lv_thick
```

| 参数 | 含义 |
| --- | --- |
| `-s` / `--snapshot` | 源是精简卷时自动建精简快照，**不要加 `-L`** |
| `-n <name>` | 快照名 |
| `--thinpool <池>` | 外部源快照时指定归属池 |
| `--setactivationskip y\|n`（lvchange） | 设置/取消激活跳过属性 |
| `-K` / `--ignoreactivationskip`（lvchange） | 忽略激活跳过属性 |

## 2.9 lvchange —— 精简专属属性

```bash
lvchange --discards passdown vg0/pool0     # 开启 TRIM 透传
lvchange --errorwhenfull n vg0/pool0       # 池满改为挂起（应急）
lvchange --errorwhenfull y vg0/pool0       # 处置完改回报错
lvchange --setactivationskip n vg0/snap    # 取消快照激活跳过
lvchange --monitor y vg0/pool0             # 交给 dmeventd 监控
```

| 参数 | 含义 |
| --- | --- |
| `--discards passdown\|nopassdown\|ignore` | TRIM 是否透传到底层，空间回收需 `passdown` |
| `--errorwhenfull y\|n` | 池满时 `y`=报错保护数据；`n`=挂起等待扩容 |
| `--setactivationskip y\|n` | 快照的激活跳过属性 |
| `-K` | 激活时忽略跳过属性 |
| `--monitor y\|n` | 是否注册 dmeventd 监控 |

## 2.10 空间回收（discard / fstrim）

```bash
# 池启用透传
lvchange --discards passdown vg0/pool0

# 卷挂载时带 discard（自动回收）
mount -o discard /dev/vg0/lv_data /data

# 或定期手动回收
fstrim -v /data
```

```
/data: 12.3 GiB (13212057600 bytes) trimmed
```

> 缩小精简卷的虚拟大小、或删除精简卷后，物理块**不一定立即回池**，需要 `fstrim` 或等待 LVM 回收。

## 2.11 缩精简卷

```bash
# ext4：先缩文件系统再缩 LV，之后 fstrim 把空间还给池
umount /data
e2fsck -f /dev/vg0/lv_data
resize2fs /dev/vg0/lv_data 40G
lvreduce -L 40G /dev/vg0/lv_data
mount /dev/vg0/lv_data /data
fstrim -v /data
```

XFS 不支持缩容。

## 2.12 监控与池满应急

```bash
# 池占用（必须同时看两个百分比）
lvs -o lv_name,data_percent,metadata_percent vg0/pool0
lvs -a -o lv_name,lv_size,segtype,data_percent,metadata_percent,chunksize vg0

# 每个精简卷的实际占用
lvs -o lv_name,lv_size,pool_lv,data_percent,disk_usage vg0

# 池满应急
lvchange --errorwhenfull n vg0/pool0     # 临时改挂起，争取时间
lvextend -L +50G vg0/pool0               # 扩池
lvchange --errorwhenfull y vg0/pool0     # 改回
```

| 指标 | 预警线 | 处置 |
| --- | --- | --- |
| `data_percent` | ≥70% 自动扩；≥80% 人工介入 | `lvextend -L` 扩池，或清数据 + `fstrim` |
| `metadata_percent` | ≥80% 立即处理 | `lvextend --poolmetadatasize` |

## 2.13 删除

```bash
# 删单个精简卷（不影响池和其他卷）
umount /data
lvchange -an vg0/lv_data
lvremove -y vg0/lv_data

# 删快照
lvremove -y vg0/lv_data_snap

# ⚠️ 删整个池：级联删除池内所有精简卷和快照，不可逆
lvs -o lv_name,pool_lv vg0            # 执行前先列出全部子卷确认
lvremove -y vg0/pool0

# 清理 VG / PV
vgremove -y vg0
pvremove /dev/sdb /dev/sdc
```

## 2.14 精简 LVM 完整流程

```bash
# ── 创建 ──
pvcreate /dev/sdb /dev/sdc
vgcreate -s 16M vg0 /dev/sdb /dev/sdc
lvcreate -L 100G --thinpool pool0 --chunksize 128K --poolmetadatasize 1G \
         --zero y --errorwhenfull y --discards passdown vg0
lvs -a -o lv_name,lv_size,segtype,data_percent,metadata_percent vg0

# 建精简卷（虚拟 200G，池只有 100G）
lvcreate -y -V 200G --thin -n lv_data vg0/pool0
lvs -o lv_name,lv_size,pool_lv,data_percent vg0

# ── 使用 ──
mkfs.xfs /dev/vg0/lv_data
mkdir -p /data
mount -o discard /dev/vg0/lv_data /data
UUID=$(blkid -s UUID -o value /dev/vg0/lv_data)
echo "UUID=$UUID  /data  xfs  defaults,noatime,discard,nofail  0 0" >> /etc/fstab
df -hT /data

# ── 写入并观察池占用增长 ──
dd if=/dev/zero of=/data/testfile bs=1M count=2048 oflag=direct
lvs -o lv_name,data_percent,metadata_percent vg0/pool0

# ── 扩容：卷 ──
lvextend -r -L +50G /dev/vg0/lv_data
df -hT /data

# ── 扩容：池数据区 ──
lvextend -L +50G vg0/pool0
vgs vg0 -o vg_name,vg_size,vg_free

# ── 扩容：池元数据区 ──
lvextend --poolmetadatasize +200M vg0/pool0

# ── 自动扩容配置 ──
# /etc/lvm/lvm.conf: thin_pool_autoextend_threshold = 70 / thin_pool_autoextend_percent = 20
systemctl enable --now lvm2-monitor
lvextend --use-policies vg0/pool0

# ── 快照 ──
lvcreate -s -n lv_data_snap vg0/lv_data
lvchange --setactivationskip n vg0/lv_data_snap
lvchange -ay vg0/lv_data_snap
mount -o ro,nouuid /dev/vg0/lv_data_snap /mnt/snap && ls /mnt/snap && umount /mnt/snap

# ── 回滚 ──
umount /data
lvchange -an vg0/lv_data
lvconvert --merge vg0/lv_data_snap
lvchange -ay vg0/lv_data && mount /dev/vg0/lv_data /data

# ── 空间回收 ──
fstrim -v /data

# ── 删除 ──
umount /data
sed -i "\|$UUID|d" /etc/fstab
lvchange -an vg0/lv_data
lvremove -y vg0/lv_data
lvremove -y vg0/pool0        # 级联删除池内剩余所有卷
vgremove -y vg0
pvremove /dev/sdb /dev/sdc
```

---

# 第三部分　传统 LVM 与精简 LVM 对比

## 3.1 模型对比

| 维度 | 传统 LVM | 精简 LVM |
| --- | --- | --- |
| 层次结构 | PV → VG → LV | PV → VG → **精简池** → 精简卷 |
| 空间分配 | 创建即分配，预占物理空间 | 首次写入时按 chunk 分配 |
| 最小分配单位 | PE，默认 4MiB | chunk，默认 64KiB |
| 超分 | 不支持，空间不足即报错 | 支持，虚拟总量可数倍于物理容量 |
| 内部结构 | 单个 LV | data LV + metadata LV + pmspare |
| 空间耗尽后果 | 创建/扩容失败，**已有数据安全** | 池满后写入报错或挂起，**可能损坏池** |
| 快照成本 | 需预留 COW 空间，满了失效 | 与原卷共享池，无需预留 |
| 快照层级 | 一个原卷一个活动快照 | 支持快照的快照，可成链/成树 |
| 性能 | 略高，无映射表开销 | 有映射开销，首次写入需分配块 |
| 缩容后空间 | 立即释放回 VG | 需 `fstrim` 才回池 |
| 运维复杂度 | 低 | 中，须持续监控 Data% / Meta% |
| 典型场景 | 数据库、根分区、要求确定性的业务 | KVM/Proxmox 虚拟化、LXC 容器、多租户、VDI、开发测试 |

## 3.2 命令对比

| 操作 | 传统 LVM | 精简 LVM |
| --- | --- | --- |
| 建 PV/VG | `pvcreate` / `vgcreate` | 完全相同 |
| 建存储池 | 无需（VG 即池） | `lvcreate -L 100G --thinpool pool0 vg0` |
| 转换已有卷 | — | `lvconvert --type thin-pool vg0/lv_old` |
| 建卷 | `lvcreate -L 100G -n lv_data vg0` | `lvcreate -V 100G --thin -n lv_data vg0/pool0` |
| 建卷（-T 形式） | — | `lvcreate -T vg0/pool0 -V 5GiB -n thin1` |
| 池+卷一次建 | — | `lvcreate --type thin -L 100G -V 5GiB --thinpool pool0 -n thin1 vg0` |
| 格式化挂载 | `mkfs.xfs` + `mount` | 完全相同 |
| 扩卷 | `lvextend -r -L +50G vg0/lv_data` | 相同命令 |
| 扩存储 | `vgextend` 后扩 LV | `lvextend -L +50G vg0/pool0` |
| 扩元数据 | 无此概念 | `lvextend --poolmetadatasize +200M vg0/pool0` |
| 自动扩容 | 无 | `thin_pool_autoextend_threshold` + `lvextend --use-policies` |
| 缩卷 | `resize2fs` + `lvreduce` | 相同，之后需 `fstrim` 回收 |
| 建快照 | `lvcreate -L 20G -s -n snap vg0/lv_data` | `lvcreate -s -n snap vg0/lv_data`（无 `-L`） |
| 激活快照 | `lvchange -ay vg0/snap` | `lvchange -ay -K vg0/snap` |
| 快照监控 | `lvs -o snap_percent` | 看池的 `data_percent` |
| 回滚 | `lvconvert --merge vg0/snap` | 相同（原卷需先 `-an`） |
| 使用率查看 | `df -hT` | `df -hT` + `lvs -o data_percent,metadata_percent` |
| 空间回收 | 无需 | `--discards passdown` + `fstrim` |
| 池满应急 | 无此概念 | `lvchange --errorwhenfull n` + `lvextend` |
| 删卷 | `lvremove vg0/lv_data` | 相同 |
| 删池 | 无此概念 | `lvremove vg0/pool0`（**级联删全部子卷**） |
| 查看隐藏卷 | `lvs -a`（一般无内容） | `lvs -a` 可见 `_tdata`/`_tmeta`/`_pmspare` |

## 3.3 关键参数对比

| 参数 | 所属命令 | 传统 | 精简 |
| --- | --- | --- | --- |
| `-L` | `lvcreate` | 卷的实际大小 | 池的物理容量 |
| `-V` | `lvcreate` | 不适用 | 精简卷的虚拟大小 |
| `-T` | `lvcreate` | 不适用 | 配 `-L` 建池 / 配 `-V` 建卷 |
| `--thin` | `lvcreate` | 不适用 | 声明精简卷（不带参数） |
| `--thinpool` | `lvcreate` / `lvconvert` | 不适用 | 池名或源池 |
| `--type` | `lvcreate` / `lvconvert` | `linear`/`striped`/`raid1` | `thin-pool`/`thin` |
| `-c` / `--chunksize` | `lvcreate` | COW 快照块大小 | 池的分配粒度（64K~1G） |
| `-s` | `lvcreate` | COW 快照，需配 `-L` | 精简快照，不配 `-L` |
| `--poolmetadatasize` | `lvcreate` / `lvextend` / `lvconvert` | 不适用 | 元数据区大小 |
| `--use-policies` | `lvextend` | 不适用 | 按策略自动扩池 |
| `--errorwhenfull` | `lvcreate` / `lvchange` | 不适用 | 池满行为 |
| `--discards` | `lvcreate` / `lvchange` | 不适用 | TRIM 透传 |
| `--zero` | `lvcreate` | 前 4KiB 清零 | 新分配块是否清零 |
| `--poolmetadataspare` | `lvcreate` | 不适用 | 是否建 pmspare |
| `-K` / `--setactivationskip` | `lvchange` | 不适用 | 精简快照激活控制 |
| `-r` | `lvextend` / `lvresize` | 通用 | 通用 |

## 3.4 选型建议

**用传统 LVM**：磁盘数量有限、跑数据库、根分区、对性能和容量确定性要求高、不想引入额外监控的场景。

**用精简 LVM**：需要给大量虚拟机/容器/租户分盘、想提高存储利用率、快照需求频繁、开发测试环境、能接受并配好池占用监控的场景。

**混用**：同一 VG 内可并存——数据库走传统卷保证确定性，虚拟机走精简池提高利用率。

## 3.5 通用注意事项

- `pvcreate` 会覆盖磁盘签名，执行前用 `lsblk -f` + `blkid` + `wipefs -n` 三重确认目标盘空闲
- 删卷后必须同步清理 `/etc/fstab`，否则下次开机进应急模式
- XFS 只能扩不能缩；ext4 缩容必须「先缩文件系统、再缩 LV」
- 扩容四层顺序不可颠倒：PV → VG → LV → 文件系统
- 变更前执行 `vgcfgbackup vg0` 备份元数据（自动备份位于 `/etc/lvm/archive/`）
- 精简池的 `metadata_percent` 满会导致整池不可用，是与 `data_percent` 同等重要的监控项

