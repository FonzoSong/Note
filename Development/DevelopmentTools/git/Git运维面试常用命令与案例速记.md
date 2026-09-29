# Git 常用命令与案例速记手册（Linux 运维面试版）

> **用途**：Linux 运维面试的 Git 专项背诵材料。延续《Linux运维面试常用命令速记手册》的「命令 + 必背示例 + 考察点」三栏结构，第十一章另配**可直接口述的完整案例**。
> **标记说明**：★ = 面试必考，要做到"说出命令 + 参数 + 用途"三件套；☆ = 高频加分项。
> **背诵方法**：先记**流转图**（原理层）→ 再记**日常五命令**（肌肉记忆层）→ 最后记**撤销回退**（救火层，也是面试官最爱追问的层）。案例部分按"场景 → 命令序列 → 结果"三段式默写。

---

## 一、先背这张流转图（原理题第一问）

**四个区域、一次流转**：

```text
工作区(Working)  --git add-->  暂存区(Index/Stage)  --git commit-->  本地仓库(.git/HEAD)  --git push-->  远程仓库(origin)
     ^                              |                                     |
     |---------- git checkout / restore 撤销 ----------|--- git pull/fetch 回来 ---|
```

**文件四种状态（必背）**：

| 状态 | 通俗解释 | 进入下一状态的命令 |
|------|---------|------------------|
| Untracked 未跟踪 | Git 还不认识这个新文件 | `git add` |
| Modified 已修改 | 改过了，还没放进待提交清单 | `git add` |
| Staged 已暂存 | 已进清单，等确认提交 | `git commit` |
| Committed 已提交 | 已存进本地档案柜 | `git push` |

**反向操作（追问必考，比正向更容易答错）**：

| 想干什么 | 命令 | 效果落在哪 |
|---------|------|-----------|
| 已暂存 → 退回未跟踪（不再管它） | ★ `git rm --cached 文件名` | 文件还在磁盘，Git 不再跟踪 |
| 已暂存 → 退回已修改（撤销 add） | ★ `git restore --staged 文件名` | 改动保留在工作区 |
| 已修改 → 丢弃工作区改动 | ★ `git restore 文件名`（旧写法 `git checkout -- 文件名`） | 改动**永久丢失** |
| 已提交 → 撤销提交 | ★ `git reset` / `git revert`（见第六章） | 分本地/公共两种场景 |

**底层原理三句话（能答上就加分）**：
- Git 存的是**快照**（每次提交存整份文件状态），SVN 存的是**差异**。
- 四种对象：`Blob`（文件内容）、`Tree`（目录结构）、`Commit`（提交信息 + 父指针）、`Tag`，全部用 SHA-1 哈希唯一标识。
- **分支本质是指向 Commit 的可移动指针**，存在 `.git/refs/heads/`；创建分支不复制文件，所以秒级完成。`HEAD` 是"你当前在哪"的指针。

**Git 与 SVN 区别（几乎必问）**：Git 分布式，每人本地都有完整仓库副本，支持离线提交、分支轻量快速；SVN 集中式，必须联网到中央服务器，分支要复制整个目录。

---

## 二、配置与建仓

| 命令 | 必背示例 | 记忆点 / 考察点 |
|------|---------|----------------|
| ★ `git config` | `git config --global user.name "ops-zhang"`<br>`git config --global user.email "zhang@example.com"` | 提交前**必须先配**，否则提交记录匿名、无法追溯责任人 |
| ★ `git config --list` | `git config --list` | 查看当前生效配置 |
| ☆ 配置层级 | 系统 `--system` → 全局 `~/.gitconfig` → 仓库 `.git/config` | 就近覆盖：仓库级优先，面试问"只改一个仓库的提交身份"答 `git config user.name`（不加 --global） |
| ★ `git init` | `git init` / `git init --initial-branch=main` | 生成 `.git` 隐藏目录即建仓完成；新仓库默认分支现多为 `main` |
| ★ `git clone` | `git clone git@gitlab.example.com:ops/ansible.git` | 克隆**自动关联远程并命名为 origin**，无需再 `remote add` |
| ☆ 浅克隆 | `git clone --depth 1 仓库地址` | 只拉最新一次提交，CI/CD 构建机上省时间省空间（运维高频） |
| ☆ 关联远程 | `git remote add origin URL` / `git remote -v` / `git remote set-url origin 新URL` | `-v` 查看；`set-url` 换地址（如仓库迁移、HTTP 换 SSH） |
| ★ SSH 免密 | `ssh-keygen -t ed25519 -C "邮箱"` → `cat ~/.ssh/id_ed25519.pub` → 贴到 GitLab/GitHub | 运维拉代码标配；`ssh -T git@gitlab.example.com` 验证是否连通 |

