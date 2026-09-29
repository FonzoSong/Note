# Linux 运维面试常用命令速记手册

> **用途**：Linux 系统运维面试的记忆背诵材料。覆盖面试高频命令 + 可直接口述的使用示例 + 考察点提示。
> **版本说明**：第 1–14 章为命令与排障主线；第 15–20 章为初级岗（0–2 年）专项补充：操作系统/网络理论、Nginx/MySQL/Redis、现场手写脚本、监控容器认知、人题与软技能。
> **标记说明**：★ = 面试必考，必须能做到"说出命令 + 参数 + 用途"三件套；☆ = 高频加分项。
> **背诵方法建议**：先记场景（排查什么问题）→ 再记命令骨架 → 最后记参数。面试官考的是"什么情况下用它"，不是参数手册。

---

## 一、系统信息与硬件查看

| 命令 | 必背示例 | 记忆点 / 考察点 |
|------|---------|----------------|
| ★ `uname` | `uname -a` | 看内核版本、主机名、架构；`-r` 只看内核版本 |
| ★ `cat /etc/os-release` | `cat /etc/os-release` | 看发行版和版本（CentOS/Ubuntu 面试第一步就问） |
| ☆ `hostnamectl` | `hostnamectl` / `hostnamectl set-hostname web01` | 查看/修改主机名，立即生效 |
| ★ `lscpu` | `lscpu` | 核数、架构、CPU 型号；面试常问"怎么看逻辑核数" |
| ★ `free` | `free -m` / `free -h` | used 不等于真实占用！看 **available**；buffer/cache 可回收 |
| ★ `df` | `df -h` / `df -i` | `-h` 看磁盘容量；`-i` 看 inode 使用率（磁盘没满但无法写入 → inode 耗尽） |
| ★ `du` | `du -sh /var/log/*` | 看目录大小，排查"谁把磁盘占满了"；`-s` 只看汇总 |
| ☆ `uptime` | `uptime` | 一行看运行时长 + 1/5/15 分钟负载，面试第一反应命令 |
| ★ `dmesg` | `dmesg -T \| tail -50` | 内核日志：硬件报错、OOM、磁盘掉线都在这；`-T` 人类可读时间 |
| ☆ `lspci` / `lsusb` | `lspci` | 看 PCI 设备（网卡、RAID 卡） |
| ☆ `lsblk` | `lsblk` | 看块设备层级（盘和分区关系），比 fdisk -l 直观 |
| ☆ `dmidecode` | `dmidecode -t memory` / `-t 1` | 看内存条槽位、序列号、是否虚拟机（-t 1 系统信息） |
| ★ `cat /proc/loadavg` | `cat /proc/loadavg` | 负载三值：1/5/15 分钟；与 CPU 核数比较判断压力 |
| ☆ `timedatectl` | `timedatectl set-timezone Asia/Shanghai` | 时区、NTP 同步状态（chronyc sources） |
| ☆ `nproc` | `nproc` | 快速输出 CPU 逻辑核数 |

**面试问答点**：
- 「服务器 load 高怎么看？」→ `uptime` 确认负载 → `top` 定位进程 → 区分是 CPU 高还是 iowait 高。
- 「free 里内存快满了正常吗？」→ 正常，Linux 空闲内存是浪费；重点看 available，cache 在应用需要时会自动回收。

---

## 二、文件与目录基础操作

| 命令 | 必背示例 | 记忆点 / 考察点 |
|------|---------|----------------|
| ★ `ls` | `ls -lht` | `-a` 含隐藏文件、`-l` 详情、`-h` 人类可读、`-t` 按时间排序；组合用排故障 |
| ★ `cd` | `cd -` | 返回上一个目录（面试爱考的冷知识） |
| ★ `cp` | `cp -a /etc /etc_bak` | `-r` 递归；**`-a` 保留权限/时间/软链接**，备份配置必用；`\cp` 绕过 alias 强制覆盖 |
| ★ `mv` / `rm` | `rm -rf dir/` | `rm -i` 逐个确认；`rm -rf` 是高危命令，面试必问"误删了怎么办"（见第九章） |
| ★ `mkdir` | `mkdir -p /a/b/c` | `-p` 递归创建、已存在不报错（脚本里必备） |
| ★ `touch` | `touch file` | 新建空文件 / 更新 mtime；可测磁盘是否可写 |
| ★ `ln` | `ln -s /app/v1/conf nginx.conf` | **软链接独立 inode 可跨分区可链目录；硬链接共享 inode，不能跨分区、不能链目录**（必考） |
| ☆ `find` | 见 3.1 | 单独成章，笔试重灾区 |
| ☆ `tree` | `tree -L 2` | 目录树展示，看项目结构 |
| ★ `stat` | `stat file` | 精确看 atime/mtime/ctime：修改时间是 mtime，改权限属主是 ctime |

---

## 三、文本处理（笔试大题高发区）

### 3.1 find —— 按条件找文件

| 必背示例 | 说明 |
|---------|------|
| `find /var/log -name "*.log"` | 按名字找；`-iname` 忽略大小写 |
| `find /tmp -type f` / `-type d` | 只找文件 / 只找目录 |
| `find / -mtime +30 -type f` | 30 天**之前**修改的（-mtime -1 为 24 小时内，-mmin -10 为 10 分钟内） |
| `find / -size +500M -type f` | 找 500MB 以上大文件（磁盘爆满排查） |
| `find /tmp -name "*.log" -exec rm -f {} \;` | 找到后执行命令；`{}` 代表结果 |
| `find /var/log -name "*.log" -mtime +30 -delete` | 删除 30 天前日志，日志清理标准答案 |
| `find /data -path "*/node_modules" -prune -o -name "*.js" -print` | 排除目录（进阶加分） |

### 3.2 grep —— 按内容找行

| 必背示例 | 说明 |
|---------|------|
| `grep -n "error" app.log` | 显示行号 |
| `grep -ri "timeout" /etc/nginx/` | 递归 + 忽略大小写，全目录搜配置 |
| `grep -v "^#" /etc/profile` | **反向匹配**：过滤注释行，看有效配置 |
| `grep -c "Failed" /var/log/secure` | 统计次数（爆破排查） |
| `grep -E "ERR|WARN" app.log` | 或逻辑，多关键字；等价 `egrep` |
| `grep -C 3 "OOM" /var/log/messages` | 前后 3 行上下文；`-A 3` 后、`-B 3` 前 |
| `grep -wl "token" *.log` | 只列出包含关键字的文件名 |

### 3.3 awk —— 按列取数、统计

| 必背示例 | 说明 |
|---------|------|
| `awk -F: '{print $1}' /etc/passwd` | 指定分隔符取第 1 列（所有账号） |
| `awk '{print $1}' access.log \| sort \| uniq -c \| sort -rn \| head -10` | **笔试模板**：访问日志 Top10 IP |
| `awk '$3>100 {print $1}' cpu.txt` | 条件过滤第 3 列大于 100 的行 |
| `awk '{sum+=$1} END {print sum/NR}' data.txt` | END 块求平均（累计求和思想） |
| `awk 'NR>=5 && NR<=20' file` | 取第 5~20 行（sed 也能做，对比记忆） |
| `ps aux \| awk '{print $1}' \| sort \| uniq -c \| sort -rn` | 按用户统计进程数 |

### 3.4 sed —— 批量改文本（流式编辑）

