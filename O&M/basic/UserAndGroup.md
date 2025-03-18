### Linux用户与组管理指南

---

#### 一、Linux用户分类
Linux系统中的用户根据权限可分为以下三类：
1. **管理员（超级用户）**  
   - 用户名：`root`  
   - 特权：拥有系统最高权限，可执行任何操作，包括修改系统配置、管理用户及组等。  
   - **注意**：直接使用root账户存在安全风险，建议通过`sudo`命令临时获取权限。

2. **系统用户**  
   - 用途：由系统或应用程序创建，用于运行特定服务（如`apache`、`mysql`）。  
   - 特点：通常不可登录，主目录为`/dev/null`，默认Shell为`/sbin/nologin`。  
   - 示例：`nginx`, `postfix`。

3. **普通用户**  
   - 用途：普通用户登录系统，权限受限，仅能访问自身目录及授权资源。  
   - 特点：需通过`passwd`设置密码，通过`usermod`调整权限。

---

#### 二、用户账号管理

##### 1. 添加新用户（`useradd`）
**语法**：  
```bash
useradd [选项] 用户名
```

**常用选项**：  
| 选项         | 说明           | 示例                           |
| ------------ | -------------- | ------------------------------ |
| `-c "注释"`  | 添加用户描述   | `useradd -c "Developer" alice` |
| `-d 目录`    | 指定主目录     | `useradd -d /opt/project bob`  |
| `-m`         | 自动创建主目录 | `useradd -d /data -m charlie`  |
| `-g 用户组`  | 指定主组       | `useradd -g developers dave`   |
| `-G 组1,组2` | 添加附加组     | `useradd -G sudo,admin eve`    |
| `-s Shell`   | 指定登录Shell  | `useradd -s /bin/zsh frank`    |
| `-u UID`     | 指定用户ID     | `useradd -u 2000 grace`        |

**示例**：  
```bash
# 创建用户sam，主目录为/home/sam，自动创建目录
useradd -d /home/sam -m sam

# 创建用户gem，主组为group，附加组为adm和root，Shell为/bin/sh
useradd -s /bin/sh -g group -G adm,root gem
```

##### 2. 删除用户（`userdel`）
**语法**：  
```bash
userdel [选项] 用户名
```

**常用选项**：  
| 选项 | 说明                 |
| ---- | -------------------- |
| `-r` | 同时删除主目录及邮箱 |

**示例**：  
```bash
# 删除用户sam及主目录
userdel -r sam
```

##### 3. 修改用户（`usermod`）
**语法**：  
```bash
usermod [选项] 用户名
```

**常用选项**：  
| 选项          | 说明          | 示例                          |
| ------------- | ------------- | ----------------------------- |
| `-l 新用户名` | 修改用户名    | `usermod -l alice bob`        |
| `-d 新目录`   | 修改主目录    | `usermod -d /opt/alice alice` |
| `-g 新组`     | 修改主组      | `usermod -g staff alice`      |
| `-aG 组`      | 添加附加组    | `usermod -aG sudo alice`      |
| `-L/-U`       | 锁定/解锁账户 | `usermod -L alice`            |

**示例**：  
```bash
# 修改用户sam的主目录、主组及Shell
usermod -d /home/new_sam -g developers -s /bin/bash sam
```

##### 4. 用户口令管理（`passwd`）
**语法**：  
```bash
passwd [选项] [用户名]
```

**常用选项**：  
| 选项 | 说明                 |
| ---- | -------------------- |
| `-l` | 锁定账户             |
| `-u` | 解锁账户             |
| `-d` | 删除密码（免密登录） |
| `-e` | 设置密码过期时间     |

**示例**：  
```bash
# 为用户sam设置密码
passwd sam

# 锁定用户alice
passwd -l alice

# 允许用户bob下次登录时修改密码
passwd -e bob
```

---

#### 三、用户组管理

##### 1. 添加组（`groupadd`）
**语法**：  
```bash
groupadd [选项] 组名
```

**常用选项**：  
| 选项     | 说明        |
| -------- | ----------- |
| `-g GID` | 指定组ID    |
| `-o`     | 允许重复GID |

**示例**：  
```bash
# 创建组developers，GID为500
groupadd -g 500 developers
```

##### 2. 删除组（`groupdel`）
**语法**：  
```bash
groupdel 组名
```

**示例**：  
```bash
# 删除组test
groupdel test
```

##### 3. 修改组（`groupmod`）
**语法**：  
```bash
groupmod [选项] 组名
```

**常用选项**：  
| 选项        | 说明     |
| ----------- | -------- |
| `-n 新组名` | 修改组名 |
| `-g 新GID`  | 修改GID  |

**示例**：  
```bash
# 将组developers的GID改为501
groupmod -g 501 developers

# 将组名从old改名为new
groupmod -n new old
```

##### 4. 用户组切换
- **`newgrp`命令**：切换到指定组并继承其权限。  
  ```bash
  newgrp developers  # 切换到developers组
  ```

---

#### 四、系统文件详解

##### 1. `/etc/passwd`
格式：  
```plaintext
用户名:密码:UID:GID:注释:主目录:Shell
```

**字段说明**：  
- **密码字段**：`x`表示密码存储在`/etc/shadow`中。  
- **UID/GID**：`0`为root，`1-499`为系统用户，`500+`为普通用户。  
- **示例条目**：  
  ```plaintext
  alice:x:1000:100:Developer:/home/alice:/bin/bash
  ```

##### 2. `/etc/shadow`
格式：  
```plaintext
用户名:加密密码:上次修改时间:最小间隔:最大间隔:警告天数:不活动天数:失效时间:标志
```

- **密码过期策略**：  
  - `最小间隔`：两次修改密码的最短天数（如`7`表示7天内不可重复修改）。  
  - `最大间隔`：密码最长有效天数（如`90`表示90天后必须修改）。  
- **示例条目**：  
  ```plaintext
  alice:$6$rounds=656000$...:20000:0:99999:7:::
  ```

##### 3. `/etc/group`
格式：  
```plaintext
组名:密码:GID:用户列表
```

- **用户列表**：以逗号分隔的组成员，如：`developers:x:500:alice,bob`。

---

#### 五、安全配置

##### 1. 禁用root远程登录
1. 编辑SSH配置文件：  
   ```bash
   sudo vi /etc/ssh/sshd_config
   ```
2. 修改以下参数：  
   ```plaintext
   PermitRootLogin no
   ```
3. 重启SSH服务：  
   ```bash
   sudo systemctl restart sshd
   ```

##### 2. 其他安全建议
- **强制密码策略**：通过`/etc/login.defs`或`pam_pwquality`模块设置密码复杂度。  
- **使用SSH密钥认证**：禁用密码登录，配置`~/.ssh/authorized_keys`。  
- **定期清理无效账户**：使用`lastlog`和`faillog`检查用户登录记录。

---

#### 六、工具与命令速查

| 场景         | 命令                            |
| ------------ | ------------------------------- |
| 查看用户列表 | `cat /etc/passwd | cut -d: -f1` |
| 查看组信息   | `groups 用户名`                 |
| 生成随机密码 | `openssl rand -base64 12`       |
| 查看登录历史 | `last`                          |

---

#### 七、注意事项
1. **谨慎操作root权限**：避免直接使用root账户，优先通过`sudo`执行管理任务。  
2. **备份系统文件**：修改`/etc/passwd`、`/etc/shadow`前建议备份。  
3. **权限最小化原则**：普通用户仅分配必要权限，减少安全风险。

