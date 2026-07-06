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

### 2. 首次使用自动配置（Onboarding）

当 Danger Guard 首次激活时，会自动引导你完成配置：

**第 1 步：检测系统 + 输入密码**
```
🛡️ Danger Guard 首次配置

检测到系统: macOS

请输入你的 sudo 密码（用于验证危险操作）：
密码 [你的开机密码]
```

**第 2 步：配置邮件告警（可选）**
```
是否需要配置邮件告警？（可选）
拦截危险命令时，除了飞书通知外，还会发邮件给你。

1. 不需要邮件（只发飞书告警）
2. 原生邮件（macOS 用 mail 命令，Windows 用 PowerShell）
3. Resend API（免费每月 3000 封，需要 API Key）

回复 1/2/3
```

**第 3 步：配置 Shell Wrapper（可选）**
```
是否需要安装 Shell Wrapper？（可选）

Shell Wrapper 会在你直接在终端敲命令时也拦截危险操作。
不只是通过 AI 工具，而是所有 shell 命令都会被保护。

保护范围：rm、mv、find、dd、mkfs、chmod、chown、git、docker 等

⚠️ 注意：所有受保护的命令都会要求输入 sudo 密码，包括安全操作（如 rm file.txt）。
   如果你经常用这些命令，可能会觉得繁琐。

1. 不需要 Shell Wrapper（只保护通过 AI 执行的命令）
2. 安装 Shell Wrapper（系统级全面保护）

回复 1/2
```

**第 4 步：保存配置**
- 密码转 SHA256 哈希存储
- 写入 `memory/danger-guard-config.json`
- 如选择 Shell Wrapper，自动安装到 `~/.local/bin/` 并配置 shell 别名

**第 5 步：测试告警**
- 如配置了邮件，发送测试邮件确认能收到

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

Danger Guard 提供 **5 种配置方案**，覆盖主流 AI 开发工具和系统级保护。所有配置文件都在 `configs/` 目录中。

### 1. Claude Code (Anthropic CLI)

**配置文件：**
- `configs/claude-code/settings.json` - 70+ 条正则表达式拦截规则
- `configs/claude-code/CLAUDE.md` - 安全规则说明

**安装：**
```bash
mkdir -p .claude
cp /path/to/danger-guard/configs/claude-code/settings.json .claude/settings.json
cp /path/to/danger-guard/configs/claude-code/CLAUDE.md ./CLAUDE.md
```

**工作原理：** 使用 Claude Code 的 `permissions.deny` 功能，通过正则表达式匹配阻止危险命令执行。

### 2. OpenAI Codex

**配置文件：** `configs/codex/AGENTS.md`

**安装：**
```bash
cp /path/to/danger-guard/configs/codex/AGENTS.md ./AGENTS.md
```

**工作原理：** Codex 会读取项目根目录的 `AGENTS.md`，遵循其中定义的安全规则。

### 3. Cursor / Windsurf / GitHub Copilot

**配置文件：** `configs/cursor/.cursorrules`

**安装：**
```bash
cp /path/to/danger-guard/configs/cursor/.cursorrules ./.cursorrules
```

**工作原理：** 编辑器读取 `.cursorrules` 中的指令，拒绝执行禁止的操作。

### 4. Shell Wrapper（系统级保护）

**配置文件：** `configs/shell-wrapper/danger-guard`

**安装：**（如果 onboarding 时选择安装，会自动完成以下步骤）
```bash
mkdir -p ~/.local/bin
cp /path/to/danger-guard/configs/shell-wrapper/danger-guard ~/.local/bin/danger-guard
chmod +x ~/.local/bin/danger-guard
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
echo 'alias rm="danger-guard rm"' >> ~/.zshrc
echo 'alias git="danger-guard git"' >> ~/.zshrc
echo 'alias docker="danger-guard docker"' >> ~/.zshrc
source ~/.zshrc
```