| 必背示例 | 说明 |
|---------|------|
| `sed -i 's/8080/9090/g' nginx.conf` | **原地**全局替换（脚本改配置必背）；`-i.bak` 自动留备份 |
| `sed -n '10,20p' big.log` | 只看第 10~20 行（`-n` + `p` 才不打印全文） |
| `sed -i '3d' crontab.txt` | 删除第 3 行 |
| `sed -i '2s/^/#/' /etc/fstab` | 给第 2 行行首加 `#` 注释掉 |
| `sed 's#/usr/local#/opt#g' file` | 替换含 `/` 的路径时换分隔符 `#`（加分细节） |

### 3.5 其他必背

| 命令 | 必背示例 | 记忆点 |
|------|---------|--------|
| ★ `sort` | `sort -nr -k 2` | `-n` 数字序（默认字典序，10 会排 2 前——常考坑）、`-r` 倒序、`-k` 按列 |
| ★ `uniq` | `sort \| uniq -c \| sort -rn` | **uniq 必须先 sort**；`-c` 计数，去重统计固定搭配 |
| ★ `wc` | `wc -l` | 行数；`grep -c` 也能计数但统计的是行 |
| ★ `cut` | `cut -d: -f1 /etc/passwd` | 按分隔符切列，比 awk 轻量 |
| ★ `head` / `tail` | `tail -f app.log` | `-f` 实时跟踪日志（必考）；`-F` 文件轮转后继续跟；`tail -n 100` 最后 100 行 |
| ★ `less` | `less +G big.log`、`/keyword` | 大文件分页查看，`?` 反向搜、`G` 末行、`q` 退出；比 cat 安全 |
| ☆ `tr` | `tr 'a-z' 'A-Z'` | 字符转换/删除 `tr -d '\r'`（修 Windows 换行符） |
| ☆ `diff` | `diff -r dir1 dir2` | 配置对比 |
| ☆ `xargs` | `find . -name "*.tmp" \| xargs rm -f` | 把前段输出转成后段参数；文件名带空格用 `xargs -0` |
| ★ `tee` | `echo "hello" \| tee -a /etc/motd` | 一边显示一边写入；脚本里 `cmd \| tee log`；`>>` 追加重定向加分 |

---

## 四、权限、用户与组

| 命令 | 必背示例 | 记忆点 / 考察点 |
|------|---------|----------------|
| ★ `chmod` | `chmod 755 app.sh` / `chmod u+x file` | r=4 w=2 x=1；**755=属主全权、其他人可读可执行；644=普通文件推荐权限**；目录必须带 x 才能进入 |
| ★ `chown` | `chown -R www:www /data/web` | 改属主属组，Web 目录 403 先查它；`-R` 递归 |
| ☆ `chgrp` | `chgrp users file` | 只改属组 |
| ☆ `umask` | `umask` | 新建默认权限掩码：umask 022 → 文件默认 644、目录 755 |
| ★ `su` / `sudo` | `su - user` / `sudo -u user cmd` | su 切换需目标密码、加载新环境（`su -`）；**sudo 用自身密码、可细粒度授权**，配置在 `/etc/sudoers`（必须 `visudo` 编辑），`sudo -l` 查自己权限 |
| ★ `useradd` / `usermod` / `userdel` | `useradd -m -s /bin/bash tom` | `-m` 建家目录，`-G docker,users` 加附加组，`userdel -r` 连家目录删 |
| ☆ `passwd` | `passwd -l user` / `passwd -S user` | 锁账户/查账户状态；`chage -l user` 看密码过期策略 |
| ★ `id` | `id tom` | 看 uid/gid/附加组，权限问题第一步 |
| ☆ `who` / `w` | `w` | 谁在线、在跑什么；`last` 成功登录、`lastb` 失败登录（安全审计考点） |

**必背配置文件**：`/etc/passwd`（7 字段：用户:密码占位:UID:GID:注释:家目录:Shell，**shell 设为 /sbin/nologin 即禁止登录**）、`/etc/shadow`（密码哈希与过期策略）、`/etc/group`。

**面试问答点**：
- 「777 权限有什么问题？」→ 任何人可改可执行，脚本可被植入后门；正确做法是给属主合适权限 + 调整属主/属组。
- 「SUID/SGID/Sticky 是什么？」→ SUID：执行时获得文件主权限（passwd 是典型，`chmod u+s`）；SGID：目录内新文件继承组；Sticky：目录内只能删自己的文件（/tmp）。

---

## 五、进程管理与任务调度

| 命令 | 必背示例 | 记忆点 / 考察点 |
|------|---------|----------------|
| ★ `ps` | `ps -ef \| grep nginx` / `ps aux --sort=-%cpu \| head` | `-ef` 全进程树视图、`aux` 含 %CPU/%MEM；**排序取 Top 是笔试模板** |
| ★ `top` | `top` 中按 `P` CPU、`M` 内存、`1` 展开每核、`c` 显示完整命令、`k` 杀进程、`H` 看线程 | 交互键必考！重点列：PID/USER/%CPU/%MEM/STAT/TIME+ |
| ☆ `STAT` 列 | `Z` 僵尸进程、`D` 不可中断（多为磁盘 IO 卡） | **僵尸进程=子进程退出、父进程未回收，杀不掉，只能杀掉/重启父进程**；D 状态常伴随 IO hang |
| ★ `kill` | `kill -15 1234` / `kill -9 1234` | **-15（TERM）默认，可捕获做清理；-9（KILL）强制，进程来不及释放资源**；先 15 后 9 是标准操作；`-1`（HUP）平滑重载配置 |
| ☆ `pkill` / `killall` | `pkill -f "python app.py"` | 按名字/关键字批量杀；`-f` 匹配完整命令行 |
| ★ `nohup` | `nohup ./long.sh > run.log 2>&1 &` | 退出终端不挂断；`2>&1` 把错误并入标准输出（重定向写法必考）；配合 `disown`/`&` 记忆 |
| ★ `jobs` / `fg` / `bg` | `Ctrl+Z` 挂起 → `bg %1` 后台继续 | 前台任务停到后台的标准动作 |
| ☆ `nice` / `renice` | `renice -n 10 -p 1234` | 优先级调整，值越小越优先（-20~19） |
| ★ `crontab` | `crontab -e`；`0 2 * * * /backup.sh`；`*/5 * * * * cmd` | **五字段：分 时 日 月 周**；`*/5` 每 5 分钟；`0 2 * * *` 每天凌晨 2 点；`crontab -l` 查看；生产注意：写绝对路径、日志重定向、PATH 与手动执行不同（高频坑） |
| ☆ `at` | `echo "reboot" \| at 23:30` | 一次性定时任务；`atq` 查、`atrm` 删 |
| ★ `lsof` | `lsof -i :80` / `lsof \| grep deleted` | 看端口占用进程；**df 显示满但 du 找不到大文件 → 被删除但进程仍持有的句柄**（经典面试题） |
| ★ `pidof` / `pgrep` | `pidof nginx` | 快速拿 PID |

**cron 时间表达式自测**（背下来）：
- `*/10 9-18 * * 1-5` → 工作日 9 点到 18 点每 10 分钟
- `0 0 1 * *` → 每月 1 号 0 点
- `@reboot` → 开机执行

---

## 六、服务与系统管理（systemd）

