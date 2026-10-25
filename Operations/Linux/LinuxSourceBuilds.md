---
title: "Linux 源码编译安装四步法：从依赖准备到 Nginx 上线"
slug: linux-source-compile-install-nginx
summary: "一条 yum 命令装齐编译依赖，再用 configure → make → make install 三段式完成源码安装，最后用进程、端口、页面三层验证结果。"
description: "以 Nginx 为例拆解源码编译安装的完整链路：编译工具与依赖库各自解决什么问题、configure 的检查项与常用参数、Makefile 生成的判定口径、logs 目录缺失导致的启动报错，以及装完如何确认服务真的跑起来了。附源码安装与 yum 安装对比、卸载与开机自启写法。"
---

> **一句话主线**：装依赖 → `./configure` 检查环境并生成 `Makefile` → `make` 编译 + `make install` 安装 → 三层验证。四步中任何一步没通过，都不要往下一步跳。

## 一、准备编译环境：一次性装齐工具与依赖库

源码编译前先把"工具箱"装全，避免编译到一半反复补包。

```bash
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel wget pcre pcre-devel git
```

每个包到底在解决什么问题（面试常追问"为什么要装 pcre"）：

| 依赖包                         | 作用                            | 和 Nginx 的关系                               |
| --------------------------- | ----------------------------- | ----------------------------------------- |
| `gcc-c++`                   | C/C++ 编译器，源码转二进制的核心           | 没有它 `make` 直接报 `no acceptable C compiler` |
| `make`                      | 构建工具，读取 `Makefile` 并按依赖顺序执行编译 | `make` / `make install` 命令的执行者            |
| `zlib` / `zlib-devel`       | 数据压缩库及开发文件                    | 支持 `gzip` 静态资源压缩模块                        |
| `openssl` / `openssl-devel` | SSL/TLS 加密库及开发文件              | 支持 `HTTPS`，即 `--with-http_ssl_module`     |
| `pcre` / `pcre-devel`       | Perl 兼容正则表达式库                 | 支持 `location` 正则匹配与 `rewrite` 重写规则        |
| `libtool`                   | 通用库管理脚本，屏蔽不同系统的建库差异           | 编译动态库时的链接辅助                               |
| `wget`                      | 命令行下载工具                       | 拉取源码包                                     |
| `git`                       | 版本管理                          | 拉取仓库源码、打补丁                                |

> **补充提醒**：笔记里未单独列 `gcc`。装 `gcc-c++` 时通常会连带装上 `gcc`，但极简系统上建议显式确认 `gcc --version` 可用；编译 Nginx 主体用的是 C 编译器 `gcc`，`gcc-c++` 只在需要编译 C++ 组件时才必须。

## 二、运行 `./configure`：检查编译环境、生成 Makefile

去源码目录下，运行 `./configure`，检查是否满足编译环境需求。

```bash
tar -zxf nginx-1.xx.x.tar.gz
cd nginx-1.xx.x
./configure --help        # 先看支持哪些选项，再按需组合
./configure --prefix=/usr/local/nginx \
            --user=www --group=www \
            --with-http_ssl_module \
            --with-http_stub_status_module \
            --without-http_autoindex_module
```

`./configure`（后面可以指定要检查的编译的模块）常用参数：

| 参数                                | 含义                                         |
| --------------------------------- | ------------------------------------------ |
| `--prefix=路径`                     | 指定安装目录，默认 `/usr/local/nginx`               |
| `--user` / `--group`              | 指定工作进程运行的系统用户与用户组                          |
| `--with-xxx`                      | 启用某模块（如 `--with-http_ssl_module` 开启 HTTPS） |
| `--without-xxx`                   | 禁用某模块，缩减体积、加快编译                            |
| `--sbin-path` / `--conf-path`     | 单独指定可执行文件、配置文件路径                           |
| `--pid-path` / `--error-log-path` | 指定 pid 文件与错误日志路径（与第四步的 logs 目录直接相关）        |

**通过判定（原笔记要点，务必记牢）**：

> 只要最后没有出现 `configure: error:` 开头的致命错误，并且成功生成了 `Makefile` 文件，就可以继续 `make`。

自查一下有没有产物：

```bash
ls -l Makefile objs/Makefile   # Makefile 存在才说明配置阶段成功
echo $?                        # 上一条 configure 的退出码，0 为成功
```

常见报错与处理：

| 报错关键词                                                                 | 根因                  | 处理                                             |
| --------------------------------------------------------------------- | ------------------- | ---------------------------------------------- |
| `configure: error: no acceptable C compiler found`                    | 缺 `gcc` / `gcc-c++` | `yum -y install gcc gcc-c++`                   |
| `configure: error: SSL modules require the OpenSSL library`           | 缺 `openssl-devel`   | 装 `openssl openssl-devel` 后重新 configure        |
| `configure: error: the HTTP rewrite module requires the PCRE library` | 缺 `pcre-devel`      | 装 `pcre pcre-devel`，或用 `--without-pcre-jit` 绕行 |
| `configure: error: zlib library not found`                            | 缺 `zlib-devel`      | 装 `zlib zlib-devel`                            |