---

## 三、日常五命令（肌肉记忆，覆盖 90% 操作）

| 命令 | 必背示例 | 记忆点 / 考察点 |
|------|---------|----------------|
| ★ `git status` | `git status` / `git status -s` | 每天用几十次；`-s` 精简输出，`??` 未跟踪、`M` 已修改、`A` 新增、`D` 删除、`UU` 冲突 |
| ★ `git add` | `git add nginx.conf` / `git add .` / `git add -u` | `.` 当前目录起的全部变更；`-A` 整个仓库全部变更；`-u` **只处理已跟踪文件**（不含新文件） |
| ★ `git commit` | `git commit -m "fix: 修正 nginx 超时配置"` | `-a` 自动暂存**已跟踪**文件的改动再提交（跳不过新文件）；提交信息要规范化 |
| ☆ `git commit --amend` | `git commit --amend -m "新说明"` | 改最后一次提交；**已 push 的提交别 amend**，会让别人历史错乱 |
| ★ `git diff` | `git diff`（工作区 vs 暂存区）<br>`git diff --staged`（暂存区 vs 仓库，等价 `--cached`）<br>`git diff 分支1 分支2` | 面试常问"怎么看待暂存与仓库的差异"→ `git diff --staged` |
| ☆ `git rm` / `git mv` | `git rm 文件` / `git mv a.conf b.conf` | 删除/重命名并同步暂存区；`git rm --cached` 是取消跟踪 |

**提交信息规范（追问"你们怎么写 commit message"）**：`feat:` 新功能、`fix:` 修 bug、`docs:` 文档、`style:` 格式、`refactor:` 重构、`perf:` 性能、`test:` 测试、`chore:` 构建/杂项。运维场景示例：`chore: 更新生产 nginx 证书到期时间`。

---

## 四、查历史、查责任人

| 命令 | 必背示例 | 记忆点 / 考察点 |
|------|---------|----------------|
| ★ `git log` | `git log --oneline -10` | 精简看最近 10 条，最常用 |
| ★ 分支图谱 | `git log --graph --oneline --all` | 一眼看清合并关系，排查"这代码什么时候合进来的" |
| ☆ 按人/时间筛 | `git log --author=ops-zhang --since=3.days` | 排查"最近三天谁改了配置" |
| ☆ 看改动量 | `git log --stat -3` / `git log -p -1` | `--stat` 统计文件行数；`-p` 直接看 diff 内容 |
| ★ `git show` | `git show a1b2c3d` | 看某次提交的元信息 + 具体内容变更 |
| ★ `git blame` | `git blame nginx.conf` | **逐行**看是谁哪次提交改的，事故追责神器 |
| ☆ `git reflog` | `git reflog` | HEAD 移动全记录，找回误删提交/分支的唯一救命命令 |

---

## 五、分支管理

| 操作 | 必背示例 | 考察点 |
|------|---------|--------|
| ★ 查看 | `git branch`（本地，`*` 是当前）<br>`git branch -a`（含远程）<br>`git branch -vv`（含上游跟踪关系） | `-r` 只看远程分支；`-vv` 能看出本地分支在跟踪谁，答上来算加分 |
| ★ 创建 | `git branch dev` | **只创建不切换**，仍在原分支（易错点） |
| ★ 切换 | `git checkout dev` / `git switch dev`（2.23+） | 切换前须提交或 stash，否则带脏改动报错 |
| ★ 创建并切换 | `git checkout -b feature/ssl` / `git switch -c feature/ssl` | 最常用的组合动作 |
| ★ 合并 | `git checkout main` → `git merge feature/ssl` | 先切到**目标**分支再 merge，方向别搞反 |
| ☆ 变基 | `git rebase main` | 历史拉成直线，**禁止对已推送公共分支用** |
| ★ 删本地 | `git branch -d dev`（安全）/ `-D`（强删） | 不能删当前所在分支，须先切走 |
| ★ 删远程 | `git push origin --delete dev` | 合并完清理废弃分支，团队协作规范 |