| 命令 | 必背示例 | 记忆点 / 考察点 |
|------|---------|----------------|
| ★ `systemctl` | `systemctl start nginx` / `stop` / `restart` / `status` | 服务生命周期四件套 |
| ★ `systemctl enable` | `systemctl enable --now nginx` | **开机自启（enable）和立即启动（start）是两件事**，`--now` 一次搞定；`is-enabled` 检查 |
| ☆ `systemctl` 批量 | `systemctl list-units --type=service --state=running`；`systemctl list-units --failed` | 查所有运行服务 / 查启动失败的服务（排障入口） |
| ★ `journalctl` | `journalctl -u nginx -f` / `journalctl --since "10 min ago"` / `journalctl -p err` | systemd 日志统一入口；`-u` 指定服务、`-f` 实时、`-p err` 只看错误级别；`--since/--until` 按时间查 |
| ☆ `service` | `service nginx status` | 老 SysV 风格，CentOS6 记忆点：service 和 systemctl 的区别 = SysVinit vs systemd |
| ☆ `runlevel` / `get-default` | `systemctl get-default` | systemd 用 target 代替运行级别：`multi-user.target`≈3、`graphical.target`≈5 |
| ★ `systemd` 单元文件 | `cat /usr/lib/systemd/system/nginx.service` | 考点：服务起不来先看 ExecStart；自定义服务写 unit 文件后必须 `systemctl daemon-reload` |
| ☆ `reboot` / `shutdown` | `shutdown -h +10` / `shutdown -c` | 10 分钟后关机 / 取消；面试提一句"直接 reboot 不发通知是不专业的" |

---

## 七、网络配置与故障排查（面试占比最高）

### 7.1 查配置 / 查连通

| 命令 | 必背示例 | 记忆点 / 考察点 |
|------|---------|----------------|
| ★ `ip` | `ip addr` / `ip route` / `ip link set eth0 down` | 替代 ifconfig 的现代命令；`ip neigh` 看 ARP |
| ☆ `ifconfig` | `ifconfig eth0` | 老命令仍会考；up/down 状态、MTU、丢包计数 RX/TX errors |
| ★ `ping` | `ping -c 5 8.8.8.8` | 通≠业务通（只证明 ICMP 可达）；不通则分段：本机网关→DNS→目标（分层排障思路必讲） |
| ★ `ss` | `ss -antlp` / `ss -s` | **`-a` 全部 `-n` 数字 `-t` TCP `-l` 监听 `-p` 进程**，背这五个字母组合；`ss -s` 看连接数汇总；比 netstat 快 |
| ☆ `netstat` | `netstat -lntp` | 同义考察；`netstat -an \| grep TIME_WAIT \| wc -l` 统计连接状态 |
| ★ `telnet` / `nc` | `nc -zv 10.0.0.5 3306` | **测试 TCP 端口通不通的标准动作**；`telnet ip port` 也行 |
| ★ `curl` | `curl -I https://x.com`（只看头）/ `curl -s -o /dev/null -w "%{http_code} %{time_total}\n" url`（测状态码+耗时） | HTTP 层排查必考；`-L` 跟随跳转、`-X POST -d '{}'`、`-H 'Token:xx'`、`-m 5` 超时 |
| ☆ `wget` | `wget -c url` | `-c` 断点续传 |
| ★ `dig` | `dig www.baidu.com` / `dig +short x.com` / `dig -x 1.2.3.4` | DNS 解析链路：看 AUTHORITY/ANSWER 段；`nslookup` 是老版对比考点 |
| ☆ `traceroute` / `mtr` | `mtr -rw -c 20 8.8.8.8` | 链路定位哪一跳丢包；**mtr = traceroute+ping 持续统计**（加分） |
| ☆ `arping` | `arping -I eth0 192.168.1.1` | 局域网 IP 冲突定位 |
| ★ `tcpdump` | `tcpdump -i any host 10.0.0.8 and port 443 -nn -c 200 -w out.pcap` | 抓包必背骨架：`-i` 网卡、`-nn` 不解析、`-c` 数量、`-w` 存文件给 Wireshark；`-r` 读文件；口头考点"TCP 三次握手在抓包里长什么样"→ SYN / SYN,ACK / ACK |

### 7.2 防火墙

| 命令 | 必背示例 | 记忆点 / 考察点 |
|------|---------|----------------|
| ★ `iptables` | `iptables -L -n --line-numbers`；`-A INPUT -p tcp --dport 80 -j ACCEPT`；`-D INPUT 3`；`-F` | 四表五链口头考点（filter/nat + INPUT/OUTPUT/FORWARD）；**规则顺序匹配，第一条命中即停**；保存：`iptables-save > /etc/sysconfig/iptables` |
| ★ `firewalld` | `firewall-cmd --add-port=80/tcp --permanent && firewall-cmd --reload` | CentOS7 默认；**改 --permanent 后必须 reload 才生效**（高频坑）；`--list-all` 看当前 zone |
| ☆ `ufw` | `ufw allow 22/tcp` | Ubuntu 侧记忆点 |

### 7.3 网络配置文件

- ★ CentOS：`/etc/sysconfig/network-scripts/ifcfg-ens33`（`BOOTPROTO=static/dhcp`、`ONBOOT=yes`），改完 `systemctl restart network`（或 `nmcli con reload/up`）
- ★ Ubuntu：`/etc/netplan/*.yaml`，改完 `netplan apply`
- ★ 通用：`/etc/hosts`（本机静态解析）、`/etc/resolv.conf`（DNS 服务器）
- ☆ `nmcli device status` / `nmcli con up ens33`

**面试问答点**：
- 「服务器访问外网不通，怎么排查？」→ 标准链路：`ip addr` 看本机 IP → `ping` 网关 → `ip route` 看默认路由 → `ping 8.8.8.8` 证连通 → `dig` 证 DNS → `iptables/firewalld` 证策略 → 云环境查安全组。**按 OSI 分层从上往下讲，思路比命令更重要。**
- 「TIME_WAIT 很多有影响吗？」→ 主动关闭方才会积累；少量正常（等 2MSL），上万级考虑调 `tcp_tw_reuse`、用长连接/连接池，别乱开 `tcp_tw_recycle`（NAT 环境会出问题，新版内核已移除）。

---

## 八、磁盘、分区与文件系统