## 三、`make && make install`：编译并安装

如果安装目录下没有 `logs` 文件夹，先把它建出来——否则 `make install` 后启动会因为找不到日志路径直接失败；随后返回源码目录开始编译安装。

```bash
# 1) 预建日志目录（对应原笔记"没有 logs 文件夹"的处理）
mkdir -p /usr/local/nginx/logs

# 2) 提前建运行账号，避免用 root 跑服务
groupadd -r www && useradd -r -g www -s /sbin/nologin www

# 3) 编译安装：make 编译出二进制，make install 把产物拷到 prefix
make && make install

# 多核机器可并行编译提速，-j 后接 CPU 核数
make -j"$(nproc)" && make install
```

* `make`：按 `Makefile` 把 `.c` 源码逐个编译链接成可执行文件，**这一步只生成文件，不装到系统里**。
* `make install`：把编译产物按目录结构复制到 `--prefix` 指定位置。
* 两步顺序不能颠倒，`make` 失败（出现 `Error 1` / `Error 2`）时**不要**继续 `make install`，回去看是哪段编译报的错。

安装完成后目录长这样：

```bash
tree -L 1 /usr/local/nginx
# conf/  html/  logs/  sbin/
```

## 四、验证是否成功：进程 → 端口 → 页面三层检查

去生成的 `sbin` 目录下运行 `./nginx`，随后做三层确认。

```bash
/usr/local/nginx/sbin/nginx          # 启动（第 1 层：先做配置语法自检）
/usr/local/nginx/sbin/nginx -t       # nginx: the configuration file ... syntax is ok
/usr/local/nginx/sbin/nginx -v       # 版本，确认是新装的那套

ps -ef | grep nginx                  # 第 2 层：master + worker 进程是否在
ss -lntp | grep :80                  # 第 3 层：80 端口是否被 nginx 监听

curl -I http://127.0.0.1             # 应返回 HTTP/1.1 200 OK
```

最后去网页端查看是否成功——浏览器访问 `http://服务器IP`，看到 `Welcome to nginx!` 即完整闭环。

排查顺序：进程没有 → 看 `/usr/local/nginx/logs/error.log`；端口没监听 → 检查 `conf/nginx.conf` 的 `listen` 与防火墙（`firewall-cmd --add-port=80/tcp --permanent && firewall-cmd --reload`）。

## 五、延伸：装完之后还会问你的三件事

**源码安装 vs `yum` 安装怎么选**

| 维度    | 源码编译安装         | `yum` / `apt` 安装            |
| ----- | -------------- | --------------------------- |
| 版本与模块 | 自由指定版本、按需裁剪模块  | 只能用仓库里的版本与预置模块              |
| 安装路径  | 自定义 `--prefix` | 发行版固定布局                     |
| 耗时与难度 | 慢，需自行处理依赖      | 快，依赖自动解决                    |
| 卸载升级  | 手动删目录、手动替换     | `yum remove` / `yum update` |
| 适用场景  | 生产要特定模块、多版本共存  | 快速部署、标准化环境                  |

**让命令随处可用**：把可执行目录加入 `PATH`。

```bash
echo 'export PATH=$PATH:/usr/local/nginx/sbin' >> /etc/profile
source /etc/profile && nginx -v
```

**开机自启**：写一个 systemd 单元，交给 `systemctl` 托管。

```ini
# /etc/systemd/system/nginx.service
[Unit]
Description=Nginx HTTP Server
After=network.target

[Service]
Type=forked
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload && systemctl enable --now nginx
```

**卸载**：`make uninstall` 并非所有软件都支持，源码安装最干净的做法是停服务后直接删除安装目录（`rm -rf /usr/local/nginx`）并清理 `PATH` 与自启文件。

## 六、考点速记（背诵卡）

**口诀**：**一装二配三编译四验证**——装依赖、`configure` 出 `Makefile`、`make` + `make install`、进程端口页面三层查。

| 高频提问                | 秒答                                                                |
| ------------------- | ----------------------------------------------------------------- |
| 源码安装三步是什么？          | `./configure` 生成 `Makefile`；`make` 编译；`make install` 安装           |
| `configure` 干什么？    | 检测编译环境与依赖，按选项生成 `Makefile`                                        |
| 凭什么判断能进下一步？         | 无 `configure: error:` 致命错误 **且** `Makefile` 已生成                   |
| Nginx 为什么要 pcre？    | `location` 正则与 `rewrite` 重写依赖它                                    |
| 为什么要 openssl-devel？ | 编译 `--with-http_ssl_module` 支持 HTTPS                              |
| 为什么 HTTPS 编译失败？     | 只装了 `openssl` 没装 `openssl-devel`，缺头文件                             |
| 启动报日志文件找不到？         | 安装目录缺 `logs`，`mkdir -p` 后重启                                       |
| 怎么确认服务起来了？          | `ps -ef \| grep nginx` → `ss -lntp \| grep :80` → `curl -I` / 浏览器 |
| `-j` 参数作用？          | 多核并行编译提速，`make -j"$(nproc)"`                                      |
