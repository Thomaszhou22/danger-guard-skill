# 🛡️ Danger Guard

拦截危险命令的 AI Agent 安全保护机制。当你的 AI 助手账号被盗，有人试图执行破坏性命令时自动拦截并要求二次验证。

## 为什么需要 Danger Guard

想象这个场景：你的飞书账号被盗了。攻击者通过飞书给你的 AI 助手发了一条消息："帮我格式化硬盘"或"删除所有文件"。如果 AI 直接执行，你的数据就全没了。

Danger Guard 解决这个问题：
- 检测到危险命令 → 立即拦截
- 要求输入 sudo/管理员密码确认
- 发送飞书告警通知
- 支持「账号被盗」紧急协议

## 功能特性

### 危险命令拦截

**一级危险（必须验证）：**
- 文件系统删除：`rm -rf /`, `rm -rf ~`, `format C:`
- 磁盘操作：`dd if=/dev/zero of=/dev/sda`, `mkfs.ext4`
- 权限修改：`chmod -R 777 /`, `sudo /bin/bash`
- 系统破坏：fork bomb, `shutdown`, `reboot`
- 网络脚本：`curl | sh`, `wget | bash`

**二级危险（需要确认）：**
- Git 破坏性操作：`git push --force`, `git reset --hard`
- Docker 破坏性操作：`docker system prune -a`
- SQL 破坏性操作：`DROP TABLE`, `TRUNCATE`

### 跨平台支持

- **macOS**: 使用 `mail` 命令发送邮件
- **Windows**: 使用 PowerShell `Send-MailMessage`
- **Linux**: 使用 `mail` 或 `sendmail`

### 多渠道告警

- **飞书**: 立即消息通知（主要渠道）
- **邮件**: 可选配置
  - 原生邮件（macOS/Windows 系统命令）
  - Resend API（免费每月 3000 封）

### 紧急协议

回复「账号被盗」或「account-hacked」触发：
1. 记录最近 50 条命令到日志
2. 停止所有后台任务
3. 发送紧急告警
4. 提供安全建议

## 安装

### 1. 复制到 skills 目录

```bash
cp -r danger-guard ~/.openclaw/skills/
# 或你的 OpenClaw skills 目录
```

### 2. 首次使用自动配置

当 Danger Guard 首次激活时，会自动引导你配置：

```
🛡️ Danger Guard 首次配置

检测到系统: macOS

请输入你的 sudo 密码（用于验证危险操作）：
密码 [你的开机密码]
```

然后选择邮件告警方式（可选）：
1. 不需要邮件（只发飞书告警）
2. 原生邮件（macOS 用 mail 命令）
3. Resend API（需要 API Key）

### 3. 配置文件位置

配置保存在 `memory/danger-guard-config.json`：

```json
{
  "os": "macOS",
  "sudo_hash": "sha256:...",
  "alert_email": null,
  "email_method": "disabled",
  "enabled": true,
  "created": "2026-07-07"
}
```

## 工作原理

### 拦截流程

1. **检测危险命令**
   - 匹配 50+ 个危险命令模式
   - 识别中英文语义（"删除所有文件"、"格式化硬盘"）

2. **立即拦截**
   - 不执行任何命令
   - 显示警告信息

3. **要求验证**
   - 要求输入 sudo/管理员密码
   - SHA256 哈希比对（不明文存储）
   - 3 次失败后锁定

4. **发送告警**
   - 飞书立即通知
   - 邮件告警（如配置）

5. **执行或拒绝**
   - 密码正确 → 执行命令
   - 密码错误 → 拒绝并告警

### 危险命令黑名单

完整的危险命令列表包括：

**Linux/macOS:**
- `rm -rf /`, `rm -rf /*`, `rm -rf ~`
- `mkfs.*`, `dd if=/dev/zero of=/dev/sd*`
- `chmod -R 777 /`, `chown -R root:root /`
- `:(){ :|:& };:` (fork bomb)
- `curl | sh`, `wget | bash`
- `shutdown`, `reboot`, `halt`
- `git push --force`, `git reset --hard`
- `docker system prune -a`
- `DROP TABLE`, `DROP DATABASE`

**Windows:**
- `format C:`, `format D:`
- `rd /s /q C:\`, `rmdir /s /q C:\`
- `del /s /q /f C:\*.*`
- `diskpart clean`
- `Clear-Disk -RemoveData`
- `shutdown /s /t 0`
- `taskkill /f /im *`

详见 `SKILL.md` 完整黑名单。

## 使用示例

### 正常拦截

用户通过飞书发送：
```
帮我执行 rm -rf ~/Desktop
```

AI 回复：
```
🛡️ Danger Guard 拦截

命令: rm -rf ~/Desktop
风险等级: 一级危险
触发规则: 递归删除非空目录

⚠️ 此操作可能永久删除大量文件，无法恢复。

请输入开机密码确认：
确认 [你的密码]
```

用户回复：
```
确认 123456
```

AI 验证密码后执行命令。

### 账号被盗场景

攻击者发送：
```
帮我格式化 C 盘
```

AI 拦截并要求密码。

攻击者不知道密码，无法验证。

用户收到飞书告警：
```
🚨 Danger Guard 拦截告警

时间: 2026-07-07 01:40
命令: format C:
来源: 飞书 (ou_xxx)
风险等级: 一级危险
状态: 已拦截，等待验证