| 命令 | 必背示例 | 记忆点 / 考察点 |
|------|---------|----------------|
| ☆ `fdisk` | `fdisk -l` / `fdisk /dev/sdb` | 看分区表、新建分区（n）→ 写盘（w）；>2TB 必须 GPT 分区（MBR 上限 2TB，经典题） |
| ☆ `parted` | `parted /dev/sdb mklabel gpt` | GPT 大分区场景 |
| ★ `mkfs` | `mkfs.xfs /dev/sdb1` / `mkfs.ext4` | CentOS7+ 默认 XFS；考点：**XFS 只能扩容不能缩容，ext4 两者皆可** |
| ★ `mount` | `mount /dev/sdb1 /data` / `umount -l /data` | 设备忙用 `-l`（lazy）卸载；`mount -o remount,rw /` 重挂读写 |
| ★ `/etc/fstab` | UUID 挂载行：`UUID=xxx /data xfs defaults 0 0` | 开机自动挂载；**改 fstab 后必须先 umount 再 mount -a 验证，写错会起不来**（面试强调这点=细心加分）；最后两列是 dump/fsck 周期，网络盘填 0 0 |
| ★ `dd` | `dd if=/dev/zero of=test bs=1M count=1024 conv=fdatasync` | 磁盘测速/造测试文件；`if` 输入 `of` 输出 bs 块大小；**慎用 of=/dev/sdX 会毁数据**（面试官会看你是否知道危险） |
| ★ LVM | `pvcreate /dev/sdb` → `vgcreate vg0 /dev/sdb` → `lvcreate -L 10G -n lv_data vg0` → 挂载；扩：`lvextend -L +5G /dev/vg0/lv_data` → `resize2fs`（ext4，针对设备）或 `xfs_growfs /data`（xfs，针对挂载点） | **LVM 三层 PV/VG/LV 结构 + 在线扩容流程是必考题**；XFS 与 ext4 扩容命令不同是常见追问 |
| ☆ swap | `swapon -s` / `mkswap /dev/sdc && swapon /dev/sdc` | swap 分区创建；考点：内存充足也要留少量 swap 防 OOM 直接杀进程 |
| ☆ RAID | `cat /proc/mdstat`；`mdadm --detail /dev/md0`；`mdadm /dev/md0 --rebuild /dev/sdc` | 软 RAID 级别口头考点：**RAID0 条带无冗余、RAID1 镜像、RAID5 分布式校验允许坏 1 盘、RAID10 先镜像后条带** |
| ★ inode | `df -i` | inode 是什么：文件的元数据索引节点，大小固定；**大量小文件可耗尽 inode**；修复：删无用碎文件或 mkfs 时调大 inode 比例 |

---

## 九、日志与故障排查

| 命令 | 必背示例 | 记忆点 / 考察点 |
|------|---------|----------------|
| ★ 日志路径 | `/var/log/messages`（CentOS 综合）/ `/var/log/syslog`（Ubuntu）；`/var/log/secure`（CentOS 认证）/ `/var/log/auth.log`（Ubuntu）；`/var/log/cron`；`/var/log/dmesg`；`/var/log/yum.log`（dnf）/ `/var/log/dpkg.log` | 发行版差异背熟，问到 SSH 爆破立刻答 secure/auth.log |
| ★ `tail -f` | `tail -f -n 200 app.log` | 实时看日志；`-F` 应对日志轮转 |
| ★ `less` | `less app.log` 后 `/err` 搜索、`n/N` 跳转 | 大日志别用 vim/cat |
| ★ 登录审计 | `last -n 20` / `lastb` | 最近登录 / 失败尝试（安全题） |
| ★ 误删恢复 | 场景：rm -rf 误删 | 口头标准答案：**立即停止写入**（最好卸载或 remount 只读）→ ext4 可试 extundelete/debugfs，xfs 难恢复 → 靠备份/快照；**未保存的文件句柄可通过 `/proc/PID/fd` 找回内容**（进程还持有时有救）→ 引申：所以要有备份策略 |
| ★ 大日志清理 | 正确姿势 | **不能 rm 正在写的日志**（句柄不释放，空间不回收）→ 用 `> app.log` 或 `truncate -s 0 app.log` 清空；日志轮转用 logrotate（`copytruncate` vs create 模式） |

---

## 十、性能分析与压测排查

