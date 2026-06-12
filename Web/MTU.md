# WSL2 + 透明代理环境下 TLS EOF 问题排查实录: 从 curl TLS Error 到 MTU 根因分析

在一次日常网络排查中, 我遇到了一个非常典型但又极具迷惑性的网络问题。

![error](https://raw.githubusercontent.com/FonzoSong/NoteImages/master/img/99953991069122fdab2512bc974cab66.png)

环境如下:

- Windows 10
- WSL2 Fedora 43
- 全屋流量经软路由透明代理
- WSL 使用默认 `net` 网络模式
- OpenSSL 3.x
- curl/openssl HTTPS 请求失败

错误现象:

```bash
curl -I https://www.google.com
curl: (35) TLS connect error: error:0A000126:SSL routines::unexpected eof while reading
```

即使跳过证书验证:

```bash
curl -I https://www.google.com -k
```

依然失败。

最终通过修改 WSL 网卡 MTU 恢复网络:

```bash
sudo ip link set dev eth0 mtu 1400
```

本文将完整分析:

- TLS EOF 为什么出现
- WSL2 为什么是重灾区
- MTU 到底是什么
- 为什么透明代理容易导致 TLS 握手失败
- 如何系统性排查类似问题

------

# 一、先理解 MTU

## 1. 什么是 MTU

MTU:

```text
Maximum Transmission Unit
```

即:

```text
单个网络包允许传输的最大大小
```

标准以太网:

```text
MTU = 1500 bytes
```

意味着:

```text
一个 IP 数据包最大为 1500 字节
```

超过 MTU 后:

- 要么分片(Fragmentation)
- 要么丢弃
- 要么触发 PMTU 协商

------

## 2. 为什么“1500”不一定真的是1500

很多人误以为:

```text
网卡显示 1500
= 链路一定支持 1500
```

实际上并不是。

真实链路中可能经过:

| 协议/封装 | 开销 |
| --------- | ---- |
| IPv4      | 20B  |
| TCP       | 20B  |
| PPPoE     | 8B   |
| VXLAN     | 50B  |
| GRE       | 24B  |
| WireGuard | 60B  |
| TUN/TAP   | 若干 |

例如:

```text
1500
- PPPoE 8
- VXLAN 50
- TUN 40
```

真实可用 MTU 可能只剩:

```text
1400 ~ 1450
```

但系统未感知。

这就是问题开始的地方。

------

## 3. PMTU 与 PMTUD

Path MTU:

```text
路径中的最小 MTU
```

而 PMTUD:

```text
Path MTU Discovery
```

负责自动探测链路 MTU。

理论上:

1. 主机发送大包

2. 中间路由发现超 MTU

3. 返回 ICMP:

   ```text
   Fragmentation Needed
   ```

4. 主机降低 MSS/MTU

于是通信恢复正常。

------

## 4. 为什么现实世界 PMTUD 经常失效

因为很多网络设备会:

- 丢弃 ICMP
- 不返回 Fragmentation Needed
- 错误处理 Fragment
- TProxy 不支持分片
- NAT 不重组 fragment

最终形成:

```text
PMTU Black Hole
```

即:

```text
发送方以为1500可达
实际上链路已经炸了
```

这是现代网络最经典的问题之一。

------

# 二、为什么 TLS 最容易暴露问题

普通 HTTP 请求很小:

```text
GET /
```

几十字节。

但 HTTPS 不一样。

TLS1.3 ClientHello 非常大。

包含:

- Cipher Suites
- SNI
- ALPN
- Key Share
- Signature Algorithms
- Supported Groups
- ECH
- Extensions

尤其:

- Google
- Cloudflare
- Github

TLS1.3 握手包通常非常大。

于是:

```text
ClientHello > 实际链路 MTU
```

导致:

```text
IP Fragmentation
```

------

# 三、为什么透明代理容易炸

很多软路由环境:

- OpenClash
- mihomo
- PassWall
- sing-box

使用:

- TUN
- TProxy
- REDIRECT

对 TCP 流量进行接管。

链路通常如下:

```text
WSL2
 ↓
Hyper-V NAT
 ↓
Windows vEthernet
 ↓
软路由 TUN
 ↓
透明代理
 ↓
公网
```

问题在于:

```text
透明代理对分片支持通常很差
```

例如:

- 第一片到了
- 第二片丢失
- NAT 状态不一致
- TUN 不重组 fragment

结果:

TLS 握手被截断。

服务端看到的是:

```text
半个 ClientHello
```

于是直接:

```text
RST
EOF
关闭连接
```

最终 curl 报错:

```bash
unexpected eof while reading
```

------

# 四、为什么 curl -k 无效

很多人第一反应:

```bash
curl -k
```

但这里完全无效。

因为:

```text
-k 只跳过证书校验
```

而问题发生在:

```text
TLS 握手阶段
```

甚至:

```text
证书都还没收到
```

因此 `-k` 无法解决。

------

# 五、为什么 WSL2 是重灾区

WSL2 本质不是“原生 Linux”。

它实际上是:

```text
Hyper-V 虚拟机
```

网络链路包含:

- Linux TCP Stack
- Hyper-V vSwitch
- Windows NAT
- Windows TCP Stack

属于:

```text
双层网络栈
```

而透明代理最怕:

```text
多层 NAT + 多层 MTU
```

因此:

- Docker Desktop
- WSL2
- Kubernetes on Windows

都经常出现 MTU 问题。

------

# 六、最终解决方案

临时方案:

```bash
sudo ip link set dev eth0 mtu 1400
```

效果:

```text
TLS ClientHello 不再分片
```

于是 HTTPS 恢复正常。

------

# 七、更专业的长期方案

## 1. 在 Clash/mihomo 设置 MTU

例如:

```yaml
tun:
  enable: true
  mtu: 1400
```

------

## 2. 启用 MSS Clamping

Linux 路由器:

```bash
iptables -t mangle \
-A FORWARD \
-p tcp --tcp-flags SYN,RST SYN \
-j TCPMSS --clamp-mss-to-pmtu
```

这是企业环境中的标准方案。

------

## 3. WSL 固定 MTU

systemd 服务:

```bash
ExecStart=/sbin/ip link set eth0 mtu 1400
```

开机自动生效。

------

## 4. 使用显式代理替代透明代理

例如:

```bash
export https_proxy=http://192.168.x.x:7890
```

显式代理通常比透明代理稳定很多。

------

# 八、排查此类问题的标准流程

推荐按顺序排查:

## 1. IPv4 测试

```bash
curl -4 -I https://google.com
```

------

## 2. TLS1.2 测试

```bash
curl --tls-max 1.2 -I https://google.com
```

------

## 3. OpenSSL 测试

```bash
openssl s_client -connect google.com:443
```

------

## 4. MTU 测试

```bash
sudo ip link set eth0 mtu 1400
```

------

## 5. tracepath

```bash
tracepath google.com
```

用于发现真实 PMTU。

------

# 九、总结

这次问题的真正根因:

```text
PMTU Discovery Failure
```

即:

```text
路径 MTU 变化
但 ICMP fragmentation needed 未正确返回
```

导致:

```text
TLS ClientHello 分片失败
```

最终表现为:

```text
curl TLS EOF
```

这个问题在:

- WSL2
- Docker
- Kubernetes
- OpenClash
- TUN/TProxy

环境中都非常常见。

很多时候:

```text
网络不是“断了”
而是“大包断了”
```

而 TLS1.3 恰好最容易成为第一个受害者。