**merge 与 rebase 对比（必背）**：

| 维度 | merge | rebase |
|------|-------|--------|
| 结果 | 生成合并提交，历史呈分叉网状 | 提交"平移"到目标分支末端，历史一条直线 |
| 历史 | 完整保留、可追溯分支来源 | **改写提交历史**（哈希变化） |
| 优点 | 安全、不改写 | 日志整洁，便于 review 与 bisect |
| 缺点 | 合并记录多了像麻花 | 误用会坑到全组 |
| 规范 | 主干合并用 merge | 个人分支整理提交用 rebase |

---

## 六、撤销与回退（救火高发区，重中之重）

| 命令 | 必背示例 | 效果与风险 |
|------|---------|-----------|
| ★ `git restore` | `git restore nginx.conf` | 丢工作区改动，🟢 安全常用 |
| ★ `git restore --staged` | `git restore --staged nginx.conf` | 取消暂存，改动保留，🟢 安全 |
| ★ `git reset --soft` | `git reset --soft HEAD~1` | 撤销 1 次提交，改动**留在暂存区**（适合重写 commit message） |
| ★ `git reset --mixed` | `git reset HEAD~1`（**默认**） | 撤销提交 + 取消暂存，改动**留在工作区** |
| ★ `git reset --hard` | `git reset --hard HEAD~1` | 提交、暂存、工作区改动**全丢**，🔴 高危 |
| ★ `git revert` | `git revert a1b2c3d` / `git revert HEAD` | 生成一条**反向新提交**，不改历史，🟢 公共分支唯一正解 |
| ☆ `git cherry-pick` | `git cherry-pick a1b2c3d` | 跨分支挑某一笔提交过来（把 hotfix 同步到 dev） |
| ☆ 取消合并 | `git merge --abort` | 合并冲突处理不动时退回合并前状态 |

**reset 三模式记忆法（面试官最爱追问，务必说清"三个区域各动了什么"）**：

| 模式 | HEAD | 暂存区 | 工作区 |
|------|------|--------|--------|
| `--soft` | 回退 | 保留 | 保留 |
| `--mixed`（默认） | 回退 | 清空 | 保留 |
| `--hard` | 回退 | 清空 | **清空** |

**选择原则（一句话答对）**：**没推出去的本地提交用 `reset`，已经推出去的公共提交用 `revert`。** `reset --hard` 和 `push -f` 是 Git 里真正不可逆的两个操作，执行前三思。

---

## 七、远程协作

| 命令 | 必背示例 | 考察点 |
|------|---------|--------|
| ★ `git push` | `git push origin dev` | 把本地提交推到远程分支 |
| ★ 首次推送 | `git push -u origin dev` | `-u` 建立上游跟踪，之后可裸用 `git push` / `git pull` |
| ★ `git fetch` | `git fetch origin` | **只拉不合并**，先看别人改了什么再决定，更安全 |
| ★ `git pull` | `git pull` / `git pull --rebase` | 等价 `fetch + merge`；`--rebase` 保持历史线性 |
| ☆ `git remote show` | `git remote show origin` | 看远程分支与跟踪关系全景 |
| ☆ 强推 | `git push --force-with-lease origin dev` | 比 `-f` 安全（远端有别人新提交会拒绝），个人分支修历史才用 |

**fetch 与 pull 区别（必问）**：`fetch` 只把远程更新下载到本地仓库（更新 `origin/分支`），**不碰你的工作区和当前分支**，可先 `git diff origin/main` 检查；`pull` = `fetch` + `merge`，拉下来直接合并，可能立刻触发冲突。生产机上建议 `fetch` 看一眼再决定。