| 命令 | 必背示例 | 记忆点 / 考察点 |
|------|---------|----------------|
| ★ `top` | 见第五章 | 进 top 先看 load、再按 P/M 排序，看 us/sy/wa/id 比例 |
| ★ `vmstat` | `vmstat 1 5` | 每秒采样 5 次。列必讲：**r（运行队列>核数说明 CPU 争抢）、b（被 IO 阻塞的进程）、si/so（在用 swap 了）、us/sy/id/**wa**（wa 高=磁盘 IO 瓶颈）** |
| ★ `iostat` | `iostat -x 1` | 磁盘：`%util` 接近 100%=设备饱和、`await` 平均等待时长(ms)、`aqu-sz` 队列深度；配合 `du`/`find` 定位写入大户 |
| ★ `sar` | `sar -u 1 3`（CPU）/ `sar -r`（内存）/ `sar -n DEV 1`（网卡）/ `sar -n TCP,ETCP`（连接）/ `sar -q`（队列） | **能看历史数据**（sysstat 定时采集），`sar -u -f /var/log/sa/sa27` 看 27 号——事后复盘神器，很多人不知道，说出来就是亮点 |
| ☆ `mpstat` | `mpstat -P ALL 1` | 单核热点（某核 100% 其他空闲 → 单线程瓶颈/中断不均） |
| ☆ `pidstat` | `pidstat -d -p 1234 1` | 看**进程级** CPU/内存/IO 贡献 |
| ☆ `strace` | `strace -p 1234 -f -T` | 跟踪进程系统调用：进程卡住/D 状态时看它卡在哪次 syscall（文件/网络/锁） |
| ☆ `perf` | `perf top` | 函数级热点（进阶加分项，不强求） |
| ☆ 网卡流量 | `iftop -i eth0` / `nload eth0` / `ifstat 1` | 带宽被谁占了：iftop 看连接对端，nethogs 看进程 |
| ☆ 句柄 | `ulimit -n` / `lsof -p 1234 \| wc -l` / `cat /proc/sys/fs/file-max` | "too many open files" 排查链：进程级 ulimit → 全局 file-max → limits.conf 持久化 |

**标准答题模板（必背）——「服务器很慢，你怎么排查」**：
1. `uptime` 看负载 → 对比核数判断压力等级
2. `top` 分型：us 高=计算型（找进程）；wa 高=IO 型；sy 高=系统调用/中断
3. CPU 型：`top -H -p PID` 看线程 → `pidstat`/`strace` 定位
4. IO 型：`iostat -x 1` 确认饱和盘 → `find / -newer` 或 `iotop -o` 找写入进程
5. 内存型：`free -m` → `dmesg \| grep -i oom` 看是否杀过进程
6. 网络型：`sar -n DEV` / `ss -s` 看带宽与连接状态堆积
> 答题强调"从整体到局部、先定位类型再下钻进程"，思路分比命令分更重要。

---

## 十一、软件包与压缩备份

| 命令 | 必背示例 | 记忆点 / 考察点 |
|------|---------|----------------|
| ★ `yum`/`dnf` | `yum install -y nginx` / `yum list installed \| grep httpd` / `yum search` / `yum history undo 5` | CentOS 系；`-y` 免确认；`yum history` 可回滚安装（加分）；本地源/换源：`/etc/yum.repos.d/*.repo` |
| ★ `rpm` | `rpm -qa \| grep nginx`（查）/ `rpm -ql nginx`（装了哪些文件）/ `rpm -qf /usr/sbin/nginx`（**反查文件属于哪个包**）/ `rpm -ivh pkg.rpm` / `rpm -e --nodeps` | 口头考点：**yum 是上层依赖管理，rpm 是底层包格式；rpm 装包不处理依赖** |
| ★ `apt` / `dpkg` | `apt update && apt install -y nginx` / `dpkg -l` / `dpkg -i pkg.deb` / `apt-cache search` | Ubuntu 系；dpkg 之于 apt ≈ rpm 之于 yum |
| ★ `tar` | 打包：`tar czf bak.tar.gz /data`；解包：`tar xzf bak.tar.gz -C /opt`；查看不解压：`tar tzf` | **z 压缩 g gzip f 指定文件，c 打包 x 解包 t 列表**——四字母口诀背熟；考点：`tar jzf`=bzip2、`tar -tf xxx.tar.gz` 检查包内容 |
| ☆ 备份组合 | `tar czf /backup/etc-$(date +%F).tar.gz /etc` | `$(date +%F)` 动态文件名 + cron 每周全备 = 完整答题模板；配合 `find ... -mtime +30 -delete` 做保留策略 |
| ★ `source` vs `sh` | `source /etc/profile` | **source 在**当前 shell **执行（环境变量立刻生效）；sh/bash 开子 shell 执行，改了不回流**（高频口头题） |
| ☆ `which` / `whereis` / `type` | `which nginx` | which 查可执行文件路径（按 PATH）；type 还能识别 alias/builtin；`alias` 查别名，`\cmd` 临时绕过别名 |

---

## 十二、Shell 使用进阶（脚本相关必背）

| 知识点 | 必背示例 / 结论 |
|--------|----------------|
| ★ 重定向 | `cmd > out 2> err`；合并 `cmd > all.log 2>&1`；追加 `>>`；`/dev/null` 丢弃输出 |
| ★ 逻辑控制 | `cmd1 && cmd2`（成功才跑）、`cmd1 \|\| cmd2`、`cmd1; cmd2`（顺序无关）、后台 `cmd &` |
| ★ 引号 | 双引号变量生效 `"${name}"`；单引号原样 `'${name}'`；反引号/`$()` 命令替换（推荐 `$()` 可嵌套） |
| ★ 变量 | `a=1`（等号不能有空格）；`$?` 上一条命令退出码（0 成功）、`$0` 脚本名、`$1`/`$#`/`$@` 参数、`${var:-默认值}` |
| ★ 判断与循环 | `if [ ! -f "$file" ]; then ... fi`；`[ -d ]` 目录、`[ -e ]` 存在、`[ -eq ]` 数字相等（字符串用 =）；`for i in {1..10}`；`while read line` 处理文件 |
| ★ 脚本健壮性 | `#!/bin/bash` + `set -e`（遇错退出）`set -o pipefail`（管道任一段失败算失败）；核心配置脚本再加 `set -u` |
| ★ 定时任务脚本坑 | cron 中 PATH 极简、命令必须绝对路径、变量不会自动加载 /etc/profile → 脚本开头手动 source 或写全路径 |
| ☆ 快捷键 | `Ctrl+R` 历史搜索、`!!` 上条命令、`sudo !!` 用 root 重跑上条、`Ctrl+A/Home`、`Ctrl+E` 行尾、`Ctrl+L` 清屏、`Tab` 补全 |
| ☆ 会话保持 | `tmux new -d -s work` / `tmux attach -t work`（远程长任务防断线，比 nohup 灵活）；`screen -r` 同理 |
| ☆ 提权排查 | `/etc/sudoers` 用 `visudo` 编辑（语法检查防锁死）；sudo 失败先看是否 sudo 组、再看日志 |
| ☆ 环境变量 | `env` / `export PATH=$PATH:/tool`；持久化写 `~/.bashrc`（当前用户）或 `/etc/profile`（全局），改完 source |

---

## 十三、笔试必背命令组合 Top 10（逐条背）

1. **查端口被谁占用**：`ss -lntp | grep :80` 或 `lsof -i:80`
2. **CPU 最高的 10 个进程**：`ps aux --sort=-%cpu | head -11`
3. **内存最高的 10 个进程**：`ps aux --sort=-%mem | head -11`
4. **访问日志 Top10 IP**：`awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10`
5. **找出 7 天前的日志并删除**：`find /var/log -name "*.log" -mtime +7 -delete`
6. **找出 500M 以上大文件**：`find / -xdev -type f -size +500M -exec ls -lh {} \;`
7. **统计 SSH 爆破失败次数**：`grep "Failed password" /var/log/secure | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head`（或简述：按来源 IP 统计）
8. **实时看日志并过滤错误**：`tail -f app.log | grep --line-buffered -E "ERROR|FATAL"`（`--line-buffered` 是细节加分项）
9. **备份配置目录并带日期**：`tar czf /backup/etc-$(date +%F).tar.gz /etc && find /backup -mtime +30 -delete`
10. **后台跑任务且退出不挂**：`nohup ./batch.sh > batch.log 2>&1 & echo $!`（`$!` 记录 PID，kill 有据可查）

---

## 十四、高频口头题速答卡

| 问题 | 速答要点 |
|------|---------|
| 硬链接 vs 软链接 | 硬链接共享 inode、不能跨分区/不能链目录、删原文件仍可读；软链接独立 inode 存路径、可跨目录、原文件没了变死链。类比：软链接≈Windows 快捷方式 |
| kill -9 和 -15 区别 | -15 默认、可被程序捕获做收尾（关连接/落盘）；-9 内核直接回收、无清理。最佳实践：先 -15，等几秒仍不退再 -9 |
| 磁盘空间不降反升/删了没效果 | ① 句柄未释放：`lsof | grep deleted`，重启进程或 `> log` 清空；② inode 满：`df -i` |
| buffer vs cache | buffer 是写磁盘前的缓冲（异步写），cache 是读磁盘内容的缓存（加速读），都占内存都可回收 |
| load 多少算高 | 对比逻辑核数：load < 核数×0.7 正常；持续 > 核数×2 危险。还要看是 CPU 密集还是 IO 等待抬起来的 |
| 僵尸进程怎么处理 | STAT=Z，有父未 wait；kill -9 无效（已死），定位父进程 `ps -o ppid= -p PID` → 修复父进程或重启；根因是父代码没 reap |
| 密码策略在哪里改 | `/etc/login.defs`（长度/有效期默认）+ `chage -l 用户` 查看；生产常要求 90 天轮换、复杂度用 pwequality/pam_cracklib |
| SSH 优化怎么做 | `/etc/ssh/sshd_config`：禁 root 直登（PermitRootLogin no）、改端口、密钥登录（PasswordAuthentication no）、`ClientAliveInterval 60` 防超时；用 `ss -lntp` 确认端口监听生效 |
| 新盘怎么挂上去用 | `lsblk` 确认 → `fdisk/parted` 分区 → `mkfs.xfs` → 创建挂载点 → **先手动 mount 验证** → 写 `/etc/fstab`（UUID）→ `mount -a` 测试无报错 |
| 服务器时间不对 | `timedatectl` 看 NTP 状态 → `chronyc sources` → `ntpdate/chronyd` 同步 → 云机查安全组放行 NTP UDP 123 |
| 日志太大怎么办 | logrotate 轮转（按天/大小、compress、保留天数）；ELK/向量采集外送；生产禁止 rm 在线日志 |
| 系统起不来 | 单用户/救援模式（GRUB 编辑 linux 行加 `rd.break`/`single`）→ 常见原因：fstab 写错、权限改坏（/ 被改成非 755）、内核升级失败换旧内核进、SELinux 异常 |
| yum 装不了排查 | 网络 → `ping mirrors` → repo 文件配置 → `yum clean all && makecache` → 看报错里哪个 repo 挂了可 disable 临时绕过 |
| 一台机器怎么看 CPU 型号/内存条数/磁盘数 | `lscpu`、`free -h` + `dmidecode -t memory`、`lsblk`/`fdisk -l`，虚拟机需先说明 dmidecode 槽位信息可能不准 |

---

## 十五、操作系统原理与启动（初级必背理论）

**启动流程（必须能一口气口述）**：
`加电自检 POST（BIOS/UEFI）→ 引导加载器 GRUB（读 /boot/grub2/grub.cfg）→ 加载内核（vmlinuz，initramfs 临时挂载根以加载驱动）→ systemd 成为 PID 1，按依赖启动 default.target 下的服务 → 输出登录提示`。
追问必答：CentOS7 的 `multi-user.target` ≈ 老运行级别 3（命令行），`graphical.target` ≈ 5（图形）。

**运行级别口诀**：0 关机、1 单用户、2 多用户无网络、3 多用户命令行、4 未用、5 图形、6 重启。

| 主题 | 必背要点 / 命令 |
|------|----------------|
| ★ root 密码找回 | 重启在 GRUB 按 `e`，linux 行尾加 `rd.break` → `mount -o remount,rw /sysroot` → `chroot /sysroot` → `passwd root` → **`touch /.autorelabel`**（SELinux 开启时不加可能起不来）→ reboot。单用户模式是初级面试高频题 |
| ★ SELinux | `getenforce` 看模式；`setenforce 0` 临时 permissive——验证"是不是 SELinux 捣乱"的标准动作；永久关：`/etc/selinux/config` 改 `SELINUX=disabled`（需重启）；排障口诀：Nginx 静态页 403、sshd 诡异拒绝，先 `ausearch -m avc -ts recent` 看拦截记录 |
| ★ /proc 虚拟文件系统 | 不占磁盘，内核导出的运行时数据。必知：`/proc/loadavg`、`/proc/meminfo`、`/proc/cpuinfo`、`/proc/PID/fd`（进程打开的句柄）、`/proc/PID/environ`（进程环境变量）。"top 的数据哪来的？"→ 就是读 /proc |
| ★ inode 口述三点 | 固定大小的**元数据块**（权限、时间、大小、数据块指针，**不含文件名**）；读取路径 = 目录项(dentry) → inode → 数据块；inode 耗尽时磁盘有空间也建不了文件，`df -i` 查，海量小文件是主因 |
| ★ 脚本执行三种方式 | `sh a.sh`：子 shell 执行，不需要 x 权限；`./a.sh`：子 shell，**需要执行权限**+shebang；`source a.sh`：**当前 shell** 执行，改的环境变量立即生效。"source 和 sh 区别"是必考题 |
| ☆ OOM | 内存耗尽时内核"杀进程自保"（`dmesg \| grep -i oom` 查证）；挑选逻辑看 oom_score（占用内存大的先死）；防范：进程/容器设内存上限、保留适量 swap、监控内存水位 |
| ☆ cron 与 anacron | cron 到点必须机器在线，错过不补；anacron 面向不常开机的机器，开机后补跑（/etc/cron.daily 等就是它管的）——实习岗爱问的区别 |

---

## 十六、网络与协议理论（初级必背）

**OSI 七层（自下而上背）**：物理(比特) → 数据链路(帧/MAC/交换机) → 网络(包/IP/路由器) → 传输(TCP·UDP/端口) → 会话 → 表示 → 应用(HTTP)。**TCP/IP 四层**：网络接口 → 网际 → 传输 → 应用。

| 主题 | 必背话术 |
|------|---------|
| ★ TCP vs UDP | 面向连接/无连接；可靠（确认+重传+窗口+拥塞控制）/尽力而为；有序字节流/报文可乱序；首部 20B/8B；场景：网页·传输·SSH 用 TCP，DNS·语音·视频用 UDP |
| ★ 三次握手 | ① 客户端 SYN，seq=x → ② 服务端 SYN+ACK，seq=y，ack=x+1 → ③ 客户端 ACK，ack=y+1。**为什么不能两次**：防止失效的旧 SYN 到达服务端直接建连接浪费资源；且第三次才能证明"客户端收能力+服务端发能力"，双方收发各验证一次 |
| ★ 四次挥手 / TIME_WAIT | 双方各发 FIN 各回 ACK；**主动关闭方**进入 TIME_WAIT 等 2MSL：保证最后一个 ACK 可达、让滞留报文在网络中消亡。追问必答：**CLOSE_WAIT 堆积=应用程序不调 close（代码 bug）；TIME_WAIT 堆积=大量短连接**（上长连接/连接池，别乱调内核回收参数） |
| ★ DNS 解析全流程 | 浏览器缓存 → 系统缓存与 /etc/hosts → 本机 DNS 服务器（递归）→ 根域名 → 顶级域 → 权威域名（迭代）→ 逐层返回并按 TTL 缓存。记录类型：**A=IPv4、AAAA=IPv6、CNAME=别名、MX=邮件、NS=委派、TXT=验证/SPF** |
| ★ HTTPS | HTTP+TLS。握手：ClientHello（TLS 版本+密码套件）→ ServerHello+**服务端证书**→ 客户端用 CA 链验证证书真伪 → 非对称握手协商出**对称密钥** → 对称加密传数据（一句话：**非对称换钥匙，对称传数据**）。端口 443；证书常见故障：过期、证书链不全、SNI 配置错 |
| ★ HTTP 状态码 | 200 成功；301 永久重定向、302 临时、304 走缓存；400 参数错、401 未认证、403 禁止访问、404 不存在、429 限流；500 应用内部错、**502 网关收到无效响应（后端拒连/FPM 挂了）**、503 超载或维护、**504 后端超时**。502/504 排查：`ss -lntp` 看后端是否监听 → `curl 后端IP:端口` 直连测试 → 看 nginx error.log 具体报错（refused→502，timed out→504） |
| ★ 私有地址与主机数 | 10.0.0.0/8、172.16.0.0/12、192.168.0.0/16；可用主机 = 2^(32-前缀) − 2（/24=254 台，减网络地址和广播地址） |
| ☆ 正/反向代理 | 正向代理**替客户端**出去（客户端配置代理，出口审计/缓存）；反向代理**替服务端**收请求（客户端无感知，做负载均衡、SSL 卸载、隐藏源站、防护） |
| ☆ Bonding | mode0 轮询（负载均衡无容错）、mode1 主备 active-backup、mode4 LACP 动态聚合（需交换机配合）——机房驻场岗爱问 |

**常用端口速记（必背）**：
22 SSH｜21/20 FTP｜23 Telnet｜25 SMTP｜53 DNS｜67/68 DHCP｜80 HTTP｜110/143 POP3/IMAP｜443 HTTPS｜873 rsync｜3306 MySQL｜6379 Redis｜3389 RDP｜8080 应用代理｜9200 Elasticsearch｜123 NTP(UDP)｜161 SNMP(UDP)

---

## 十七、常用服务三件套：Nginx / MySQL / Redis（初级明确要求会部署）

### 17.1 Nginx

| 考点 | 必背内容 |
|------|---------|
| 目录结构 | 包安装：主配置 `/etc/nginx/nginx.conf`，子配置 `/etc/nginx/conf.d/*.conf`；日志 `/var/log/nginx/access.log、error.log`；源码编译装在 `/usr/local/nginx/`（面试可能问两者区别） |
| 配置修改三步 | 改 conf → `nginx -t`（检查语法，**改配置必跑**）→ `systemctl reload nginx`（或 `nginx -s reload`） |
| reload 原理（追问） | master 重读配置并拉起新 worker，老 worker 处理完存量连接后优雅退出，**不断连接**——这就是"平滑加载" |
| 核心指令 | `listen`、`server_name`（虚拟主机靠它区分）、`root/index`、`location`、`proxy_pass`（反代）、`upstream + proxy_pass http://name`（负载均衡） |
| location 优先级 | `=` 精确 > `^~` 前缀 > `~`/`~*` 正则（按书写顺序）> 普通前缀。口诀：**精确大于前缀，长前缀优于短前缀，正则按顺序** |
| 负载均衡算法 | 轮询（默认）、weight 权重、ip_hash（会话保持）、least_conn 最少连接 |
| 403/404 区分 | 404=路径没找到（root/alias 配错）；403=找到了但没权限（文件属主属组、目录缺 x、nginx worker 用户不对、SELinux） |
| 工具对比（口头题） | **Nginx**：4 层+7 层，Web/反代生态最好；**LVS**：4 层，内核级抗量大，无正则；**HAProxy**：4/7 层，健康检查和会话控制丰富、有监控页 |
| Keepalived | 基于 **VRRP** 协议：MASTER 发组播报文，BACKUP 超时未收到则把 **VIP** 抢过来，秒级切换；常配 Nginx/HAProxy 做"双机热备"，两机 vrid 和虚拟 IP 一致、priority 不同 |

### 17.2 MySQL

| 考点 | 必背内容 |
|------|---------|
| 装完拿密码 | CentOS 初始 root 临时密码：`grep 'temporary password' /var/log/mysqld.log` → 登录后 `ALTER USER ... IDENTIFIED BY` 改密 |
| 建号授权（最小权限原则） | `CREATE USER 'app'@'192.168.1.%' IDENTIFIED BY '强密码';` → `GRANT SELECT,INSERT,UPDATE,DELETE ON appdb.* TO 'app'@'192.168.1.%';` → `FLUSH PRIVILEGES;`。面试话术：**不给应用用 root，host 不写 %** |
| 逻辑备份 | `mysqldump -uroot -p --single-transaction appdb > appdb_$(date +%F).sql`；**--single-transaction 让 InnoDB 备份不锁表**；恢复：`mysql appdb < appdb.sql`。考点：逻辑备份=mysqldump（跨版本、可读），物理备份=xtrabackup（大库更快） |
| 主从复制原理（必考） | 主库写 **binlog** → 从库 **I/O 线程**连主库拉取写到 **relay log** → 从库 **SQL 线程**回放。检查：`SHOW SLAVE STATUS\G` 看 **Slave_IO_Running / Slave_SQL_Running 双 Yes** + `Seconds_Behind_Master` 延迟秒数；延迟原因：从库磁盘慢/大事务串行回放 |
| 慢查询 | 开 `slow_query_log`，`long_query_time=1`；分析用 `EXPLAIN` 看 type/key/rows（是否走索引、扫描行数），优化方向：加索引、改写 SQL、处理深分页 |
| 忘记 root 密码 | 配置文件加 `skip-grant-tables` 重启 → 免密进 → `FLUSH PRIVILEGES` 后 `ALTER USER` 改密 → 去掉配置重启 |

### 17.3 Redis

| 考点 | 必背内容 |
|------|---------|
| 基本操作 | 服务端 `redis-server /etc/redis.conf`；客户端 `redis-cli -h 127.0.0.1 -p 6379 -a 密码`；连通测试 `ping` → `PONG` |
| 关键配置 | `bind 0.0.0.0` + `protected-mode no`（内网生产仍需密码）、`requirepass`、`daemonize yes`、`maxmemory` + `maxmemory-policy`（如 allkeys-lru 满了按 LRU 淘汰） |
| 持久化（必考） | **RDB**：fork 子进程做内存快照（`save 900 1` / `bgsave`）——恢复快、会丢最后一次快照后的数据；**AOF**：追加写命令日志（`appendonly yes`，`everysec` 折中）——丢得少、文件大恢复慢；生产常两者都开 |
| 五大数据类型与场景 | String（缓存/计数器/session）、Hash（对象字段，如用户信息）、List（最新消息/简单队列）、Set（去重/交并集：共同好友）、ZSet（排行榜/延迟队列） |
| 为什么单线程还快 | 纯内存操作 + 无多线程上下文切换和锁竞争 + epoll 多路复用网络模型；追问"Redis6 多线程"：网络 IO 用多线程，**命令执行仍单线程** |

**LNMP 部署顺序（"让你搭一套 Web 服务怎么做"标准话术）**：
① 装 MySQL，建库建号 → ② 装 PHP + php-fpm（监听 127.0.0.1:9000，www 用户运行）→ ③ 装 Nginx，server 配 root + `location ~ \.php$ { fastcgi_pass 127.0.0.1:9000; }` → ④ 放行防火墙 80，SELinux 场景检查 httpd 相关布尔值/上下文 → ⑤ `curl -I` 验证页面，报错就分层看 nginx error.log / php-fpm 日志。**答出"每一步验证再进下一步"就是加分项。**

---

## 十八、现场手写脚本三题（初级机考真题级，整段背诵）

**脚本一：每日备份 + 保留 7 天 + 记日志**（最常被要求手写）
```bash
#!/bin/bash
# backup_data.sh —— 备份 /data 到 /backup，保留 7 天
SRC=/data
DEST=/backup
DATE=$(date +%F)
LOG=/var/log/backup.log

mkdir -p "$DEST"
tar czf "$DEST/data_$DATE.tar.gz" "$SRC" 2>>"$LOG"

if [ $? -eq 0 ]; then
    echo "$DATE backup OK" >> "$LOG"
    find "$DEST" -name "data_*.tar.gz" -mtime +7 -delete
else
    echo "$DATE backup FAILED" >> "$LOG"
fi
```
背诵要点：`$(date +%F)` 动态文件名、`$?` 判断成败、`find -mtime +7 -delete` 滚动清理、cron 每天 2 点跑：`0 2 * * * /scripts/backup_data.sh`

**脚本二：网站探活，挂了自动拉起**
```bash
#!/bin/bash
# check_web.sh —— 每 1 分钟由 cron 执行
CODE=$(curl -s -o /dev/null -w '%{http_code}' -m 5 http://127.0.0.1/)
if [ "$CODE" != "200" ]; then
    systemctl restart nginx
    echo "$(date '+%F %T') nginx down (code=$CODE), restarted" >> /var/log/web_check.log
fi
```
背诵要点：`curl -w %{http_code}` 拿状态码、非 200 才重启、必须留操作日志。

**脚本三：磁盘水位告警（阈值 85%）**
```bash
#!/bin/bash
# disk_alert.sh —— df 使用率超 85% 记录告警
df -hP | awk 'NR>1 {gsub("%","",$5); if ($5+0 > 85) print $6, $5}' | \
while read part pct; do
    echo "$(date '+%F %T') DISK ALERT: $part used $pct" >> /var/log/disk_alert.log
    # 生产可追加：mail -s "disk alert" ops@xx.com 发通知
    # 或企业微信/钉钉机器人 webhook：curl -s -X POST -d '{...}' 机器人地址
done
```
背诵要点：`df -P` 禁止换行保证 awk 取列、`gsub` 去百分号、告警出口说得出"邮件/webhook"即可。

**笔试高频一行式补题**：
- 批量看某进程所有打开文件：`ls -l /proc/$(pidof nginx | awk '{print $1}')/fd`
- for 循环给一批机器建同名用户：`for h in web1 web2; do ssh $h useradd deploy; done`（配合 SSH 免密）
- 找出昨天新建的文件：`find /data -type f -newermt "$(date -d yesterday +%F)" ! -newermt "$(date +%F)"`
- ping 扫描网段存活：`for i in {1..254}; do ping -c1 -W1 192.168.1.$i >/dev/null 2>&1 && echo 192.168.1.$i is up; done`
- 杀掉异常进程的组合拳：`ps -ef | grep python_app | grep -v grep | awk '{print $2}' | xargs kill -9`（`grep -v grep` 防杀到自己，笔试爱挖这个空）

---

## 十九、监控 / 自动化 / 容器 / 备份（初级要求"认知级"，一句话答对）

| 主题 | 一句话标准答案 |
|------|---------------|
| ★ 监控系统闭环 | 采集（Agent/exporter）→ 存储（时序库）→ 告警（规则+通知）→ 可视化（大盘）；说得出"没有监控的运维等于盲人" |
| ★ Zabbix | Server+Agent 架构，Agent 分 passive（server 拉）/active（agent 推）；模板+自动发现（LLD）批量生成监控项；触发器表达式报警；企业传统机房主流 |
| ★ Prometheus | **拉模式**采集指标（counter/gauge/histogram），PromQL 查询（`rate(x[5m])` 算 QPS），Alertmanager 做分组/抑制/静默，Grafana 展示；云原生环境主流 |
| ☆ 两者对比话术 | Zabbix 开箱模板多、传统设备全；Prometheus 多维标签+动态服务发现，适合容器化环境 |
| ★ ELK | Filebeat/Logstash 收集 → Elasticsearch 存储检索 → Kibana 展示，解决"服务多、日志散，登机器 grep 不过来"；运维日常：按关键字+时间范围查日志 |
| ★ Ansible | **无 Agent**、基于 SSH；inventory 管主机分组；`ansible all -m ping` 即席执行；playbook 用 YAML 编排任务；模块 copy/template/service/shell，**幂等**（内容一致就不动），handler 只在变更时触发重启 |
| ★ Docker vs 虚拟机 | 容器共享宿主机内核、镜像分层、秒级启动、更省资源；VM 有完整 guest OS、隔离更强 |
| ★ Docker 高频命令 | `docker run -d -p 80:80 -v /data:/html --name web nginx` / `ps -a` / `exec -it web sh` / `logs -f --tail 100` / `stop` / `rm` / `images` / `build -t app:1.0 .`；容器退了日志还能看，排障第一步 `docker logs` |
| ☆ Dockerfile 指令 | FROM（基础镜像）→ COPY（拷文件）/ADD（额外会解 tar、拉 URL，**日常优先 COPY**）→ RUN（构建时执行）→ CMD（启动默认命令，可被 run 参数覆盖）/ENTRYPOINT（固定入口，参数追加）→ EXPOSE（声明端口）→ VOLUME |
| ☆ K8s 认知底线 | Pod（最小调度单位）/Deployment（管副本与滚动升级）/Service（给一组 Pod 稳定入口）/Node；apiserver 是总入口，etcd 存集群状态，kubelet 管本节点 Pod。初级答到"知道它解决多容器编排/自愈/扩缩"即可 |
| ★ 云主机排障 | **安全组=云平台侧的实例防火墙**，iptables 放行但端口仍不通 → 先查安全组入方向规则；数据保护靠快照；网络隔离靠 VPC |
| ★ 备份三口诀 | **全量**：一次全拷，恢复只用一份；**增量**：只备"上次备份后"的变化，备得快、恢复要全量+逐份增量（最慢）；**差异**：只备"上次全量后"的变化，恢复=全量+最后一份差异。策略话术：**3-2-1 原则**——3 份数据、2 种介质、1 份异地 |
| ☆ CI/CD 一句话 | 代码提交后自动构建 → 自动测试 → 自动部署（Jenkins/GitLab CI），价值是"把发版从手艺活变成流水线"——被问到表达态度：愿意学 |

---

## 二十、初级人题、故障叙事与反问（态度分占初级面试 30%+）

**自我介绍模板（运维版，≤200 字，替换[X]）**：
> 我叫[X]，[学历/专业]背景，近[X]年从事 Linux 系统运维。日常负责[X]台服务器的[监控/发布/排障]，熟练系统命令与故障定位，常用 top/ss/lsof 排查问题；写过备份、探活、日志清理类 Shell 脚本并用 cron 定时执行；服务侧维护过 Nginx+MySQL+Redis 环境，个人实验环境在练 Docker 和 Ansible。应聘贵司是因为[岗位方向匹配点]，希望在[具体方向]持续深入。

**STAR 讲故障/项目（提前写熟 1–2 个）**：
S 背景（什么规模环境）→ T 任务（我的角色边界）→ A 行动（**具体到命令**：uptime→top→iostat→重启/切换/扩容）→ R 结果（量化：恢复用时、告警下降 + 复盘改进：加了什么监控项/巡检脚本）。没有生产经验就用虚拟机实验、家庭 lab 套这个结构，**别硬编——面试官一追问细节就穿帮**。

**值班与故障响应 SOP（体现流程意识）**：
① 告警响应（SLA 内确认）→ ② 判断影响面（是否有用户在流血）→ ③ **先恢复后根因**：重启服务/切流量/降级优先恢复业务 → ④ 每 15–30 分钟同步进展，超出权限或 30 分钟未缓解**及时升级上报** → ⑤ 根因修复 → ⑥ 故障报告：时间线+根因+改进项。说出"先恢复再根因、主动升级"这两点，初级面试直接拉开差距。

**其他高频人题**：
- 「运维的价值怎么理解？」→ 稳定压倒一切：可用性、数据安全、效率；把重复劳动自动化掉。
- 「经验少/薪资从低起步怎么看？」→ 用学习动作回答（在刷的命令、在搭的实验环境、计划考 RHCSA/CKA），表达踏实不玻璃心。
- 「接受 7×24 值班/半夜处理故障吗？」→ 接受，并补一句"同时会推动完善告警和预案，减少半夜误报"——既不怂又专业。
- 「遇到不会的问题怎么办？」→ 定位边界→查日志和 man/官方文档→最小化复现→带着排查结论去求助，不空手问。

**反问环节（问 2–3 个，别问福利）**：团队规模和值班机制？现在技术栈（公有云？容器化程度？用的什么监控）？新人有没有 mentor/培训安排？这个岗位前三个月最重要的事是什么？

**三个禁忌**：贬低前公司；把项目吹到自己答不上追问；说高风险操作"重启一下就好了"（要补：先评估影响、留现场证据再操作）。

---

## 附：CentOS 与 Ubuntu 差异对照表（防问混）

| 项目 | CentOS/RHEL | Ubuntu/Debian |
|------|-------------|---------------|
| 包管理 | yum / dnf + rpm | apt + dpkg |
| 综合日志 | /var/log/messages | /var/log/syslog |
| 认证/SSH 日志 | /var/log/secure | /var/log/auth.log |
| 防火墙 | firewalld / iptables | ufw / iptables |
| 网络配置 | ifcfg-* / nmcli | netplan |
| 默认文件系统 | XFS（7+） | ext4 |
| 添加用户组命令 | usermod -aG | usermod -aG（同）；adduser 交互式 |

