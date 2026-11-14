# Ubuntu 上把本地项目推上 GitHub 全流程

> 环境：本机已配好 SSH 密钥 `~/.ssh/id_ed25519`，已绑定 GitHub 账号 **LChi258**，
> `~/.ssh/config` 已改为走 `ssh.github.com:443`（22 端口被封时的标准解法）。
> 也就是说：**认证这块已经不用再管了，下面从建仓库开始。**

```mermaid
flowchart TD
    A([开始：本地已有项目文件夹]) --> B[打开终端并 cd 进项目目录]
    B --> C[git init<br/>把普通文件夹变成 Git 仓库]
    C --> D[创建 .gitignore<br/>只写要忽略的垃圾文件<br/>不写已经写好的代码]
    D --> E{核对 .gitignore<br/>git status 看不到 node_modules/编译产物?}
    E -- 否，看到不该提交的 --> D
    E -- 是 --> F[git add .<br/>把文件放进暂存区]
    F --> G[git status<br/>再确认一次清单]
    G --> H[git commit -m "初始提交"<br/>本地第一次存档]
    H --> I[去 GitHub 网页端<br/>点 New repository<br/>建一个【空仓库】，不要勾 README/.gitignore/License]
    I --> J[复制 SSH 地址<br/>git@github.com:LChi258/仓库名.git]
    J --> K[git remote add origin 那个地址<br/>把本地仓库和远端关联]
    K --> L[git branch -M main<br/>统一主分支叫 main]
    L --> M[git push -u origin main<br/>首次推送并记住上游]
    M --> N{推送成功?}
    N -- 端口/超时 --> O[检查 ssh -T git@github.com<br/>是否 443 走通] --> M
    N -- rejected / 有远端提交 --> P[git pull --rebase origin main<br/>合并后再推] --> M
    N -- 是 --> Q([完成：以后只需 git add/commit/push])
```

## 分步说明（照着敲）

### 0. 进目录

```bash
cd ~/你的项目文件夹
```

### 1. 初始化仓库

```bash
git init
```

> 你的 `~/.gitconfig` 里已有 `init.defaultBranch = main`，所以默认分支直接就是 `main`，
> 不需要再手动改分支名。如果老版本 git 提示 master，再补 `git branch -M main`。

### 2. 写 `.gitignore`（**这步最重要，写在 add 之前**）

```bash
cat > .gitignore <<'EOF'
# Python
__pycache__/
*.py[cod]
.venv/
venv/

# Node
node_modules/
dist/
build/

# 编辑器 & 系统
.obsidian/workspace*
.vscode/
.DS_Store

# 密钥与本地配置
.env
*.pem
EOF
```

要点：

- **先写 `.gitignore` 再 `git add`**，否则垃圾文件已经被 Git 跟踪，后面再忽略就不生效了。
- 不要无脑抄 `*` 之类的通配，会把源码也忽略掉。

### 3. 暂存 + 检查 + 提交

```bash
git add .
git status              # ← 养成习惯：这一步看清单，别跳过
git commit -m "初始提交：项目骨架"
```

### 4. 在 GitHub 网页上建空仓库

1. 打开 <https://github.com/new>
2. Repository name 填仓库名，比如 `my-project`
3. **Public / Private** 按需选
4. ⚠️ **不要勾** "Add a README file"、"Add .gitignore"、"Choose a license"
   —— 建成了非空仓库，首次 push 会因为历史不一致被拒绝，白折腾一轮
5. 点 **Create repository**

### 5. 关联并推送

```bash
git remote add origin git@github.com:LChi258/my-project.git
git push -u origin main
```

`-u` 的作用：把本地 `main` 和 `origin/main` 绑定为上下游关系，以后直接 `git push` / `git pull` 就行，不用再写后面的参数。

### 6. 验证

```bash
git remote -v           # 应显示 origin 的 fetch/push 两行
git log --oneline        # 应看到你的初始提交
git status               # 应显示 "working tree clean"
```

刷新 GitHub 页面，文件应该都在了。

---

## 常见报错对照表

| 报错信息 | 原因 | 解决 |
|---|---|---|
| `Connection refused` 连 22 端口 | GitHub 22 端口被网络阻断 | 本机**已解决**：`~/.ssh/config` 走 `ssh.github.com:443` |
| `Host key verification failed` | `known_hosts` 里没有 `[ssh.github.com]:443` | `ssh -o StrictHostKeyChecking=accept-new -T git@github.com`（本机已加过） |
| `Permission denied (publickey)` | 密钥没绑到账号 / 用错了密钥 | `ssh -T git@github.com` 测，正常会回 `Hi LChi258!` |
| `remote: Repository not found` | 仓库名拼错，或 Private 仓库无权限 | 核对 `git remote -v` 的地址 |
| `Updates were rejected because the remote contains work...` | 远端建仓库时勾了 README 等，不是空的 | `git pull --rebase origin main` 后再 `git push` |
| `fatal: remote origin already exists` | 之前已经加过 origin | `git remote set-url origin <新地址>` 或 `git remote remove origin` 后重加 |
| `src refspec main does not match any` | 还没有任何 commit，或分支名不对 | 先 `git commit`，再 `git branch -M main` |

---

## 日常循环（建完之后每天就是这么用）

```mermaid
flowchart LR
    a[改代码] --> b[git status 看改了什么]
    b --> c[git add .]
    c --> d[git commit -m "说明这次干了啥"]
    d --> e[git push]
```

三个命令记住就够：

```bash
git add .
git commit -m "提交说明"
git push
```

## 当前本机状态速查

| 项 | 值 |
|---|---|
| Git 版本 | 2.56.0 |
| user.name | `LChi258` |
| user.email | `237557995+LChi258@users.noreply.github.com` |
| SSH 私钥 | `~/.ssh/id_ed25519`（已通过 GitHub 认证） |
| 连接方式 | `ssh.github.com:443`（因 22 端口不通） |
| 凭据缓存 | 无（`credential.helper` 为空），全部走 SSH |
| 工作目录 | `~/文档/deepseek-harness/default-workspace`（**当前为空、且不是 Git 仓库**） |