---

## 八、stash / tag / bisect / .gitignore

| 命令 | 必背示例 | 考察点 |
|------|---------|--------|
| ★ `git stash` | `git stash push -m "nginx 改动做到一半"` | 临时存起未提交改动，好去切分支救火（`save` 是旧写法） |
| ★ `git stash list` | `git stash list` | 查看栈，`stash@{0}` 是最新 |
| ★ `pop` / `apply` | `git stash pop` / `git stash apply stash@{1}` | **`pop` 恢复并删除记录，`apply` 恢复但保留记录**（易错点） |
| ☆ `git stash -u` | `git stash -u` | 连**未跟踪**的新文件一起存 |
| ☆ `git stash drop` / `clear` | `git stash drop stash@{0}` / `git stash clear` | 删单条 / 清空 |
| ★ `git tag` | `git tag v1.2.0`（轻量）<br>`git tag -a v1.2.0 -m "发布说明"`（附注，推荐） | 版本发布打点；附注标签含作者时间等元数据 |
| ★ 推送标签 | `git push origin v1.2.0` / `git push --tags` | 标签**不会随 push 自动上去**，易错 |
| ☆ 删标签 | `git tag -d v1.2.0`；远程 `git push origin --delete v1.2.0` | 本地远程各删一次 |
| ☆ `git bisect` | `git bisect start` → `git bisect bad` → `git bisect good 标签` | 二分定位"从哪个提交开始坏的"，答上来很加分 |
| ★ `.gitignore` | `*.log`、`data/`、`.env`、`*.pid` | **只对未跟踪文件生效**；已提交过的文件加了规则也不会自动忽略，须 `git rm --cached 文件` 再提交 |
| ☆ `git check-ignore` | `git check-ignore -v data/mysql.sock` | 排查"为什么这文件还在被跟踪"，能看出哪条规则起作用 |

---

## 九、冲突解决标准五步（口头题「你遇到过冲突吗，怎么解决」）

**成因一句话**：两个分支改了**同一文件的同一区域**，Git 无法自动判定保留谁，标记为 `UU`（unmerged），需人工介入。

```bash
# 第 1 步：定位冲突文件
git status                       # 看 Unmerged paths 下的文件

# 第 2 步：打开文件，找到三段冲突标记
# <<<<<<< HEAD            ← 当前分支的内容
# listen 80;
# =======                 ← 分界线
# listen 8080;            ← 传入分支（origin/main）的内容
# >>>>>>> feature/change-port

# 第 3 步：人工决策改文件（保留正确内容 + 删光所有标记行）
vim nginx.conf

# 第 4 步：标记已解决并完成合并
git add nginx.conf
git commit                        # 自动带合并说明，不用 -m 也行

# 第 5 步：同步远程
git push

# 逃生通道：处理不动就退回合并前
git merge --abort
```

**避坑要点**：① 切分支前先 `stash`，别把冲突带到别的分支；② 开发前先 `pull` 最新代码，别攒到最后一次性合；③ 合并前只 `add` 解决好的文件，别顺手 `git add .` 把半成品带进去。

---

## 十、必背问答（口头题秒答卡）