**工作原理：** 
- Shell wrapper 拦截指定的命令（通过 alias）
- 检查是否匹配危险模式
- 如果匹配，要求输入 sudo 密码验证
- 密码验证通过才执行命令

**保护范围：** rm、mv、find、dd、mkfs、chmod、chown、shutdown、reboot、git、docker 等

### 5. Git Hooks

**配置文件：** `configs/git-hooks/pre-push`

**安装：**
```bash
# 单个仓库
cp /path/to/danger-guard/configs/git-hooks/pre-push .git/hooks/pre-push
chmod +x .git/hooks/pre-push

# 所有新仓库（Git 模板）
mkdir -p ~/.git-templates/hooks
cp /path/to/danger-guard/configs/git-hooks/pre-push ~/.git-templates/hooks/pre-push
chmod +x ~/.git-templates/hooks/pre-push
git config --global init.templatedir '~/.git-templates'
```

**工作原理：** Pre-push hook 在每次 push 前检查是否修改了远程历史（即 force push），如果是则拦截并提示。

**完整安装指南：** 详见 `INSTALL.md`

## 两层保护机制

Danger Guard 提供**两层互补的保护**：

| 保护层级 | 保护场景 | 配置方式 |
|---------|---------|---------|
| **OpenClaw Skill** | 拦截通过飞书发给 AI 的危险命令 | 自动激活，无需手动配置 |
| **Shell Wrapper** | 拦截所有终端命令（包括 AI 工具、你自己敲的命令） | onboarding 时可选安装 |

**为什么需要两层？**
- OpenClaw Skill 只保护通过飞书发给 AI 的指令
- Shell Wrapper 保护**所有**在终端执行的命令，包括：
  - 你自己在终端敲的命令
  - OpenClaw 执行的命令
  - Cursor、Claude Code 等其他 AI 工具执行的命令

**Shell Wrapper 的注意事项：**
- 所有受保护的命令（rm、git、docker 等）都会要求输入 sudo 密码
- 包括安全操作（如 `rm file.txt`），可能会觉得繁琐
- 如果经常在终端操作，可以只装 OpenClaw Skill，不装 Shell Wrapper

## 安全性

### 密码存储

- sudo 密码以 SHA256 哈希存储
- 不明文保存密码
- 哈希比对后立即使用，不缓存

### 保护范围

- **OpenClaw Skill**: 拦截通过飞书发来的危险命令（聊天层）
- **Shell Wrapper**: 拦截终端直接执行的危险命令（系统层）
- **AI 工具配置**: 拦截 Claude Code、Codex、Cursor 等工具的危险命令（应用层）
- **Git Hooks**: 拦截 force push 等危险 git 操作（版本控制层）

### 局限性

- 只能拦截已知的危险命令模式
- 无法拦截变体或混淆的命令
- Shell Wrapper 会拦截所有受保护命令（包括安全操作），可能影响工作效率
- 邮件告警依赖系统邮件配置或 Resend API
- Git Hooks 需要为每个仓库单独安装，或配置 Git 模板

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

## 项目结构

```
danger-guard/
├── SKILL.md              # Skill 主文件（危险命令黑名单 + 拦截逻辑）
├── README.md             # 项目说明文档
├── INSTALL.md            # 详细安装指南
├── configs/              # 跨 AI 工具配置文件
│   ├── claude-code/      # Claude Code 配置
│   │   ├── settings.json # 70+ 条拦截规则
│   │   └── CLAUDE.md     # 安全规则说明
│   ├── codex/            # OpenAI Codex 配置
│   │   └── AGENTS.md     # Codex 安全指令
│   ├── cursor/           # Cursor/Windsurf 配置
│   │   └── .cursorrules  # 禁止操作列表
│   ├── shell-wrapper/    # Shell Wrapper 脚本
│   │   └── danger-guard  # 系统级拦截脚本
│   └── git-hooks/        # Git Hooks
│       └── pre-push      # 阻止 force push
└── memory/               # 运行时配置（自动生成）
    └── danger-guard-config.json
```

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