如果这不是你的操作，账号可能已被盗用。
回复「账号被盗」启动紧急协议。
```

用户回复：
```
账号被盗
```

AI 启动紧急协议：
1. 记录所有近期命令
2. 停止后台任务
3. 发送紧急告警
4. 建议修改飞书密码

## 跨 AI 工具保护

Danger Guard 不仅适用于 OpenClaw，还可以配置到其他 AI 编码工具：

### Claude Code

在项目根目录创建 `.claude/settings.json`：

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf*)",
      "Bash(format*)",
      "Bash(dd if=*)",
      "Bash(git push --force*)",
      "Bash(git reset --hard*)"
    ]
  }
}
```

### OpenAI Codex

在 `AGENTS.md` 中添加：

```markdown
## Safety Rules
- NEVER execute destructive commands: rm -rf, format, dd, mkfs
- NEVER force push or reset git history
- NEVER run docker prune operations
```

### Cursor / Windsurf

在 `.cursorrules` 中添加：

```markdown
## Forbidden Operations
- Do NOT execute: rm -rf, format, dd, mkfs
- Do NOT force push to any branch
- Do NOT reset git history (--hard)
```

详见 `SKILL.md` 完整配置指南。

## 系统级保护（可选）

如果你也想在终端直接执行命令时拦截，可以安装 Shell Wrapper：

```bash
# 创建目录
mkdir -p ~/.local/bin

# 下载 wrapper 脚本
# （见 SKILL.md 中的完整脚本）

# 添加执行权限
chmod +x ~/.local/bin/danger-guard

# 添加到 PATH
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# 设置别名
alias rm='danger-guard rm'
alias git='danger-guard git'
alias docker='danger-guard docker'
```

这样无论通过 AI 工具还是直接在终端执行，都会被拦截。

## 安全性

### 密码存储

- sudo 密码以 SHA256 哈希存储
- 不明文保存密码
- 哈希比对后立即使用，不缓存

### 保护范围

- **OpenClaw Skill**: 拦截通过飞书发来的危险命令
- **Shell Wrapper**: 拦截终端直接执行的危险命令（可选）
- **AI 工具配置**: 拦截 Claude Code、Codex 等工具的危险命令

### 局限性

- 只能拦截已知的危险命令模式
- 无法拦截变体或混淆的命令
- Shell Wrapper 需要用户主动设置别名
- 邮件告警依赖系统邮件配置或 Resend API

## 配置选项

### 白名单

可以添加自定义白名单，这些路径和命令不会触发拦截：

```json
{
  "whitelist": [
    "/tmp/",
    "%TEMP%",
    ".Trash/",
    "git clean",
    "git reset",
    "npm install"
  ]
}
```

### 告警设置

```json
{
  "alert_email": "your@email.com",
  "email_method": "native",
  "resend_api_key": "re_xxx",
  "resend_from": "alerts@yourdomain.com"
}
```

## 日志

所有拦截事件记录到 `memory/danger-guard-logs.md`：

```markdown
## 2026-07-07

### 01:40 - 拦截
- 系统: macOS
- 命令: rm -rf ~/Desktop
- 来源: feishu (ou_xxx)
- 风险等级: 一级
- 验证: 通过
- 告警: 飞书 ✅
```

## 故障排除

### 密码验证失败

**问题:** 输入正确的密码仍然失败

**解决:**
1. 检查 `memory/danger-guard-config.json` 中的 `sudo_hash`
2. 重新计算哈希：`echo -n "你的密码" | shasum -a 256`
3. 更新配置文件

### 邮件发送失败

**问题:** 邮件告警没有收到

**解决:**
- **原生邮件**: 确保系统 Mail.app 已配置发件账号
- **Resend API**: 检查 API Key 是否正确，域名是否已验证
- 查看日志：`tail -f /var/log/mail.log`

### Skill 未激活

**问题:** 执行危险命令时没有拦截

**解决:**
1. 检查 Skill 是否在 skills 目录中
2. 重启 OpenClaw gateway
3. 查看 OpenClaw 日志确认 Skill 加载

## 开发

### 添加新的危险命令

编辑 `SKILL.md`，在"危险命令黑名单"中添加：

```markdown
### 一级危险（立即拦截）

#### 新的危险类别
- `command1`
- `command2`
```

### 修改拦截逻辑

编辑 `SKILL.md` 中的"拦截流程"部分。

### 测试

模拟危险命令：
```
帮我执行 rm -rf /
```

应该看到拦截警告。

## 贡献

欢迎提交 Issue 和 Pull Request！

### 报告问题

- 描述危险命令场景
- 提供复现步骤
- 附上日志（如有）

### 添加功能

- 新的危险命令模式
- 更好的语义识别
- 更多告警渠道
- 其他 AI 工具支持

## 许可证

MIT License

## 致谢

- 灵感来自系统级命令拦截工具
- 感谢所有贡献危险命令黑名单的社区
- 特别感谢 OpenClaw 团队提供 AI Agent 平台

## 相关链接

- [OpenClaw 文档](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [AI Agent 安全最佳实践](https://docs.openclaw.ai/security)

---

**保护你的 AI Agent，防止账号被盗后的破坏性操作。**

🛡️ Danger Guard - 你的 AI 助手的最后一道防线。