| 面试提问 | 秒答 |
|---------|------|
| Git 和 SVN 的区别？ | Git 分布式、本地完整副本、可离线提交、分支轻量；SVN 集中式、必须联网、分支复制整目录 |
| 工作区/暂存区/仓库分别是什么？ | 编辑区 / 待提交清单 / `.git` 历史库；`add` 入暂存、`commit` 入仓库、`push` 上远程 |
| `git add .` 和 `-A` 区别？ | 作用范围不同：`.` 是当前目录及以下，`-A` 是整个仓库；仓库根目录下等效。`-u` 只管已跟踪文件 |
| 已推送的错误提交怎么撤？ | `git revert 提交号` 生成反向提交，不改历史，协作安全；`reset` + `push -f` 会改写公共历史，禁止 |
| `reset` 三种模式区别？ | soft 只退 HEAD；mixed 再清暂存区；hard 连工作区一起丢（高危） |
| `fetch` 和 `pull` 区别？ | fetch 只下载不合并；pull = fetch + merge |
| merge 和 rebase 区别？ | merge 保留分叉历史、生成合并提交；rebase 平移提交、历史线性但**改写历史** |
| 什么时候能 rebase？ | 只在自己的、未推送的分支上整理提交；已推送的公共分支严禁（黄金法则） |
| 分支和标签有什么区别？ | 分支是**可移动**指针会跟着新提交走；标签是**固定**指向某提交的静态指针，用于版本发布 |
| 误删提交/回退过头怎么恢复？ | `git reflog` 找历史 HEAD 位置 → `git reset --hard HEAD@{序号}`（或提交哈希） |
| 冲突标记是什么意思？ | `<<<<<<<` 到 `=======` 是当前分支内容，`=======` 到 `>>>>>>>` 是传入分支内容，人工取舍后删标记 |
| 提交前发现漏了一个文件？ | 加进去后 `git commit --amend --no-edit`（未推送时可用） |
| 想切分支但手头改动没完成？ | `git stash` 存现场，切过去干完活回来 `git stash pop` |
| 生产机上正确的更新姿势？ | `git fetch` → `git diff origin/main` 看变化 → `git pull` → 服务 `reload` 前先 `nginx -t` |

---

## 十一、完整案例实战（可直接口述的八段）

### 案例 1 · 从零建仓并推到 GitLab（运维建配置库全流程）

```bash
mkdir -p /data/gitops/nginx-conf && cd /data/gitops/nginx-conf
git init
cat > .gitignore <<'EOF'
*.log
*.pid
data/
EOF
cp /etc/nginx/nginx.conf .
git config user.name "ops-zhang"; git config user.email "zhang@example.com"
git add . && git commit -m "chore: 初始化生产 nginx 配置基线"
git remote add origin git@gitlab.example.com:ops/nginx-conf.git
git push -u origin main
git remote -v                       # 确认关联：fetch/push 两行 URL
```

**考察点**：`init` 后必须先 `commit` 才有内容可推；`-u` 建立跟踪后，后续一句 `git push` 即可。

### 案例 2 · commit message 写错 / 漏加文件（本地未推送）

```bash
git log --oneline -3
echo "worker_processes 4;" >> nginx.conf
git add nginx.conf
git commit --amend --no-edit        # 补文件进上一笔，说明不变
git commit --amend -m "fix: 修正 nginx 超时配置并调整 worker 数"   # 只改说明
```

**考察点**：`--amend` 是**替换**而非新增，旧提交会被抛弃；已 push 就不能这么干（别人历史会错乱，须改走 revert）。

### 案例 3 · 线上配置已推送，发现是坏的（公共分支安全回滚）

```bash
git log --oneline -5                # 找到坏提交 a1b2c3d
git revert a1b2c3d                  # 生成反向提交，历史不丢
git log --oneline -3                # 顶部多出一条 Revert "..."
git push origin main
systemctl reload nginx              # 回滚后落到服务生效
```

**多笔连续回滚**：`git revert <最旧提交>^..<最新提交>`（区间会按从旧到新逐笔反向）；只想回滚某个文件的某次改动而不影响其他文件：`git checkout 提交号 -- nginx.conf`（新写法 `git restore --source=提交号 nginx.conf`）后重新提交。

### 案例 4 · 本地开发线乱了，三种 reset 亲手体验

```bash
git log --oneline -3                # 假设最新 3 笔为 d3 d2 d1
# 场景 A：只提交说明写错，内容想留着重新提交
git reset --soft HEAD~1 && git status -s     # 改动还在暂存区
git commit -m "feat: 新增 vhost 站点配置"
# 场景 B：想重新挑选要提交的文件
git reset --mixed HEAD~1 && git status -s    # 改动退回工作区（未暂存）
git add ssl.conf && git commit -m "feat: 增加 443 站点"
# 场景 C：这批改动彻底不要了
git reset --hard HEAD~2 && git status        # 干净，改动全丢
git reflog                                  # 后悔了？还能按 HEAD@{n} 捞回来
```

