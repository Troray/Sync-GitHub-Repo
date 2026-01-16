# GitHub 仓库自动同步工具

自动同步上游仓库到你的 GitHub 仓库，支持多仓库、多用户、多平台通知。

## ✨ 功能特性

- ✅ **多仓库同步** - 一个 workflow 管理多个仓库
- ✅ **多用户支持** - 不同仓库可使用不同的 PAT
- ✅ **智能检测** - 只有上游有更新时才同步
- ✅ **并行执行** - 所有仓库同时同步，效率更高
- ✅ **多平台通知** - 支持 Telegram、Discord、钉钉、企业微信、Server酱、邮件
- ✅ **完整历史** - commit 历史与上游完全一致

---

## 📖 使用说明

### 第一步：创建 Personal Access Token (PAT)

因为要跨仓库推送，需要创建一个有权限的 token：

1. 打开 GitHub → 点击头像 → **Settings**
2. 左侧菜单最底部 → **Developer settings**
3. **Personal access tokens** → **Tokens (classic)** → **Generate new token (classic)**
4. 设置：
   - **Note**: `Sync GitHub Repo`
   - **Expiration**: 选择一个合适的时间（或 No expiration）
   - **权限勾选**: `repo`（完整的仓库访问权限）
5. 点击 **Generate token**
6. **复制生成的 token**（只显示一次！）

👉 [快速创建 PAT](https://github.com/settings/tokens/new)



---

### 第二步：配置 Secrets 和 Variables

进入你的仓库 → **Settings** → **Secrets and variables** → **Actions**

#### Secrets（密钥）

| 名称 | 说明 | 示例 |
|------|------|------|
| `TOKEN_USER1` | 用户1 的 GitHub PAT | `ghp_xxxx...` |
| `TOKEN_USER2` | 用户2 的 GitHub PAT（可选） | `ghp_xxxx...` |


---
#### Variables（变量）

| 名称 | 说明 |
|------|------|
| `SYNC_REPOS` | 同步配置（JSON 格式） |
| `NOTIFY_CONFIG` | 通知配置（JSON 格式，可选） |


---
##### SYNC_REPOS 配置示例

```json
[
  {
    "upstream": "https://github.com/owner1/repo1.git",
    "target": "myuser/repo1",
    "branch": "main",
    "token": "TOKEN_USER1"
  },
  {
    "upstream": "https://github.com/owner2/repo2.git",
    "target": "myuser2/repo2",
    "branch": "main",
    "token": "TOKEN_USER2"
  }
]
```
> ⚠️注意：这里的`token`的值填写上方`Secrets（密钥）`中名称

| 字段 | 说明 |
|------|------|
| `upstream` | 上游仓库地址（https 格式） |
| `target` | 目标仓库（格式：`用户名/仓库名`） |
| `branch` | 要同步的分支 |
| `token` | Secrets 中的 Token 名称 |


---
👉 快速直达：

- [添加 Secret](../../settings/secrets/actions/new)
- [添加 Variable](../../settings/variables/actions/new)
>- 添加 Secret: `Settings → Secrets and variables → Actions → New repository secret`
>- 添加 Variable: `Settings → Secrets and variables → Actions → Variables → New repository variable`


---
### 第三步：推送 workflow 文件

```bash
git add .github/workflows/sync-upstream.yml
git commit -m "Add sync workflow"
git push origin main
```

---
### 第四步：手动触发测试

1. 进入仓库 → **Actions** 标签
2. 选择 **Sync Upstream Repositories**
3. 点击 **Run workflow**

---
## 🔔 通知配置（可选）

支持多平台同时发送通知。

### NOTIFY_CONFIG 配置示例

```json
{
  "telegram": {"chat_id": "123456789"},
  "discord": {},
  "dingtalk": {},
  "wecom": {},
  "serverchan": {},
  "email": {"to": "user@example.com", "from": "noreply@example.com"}
}
```

> 只需配置你要使用的平台，不使用的平台可以不写。

### 通知相关 Secrets

| 平台 | Secret 名称 | 获取方式 |
|------|-------------|----------|
| Telegram | `TELEGRAM_BOT_TOKEN` | [@BotFather](https://t.me/BotFather) 创建 Bot |
| Discord | `DISCORD_WEBHOOK` | 服务器设置 → 整合 → Webhook |
| 钉钉 | `DINGTALK_WEBHOOK` | 群设置 → 智能群助手 → 添加机器人 |
| 企业微信 | `WECOM_WEBHOOK` | 群设置 → 添加群机器人 |
| Server酱 | `SERVERCHAN_KEY` | [sct.ftqq.com](https://sct.ftqq.com/) |
| 邮件 | `SMTP_SERVER` | 如 `smtp.gmail.com:465` |
| | `SMTP_USERNAME` | 邮箱用户名 |
| | `SMTP_PASSWORD` | 邮箱密码或应用专用密码 |

---
## 📊 方案对比

| 特性 | 本方案（Actions 自动同步） | GitHub 内置 Sync Fork |
|------|---------------------------|----------------------|
| **同步效果** | ✅ 与上游一致 | ✅ 与上游一致 |
| **commit 历史** | ✅ 完全一致 | ✅ 完全一致 |
| **自动执行** | ✅ 定时自动 | ❌ 需要手动点击 |
| **多仓库支持** | ✅ 支持 | ❌ 不支持 |
| **通知功能** | ✅ 多平台 | ❌ 无 |
| **同步方式** | Force push（强制覆盖） | Merge/Fast-forward |
| **本地修改** | ⚠️ 会被覆盖 | ⚠️ 可能产生冲突 |
| **适用场景** | 纯镜像，不做任何修改 | 可能有自己的改动 |

---
## ⚙️ 高级配置

### 同步所有分支和标签

修改 workflow 中的推送命令：

```bash
# 同步所有分支 + 所有标签（会覆盖/删除目标仓库的其他分支）
git push --mirror --force "$TARGET_URL"
```

### 同步多个指定分支

```bash
git push --force "$TARGET_URL" main:main dev:dev release:release
```

### 修改同步频率

编辑 `cron` 表达式：

```yaml
schedule:
  - cron: '0 0 * * *'    # 每天 UTC 0:00（北京时间 8:00）
  - cron: '0 */6 * * *'  # 每 6 小时
  - cron: '*/30 * * * *' # 每 30 分钟
```

---

## 📝 License

MIT
