# Danger Guard

AI 智能体安全防护。在执行前拦截危险命令，要求密码验证，当有人试图通过被盗账号破坏你的系统时发送告警。

[English](README.md)

---

## 问题

你的 AI 助手拥有机器的完整权限。如果有人盗取了你的通讯账号——飞书、Telegram、Discord、WhatsApp 等——他们可以告诉 AI 执行 `rm -rf /`、`format C:` 或 `git push --force`。AI 会直接执行。

Danger Guard 阻止这一切。

## 工作原理

```
检测到危险命令
        ↓
立即拦截（不执行任何操作）
        ↓
显示警告 + 风险等级
        ↓
要求 sudo/管理员密码验证
        ↓
3 次失败 → 锁定 + 告警
        ↓
密码正确 → 执行
密码错误 → 拒绝 + 告警
```

## 支持的平台

Danger Guard 适用于任何连接到 AI 智能体的通讯平台：

| 平台 | 告警方式 | 备注 |
|------|---------|------|
| 飞书 | 私信 | 主要平台 |
| Telegram | Bot 消息 | 通过 OpenClaw |
| Discord | Bot 消息 | 通过 OpenClaw |
| WhatsApp | 消息 | 通过 OpenClaw |
| Signal | 消息 | 通过 OpenClaw |
| Slack | Bot 消息 | 通过 OpenClaw |
| 任何 OpenClaw 渠道 | 自动检测 | 告警发送到当前对话 |

## 支持的 AI 工具

Danger Guard 为所有主流 AI 编码工具提供配置：

| 工具 | 配置文件 | 位置 |
|------|---------|------|
| **OpenClaw** | `SKILL.md` | `~/.openclaw/skills/danger-guard/` |
| **Claude Code** | `settings.json` + `CLAUDE.md` | `.claude/` + 项目根目录 |
| **OpenAI Codex** | `AGENTS.md` | 项目根目录 |
| **Cursor / Windsurf** | `.cursorrules` | 项目根目录 |
| **GitHub Copilot** | `.cursorrules` | 项目根目录 |
| **Shell（系统级）** | `danger-guard` 脚本 | `~/.local/bin/` |
| **Git Hooks** | `pre-push` | `.git/hooks/` |

## 拦截内容

### 一级危险 — 始终拦截（需要密码）

**文件系统破坏：**
- `rm -rf /`, `rm -rf /*`, `rm -rf ~`, `rm -rf /Users`
- `find / -exec rm {} +`, `find . -delete`
- `mv / /dev/null`
- `format C:`, `rd /s /q C:\`, `del /s /q /f C:\*.*`

**磁盘操作：**
- `dd if=/dev/zero of=/dev/sda`, `dd if=/dev/urandom of=/dev/sda`
- `mkfs.*`（所有文件系统格式）
- `diskpart clean`, `Clear-Disk -RemoveData`

**权限滥用：**
- `chmod -R 777 /`, `chmod -R 000 /`
- `chown -R root:root /`
- `sudo /bin/bash`, `sudo su -`

**系统破坏：**
- Fork bomb: `:(){:|:&};:`, `fork while fork`
- `shutdown`, `reboot`, `halt`, `poweroff`
- `kill -9 -1`（杀死所有进程）
- `history | sh`

**远程代码执行：**
- `curl ... | sh`, `curl ... | bash`
- `wget ... | sh`, `wget ... | bash`
- `powershell iwr ... | iex`

### 二级危险 — 需要确认

**Git：**
- `git push --force`, `git push -f`, `git push --mirror`
- `git reset --hard`, `git clean -fd`, `git clean -fdx`
- `git branch -D`, `git stash clear`, `git reflog expire`

**Docker：**
- `docker system prune -a --volumes`
- `docker volume prune -f`
- `docker-compose down -v`

**SQL：**
- `DROP TABLE`, `DROP DATABASE`, `DROP SCHEMA`
- `TRUNCATE TABLE`
- `DELETE` 无 `WHERE` 条件
- `UPDATE` 无 `WHERE` 条件

### 语义检测

Danger Guard 还能识别中英文自然语言尝试：
- "Delete all files" / "删除所有文件"
- "Format the disk" / "格式化硬盘"
- "Clear everything" / "清空整个目录"
- "Reset the system" / "重置系统"

## 安装

### 快速开始（OpenClaw）

```bash
# 克隆仓库
git clone https://github.com/Thomaszhou22/danger-guard.git

# 复制到 OpenClaw skills 目录
cp -r danger-guard ~/.openclaw/skills/

