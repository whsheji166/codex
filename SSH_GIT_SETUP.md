# Codex CLI - SSH Git 推送配置教程

## 问题

当你通过 Codex CLI 修改文件后，尝试推送时会失败，因为当前仓库使用 **HTTPS** 协议连接 GitHub，Codex CLI 无法交互式输入密码或 Personal Access Token。

## 解决方案：切换到 SSH

### 第一步：检查现有 SSH Key

Windows 上 SSH key 通常位于 `C:\Users\<用户名>\.ssh\`：

```bash
ls ~/.ssh/
```

你应该能看到：
- `id_ed25519` / `id_ed25519.pub`（推荐使用）
- 或 `id_rsa` / `id_rsa.pub`

如果没有，生成新的：

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

### 第二步：将公钥添加到 GitHub

查看公钥内容：

```bash
cat ~/.ssh/id_ed25519.pub
```

复制输出，然后：
1. 打开 https://github.com/settings/keys
2. 点击 **New SSH key**
3. 粘贴公钥，保存

### 第三步：测试 SSH 连接

```bash
ssh -T git@github.com
```

成功时看到：
```
Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.
```

### 第四步：将仓库远程从 HTTPS 改为 SSH

**方法 A：直接修改（推荐，一次搞定）**

```bash
git remote set-url origin git@github.com:whsheji166/codex.git
```

**方法 B：修改 .git/config 文件**

打开 `.git/config`，找到：

```
[remote "origin"]
    url = https://github.com/whsheji166/codex.git
```

改为：

```
[remote "origin"]
    url = git@github.com:whsheji166/codex.git
```

### 第五步：验证

```bash
git remote -v
```

应该显示 SSH 格式：
```
origin  git@github.com:whsheji166/codex.git (fetch)
origin  git@github.com:whsheji166/codex.git (push)
```

现在尝试推送：

```bash
git push origin main
```

应该可以直接成功，无需输入密码。

## 给 Codex CLI 的指令

当 Codex CLI 在 `/d/codex` 或 `/d/claude code` 目录下工作、需要推送代码时：

1. **先检查远程协议** — 运行 `git remote -v`
2. **如果是 HTTPS** — 先执行 `git remote set-url origin git@github.com:<用户>/<仓库>.git`
3. **然后正常修改、commit、push**

无需每次重新设置，改一次永久生效。

## 其他常见问题

### Q: 提示 `Permission denied (publickey)`
A: 公钥未添加到 GitHub。运行 `cat ~/.ssh/id_ed25519.pub`，去 GitHub Settings > SSH keys 添加。

### Q: 多个 SSH key 怎么办？
A: 在 `~/.ssh/config` 中配置：

```
Host github.com
    HostName github.com
    IdentityFile ~/.ssh/id_ed25519
```

### Q: 还是想用 HTTPS？
A: GitHub 已弃用密码认证，HTTPS 必须使用 Personal Access Token：
1. https://github.com/settings/tokens 生成 token
2. 使用 `git push https://<token>@github.com/whsheji166/codex.git`