**考察点**：说清"三个区域分别被怎么动"，这是面试官判断你是真会用还是背题的分水岭。

### 案例 5 · 误删分支 / reset 过头，用 reflog 救命

```bash
git branch -D hotfix/pay            # 手滑删了还没合并的分支
git reflog                          # 输出形如：a1b2c3d HEAD@{7}: checkout: moving from main to hotfix/pay
git branch hotfix/pay a1b2c3d       # 用找回的哈希重建分支
# 或 reset --hard 过头：
git reflog
git reset --hard HEAD@{3}
```

**考察点**：只要提交过，本地就还留有痕迹（默认保留 90 天），`reflog` 是本地操作流水，别人看不见 —— 这是它和 `git log` 的本质区别。

### 案例 6 · 改配置改一半，线上突然要救火（stash 切换）

```bash
git status -s                       # 工作区有未提交改动
git stash push -u -m "改一半的 gzip 配置"
git checkout main && git pull
vim nginx.conf                      # 处理线上问题
git add nginx.conf && git commit -m "fix: 修复线上 502" && git push
git checkout feature/gzip
git stash list                      # stash@{0} 还在
git stash pop                       # 恢复现场并清掉记录
```

### 案例 7 · 团队协作标准流（feature 分支 + MR + 清理）

```bash
git checkout main && git pull                     # 先取最新，避免后期大冲突
git checkout -b feature/add-vhost
# ... 开发、多次提交 ...
git fetch origin && git rebase origin/main        # 个人分支变基，历史保持线性
git push -u origin feature/add-vhost
# 网页端提 MR，指定 reviewer；评审意见改完继续 push 到同一分支
git checkout main && git merge feature/add-vhost
git push && git branch -d feature/add-vhost && git push origin --delete feature/add-vhost
```

**考察点**：`main` 是生产稳定分支，**禁止直接开发提交**；分支命名 `feature/`、`hotfix/`、`release/`；网页端可选 Squash merge 把多笔提交压成一笔，主干日志更干净（但会丢失分支内细粒度历史）。

### 案例 8 · 生产目录直接跑 `git pull` 报冲突（运维真实踩坑）

```text
error: Your local changes to the following files would be overwritten by merge:
        nginx.conf
```

```bash
git diff nginx.conf                 # 先看本地被手工改了什么（常见：有人直接上服务器改配置）
git stash push -m "生产机上的临时手改"
git pull
git diff stash@{0}                  # 对比手改与仓库版本的差异，走流程把改动收编回仓库
```

**正确处理姿势（面试可作为"你怎么看配置漂移"的答法）**：生产机上的配置应以仓库为唯一可信源，本地临时改动要么提交回仓库、要么回滚；长期方案是改用 Ansible/SaltStack 下发，禁止 SSH 上机直接改，从根上消灭这类冲突。

---

## 十二、运维场景组合拳（一句话串起来）

| 工作场景 | 命令串联 |
|---------|---------|
| 每日开发 | `git status` → `git pull` → 改文件 → `git diff` → `git add` → `git commit -m` → `git push` |
| 配置上线 | `git checkout release/v1.3` → `git pull` → `nginx -t` → `systemctl reload nginx` |
| 线上回滚 | `git log --oneline` 定位版本 → `git revert` 或 `git checkout 旧标签 -- 文件` → push → 重载服务 |
| 排查是谁改坏的 | `git log --oneline -- 文件` → `git show 提交号` → `git blame 文件` |
| CI/CD 触发 | 代码 `push` 到 Git 仓库 → 自动触发流水线构建/测试/部署，仓库为唯一可信源 |
| 服务器首次拉库 | `ssh-keygen` → 公钥加到 GitLab → `git clone --depth 1 URL` |

**一页流记忆顺序（按手指顺序背）**：
`config` → `init`/`clone` → `status` → `add` → `commit` → `log`/`diff` → `branch -a`/`checkout -b` → `merge`/`rebase` → `fetch`/`pull`/`push -u` → `reset`/`revert`/`restore` → `stash` → `tag` → `reflog`