# 重启 OpenClaw gateway（或自动检测新 skill）
```

Danger Guard 在首次使用时自动激活并引导你完成设置。

### Onboarding 流程

首次检测到危险命令时，自动运行设置：

```
🛡️ Danger Guard 设置

检测到系统：macOS

第 1 步：输入 sudo 密码（开机密码）
  → 以 SHA256 哈希存储，不明文保存

第 2 步：邮件告警？（可选）
  1. 不需要（仅聊天告警）
  2. 原生邮件（macOS mail / Windows PowerShell）
  3. Resend API（免费每月 3000 封）

第 3 步：Shell Wrapper？（可选）
  1. 不需要（仅保护 AI 执行的命令）
  2. 需要（系统级保护所有终端命令）
```

### 其他 AI 工具

从 `configs/` 复制对应配置到你的项目：

```bash
# Claude Code
cp configs/claude-code/settings.json .claude/settings.json
cp configs/claude-code/CLAUDE.md ./CLAUDE.md

# OpenAI Codex
cp configs/codex/AGENTS.md ./AGENTS.md

# Cursor / Windsurf / Copilot
cp configs/cursor/.cursorrules ./.cursorrules

# Git Hooks（每个仓库）
cp configs/git-hooks/pre-push .git/hooks/pre-push
chmod +x .git/hooks/pre-push
```

完整安装指南：[INSTALL.md](INSTALL.md)

## 保护层

Danger Guard 提供两层互补的保护：

```
┌─────────────────────────────────────────┐
│  第 1 层：AI 智能体保护                  │
│  (OpenClaw Skill / Claude Code 等)      │
│                                         │
│  拦截通过 AI 工具和聊天平台发送的        │
│  危险命令                              │
├─────────────────────────────────────────┤
│  第 2 层：系统级保护                     │
│  (Shell Wrapper — 可选)                 │
│                                         │
│  拦截所有终端命令，无论谁或什么执行它们   │
└─────────────────────────────────────────┘
```

| 层级 | 保护内容 | 设置方式 |
|------|---------|---------|
| **AI 智能体** | 来自聊天（飞书、Telegram 等）的命令 | Skill 安装后自动激活 |
| **AI 工具** | 来自 Claude Code、Codex、Cursor 的命令 | 复制配置文件 |
| **Shell Wrapper** | 所有终端命令（系统级） | Onboarding 时可选 |
| **Git Hooks** | 危险 git 操作（force push） | 复制到 `.git/hooks/` |

## 紧急协议

如果你怀疑账号被盗，回复：

**"account-hacked"** 或 **"账号被盗"**

Danger Guard 将：
1. 记录最近 50 条命令到 `memory/security-breach-YYYY-MM-DD.md`
2. 停止所有后台任务
3. 发送紧急告警（聊天 + 邮件）
4. 提供安全建议

## 配置

配置存储在 `memory/danger-guard-config.json`：

```json
{
  "os": "macOS",
  "sudo_hash": "sha256:...",
  "alert_email": null,
  "email_method": "none",
  "enabled": true,
  "created": "2026-07-07"
}
```

### 白名单

跳过拦截的命令和路径：
- `/tmp/`、`%TEMP%` 目录
- `.Trash/`、回收站操作
- `git clean`、`git reset`（有版本控制）
- `npm install`（项目级，非全局）

## 项目结构

```
danger-guard/
├── SKILL.md                  # 核心 skill（黑名单 + 拦截逻辑）
├── README.md                 # 英文文档
├── README-CN.md              # 中文文档（本文件）
├── INSTALL.md                # 详细安装指南
├── configs/
│   ├── claude-code/          # Claude Code 配置
│   │   ├── settings.json     # 70+ 条拦截规则
│   │   └── CLAUDE.md         # 安全规则
│   ├── codex/                # OpenAI Codex 配置
│   │   └── AGENTS.md         # Agent 指令
│   ├── cursor/               # Cursor/Windsurf 配置
│   │   └── .cursorrules      # 禁止操作
│   ├── shell-wrapper/        # 系统级保护
│   │   └── danger-guard      # Bash 拦截脚本
│   └── git-hooks/            # Git 安全
│       └── pre-push          # 阻止 force push
└── memory/                   # 运行时配置（自动生成）
    └── danger-guard-config.json
```

## 贡献

### 添加新的危险命令

编辑 `SKILL.md`，在相应层级添加模式。

### 添加 AI 工具支持

在 `configs/` 下创建新目录，包含该工具的配置格式。

### 报告问题

- 描述未被拦截（或误拦截）的命令
- 包含确切的命令语法
- 注明触发它的 AI 工具/平台

## 许可证

MIT

## 链接

- [OpenClaw 文档](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)

---

你的 AI 的最后一道防线。
