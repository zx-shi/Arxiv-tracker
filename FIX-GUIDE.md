# Arxiv-tracker 完整修复指南

## ✅ 已完成的修复

### 1. 修复了核心配置问题

**问题 1：`freshness.since_days: 3650`**
- ❌ 之前：3650 天（10 年）
- ✅ 修复：3 天（CV/AI）、7 天（Finance）

**问题 2：关键词范围太窄**
- ❌ 之前：只匹配 "open vocabulary segmentation"
- ✅ 修复：扩展到 10+ 个关键词，覆盖 CV、AI、Transformer、Zero-shot 等

**问题 3：exclude_keywords 被注释**
- ❌ 之前：不过滤 LLM 论文
- ✅ 修复：启用 exclude_keywords，过滤 "LLM", "Large Language Model", "ChatGPT"

**问题 4：单一 workflow 混合运行**
- ❌ 之前：每天跑一个 workflow，混在一起
- ✅ 修复：创建 3 个独立 workflow，按星期循环

---

## 📦 新的配置结构

### 配置文件

| 文件 | 领域 | 运行时间 |
|-----|-------|---------|
| `config.yaml` | CV | 周一、周四 11:00 |
| `config_ai.yaml` | AI | 周二、周五 11:00 |
| `config_finance.yaml` | Finance | 周三、周六、周日 11:00 |

### GitHub Actions Workflow

| 文件 | 对应配置 | 触发时间 |
|-----|---------|---------|
| `digest_cv.yml` | config.yaml | 周一、周四 19:00 UTC |
| `digest_ai.yml` | config_ai.yaml | 周二、周五 19:00 UTC |
| `digest_finance.yml` | config_finance.yaml | 周三、周六、周日 19:00 UTC |

### 去重状态文件

| 文件 | 领域 |
|-----|-------|
| `.state/seen_cv.json` | CV |
| `.state/seen_ai.json` | AI |
| `.state/seen_finance.json` | Finance |

---

## 🚀 如何推送到 GitHub

### 方法 1：使用 GitHub CLI（推荐）

```bash
cd ~/Documents/GitHub/Arxiv-tracker  # 或你的实际路径

# 1. 验证登录状态
gh auth status

# 2. 推送
git push origin main
```

如果提示权限错误，运行：

```bash
# 重新认证
gh auth login

# 然后再推送
git push origin main
```

### 方法 2：使用 Personal Access Token

如果 GitHub CLI 有问题，创建 Personal Access Token：

1. **访问** https://github.com/settings/tokens
2. **生成新 token**：
   - Name: Arxiv-tracker
   - Expiration: No expiration
   - Scopes: ✅ repo (全选）
3. **复制 token**（只显示一次，立即复制）

然后在终端：

```bash
cd ~/Documents/GitHub/Arxiv-tracker

# 推送时输入 token
git push https://YOUR_TOKEN@github.com/colorfulandcjy0806/Arxiv-tracker.git main
```

### 方法 3：使用 SSH（高级用户）

如果已经配置了 SSH：

```bash
cd ~/Documents/GitHub/Arxiv-tracker

# 验证 SSH
ssh -T git@github.com

# 推送
git push origin main
```

---

## ✅ 推送后的验证步骤

### 1. 检查 GitHub Actions

访问：https://github.com/colorfulandcjy0806/Arxiv-tracker/actions

确认看到 3 个 workflow：
- ✅ arxiv-digest-cv
- ✅ arxiv-digest-ai
- ✅ arxiv-digest-finance

### 2. 手动测试运行

在每个 workflow 页面：
1. 点击 "Run workflow"
2. 在 "send_email" 选择 "true"
3. 点击 "Run workflow" 按钮
4. 查看运行日志

### 3. 检查 GitHub Pages

访问：https://colorfulandcjy0806.github.io/Arxiv-tracker/

应该看到 3 个子目录：
- `/` (CV 领域)
- `/ai/` (AI 领域)
- `/finance/` (Finance 领域)

---

## 🔍 如果还有问题

### 问题 1：Actions 运行失败

**检查点**：
- ✅ 环境变量是否配置（Settings → Secrets and variables）
  - `EMAIL_TO`
  - `EMAIL_SENDER`
  - `SMTP_USER`
  - `SMTP_PASS`
  - `OPENAI_COMPAT_API_KEY`

**验证方法**：
在每个 workflow 的 "Show env (diagnostics)" 步骤中查看输出。

### 问题 2：LLM API 调用失败

**检查点**：
- ✅ `OPENAI_COMPAT_API_KEY` 是否正确（SiliconFlow）
- ✅ `base_url` 是否为 `https://api.siliconflow.cn`
- ✅ `model` 是否为 `Qwen/Qwen2.5-7B-Instruct`

**测试 API Key**：

```bash
# 测试 SiliconFlow API
curl -X POST https://api.siliconflow.cn/v1/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen/Qwen2.5-7B-Instruct",
    "messages": [{"role": "user", "content": "Hello"}],
    "max_tokens": 50
  }'
```

### 问题 3：邮件发送失败

**检查点**：
- ✅ QQ 邮箱是否开启 SMTP
- ✅ 授权码是否正确（不是登录密码）
- ✅ 端口是否正确（465/SSL 或 587/STARTTLS）

**获取 QQ SMTP 授权码**：
1. 登录 QQ 邮箱
2. 设置 → 账户 → POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV 服务
3. 生成授权码

---

## 📊 预期效果

### CV 领域（周一、周四）

- 每次约 30-50 篇论文
- 覆盖：CV、Transformer、Zero-shot 等
- 过滤：LLM、ChatGPT 等

### AI 领域（周二、周五）

- 每次约 40-60 篇论文
- 覆盖：LLM、推理、多模态等
- 过滤：计算机视觉、遥感等

### Finance 领域（周三、周六、周日）

- 每次约 15-25 篇论文
- 覆盖：机器学习、时间序列、量化等
- 过滤：纯计算机视觉等

---

## 🎯 下一步

### 立即执行

1. **推送到 GitHub**（使用上面方法之一）
2. **手动触发一次测试**（每个 workflow）
3. **检查邮件是否收到**
4. **检查 GitHub Pages 是否更新**

### 持续优化

- 调整关键词：根据收到论文的相关性
- 调整时间：根据实际论文数量
- 调整过滤：减少噪音论文

---

## 📞 需要帮助？

如果推送后还有问题，告诉我：

1. 具体的错误信息
2. Actions 日志的截图
3. 收到邮件的情况（收到/没收到/部分收到）

我会帮你进一步诊断！

---

**最后更新**：2026-03-30 21:55
**修复内容**：freshness、关键词、exclude_keywords、workflow 分离
