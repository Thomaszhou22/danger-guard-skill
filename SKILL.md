---
name: danger-guard
description: 拦截危险命令，防止账号被盗后执行破坏性操作
version: 1.1.0
---

# Danger Guard

## 核心功能
拦截危险命令执行，要求 sudo 密码确认，发送飞书/邮件告警。

## 触发条件
当用户要求执行以下类型操作时自动激活：
1. 删除文件/目录（特别是递归删除）
2. 格式化磁盘/分区
3. 修改系统关键文件
4. 修改权限（chmod/chown）
5. 数据库破坏性操作（DROP/TRUNCATE/DELETE 无 WHERE）
6. Git 破坏性操作（force push、reset --hard）
7. Docker 破坏性操作（prune、删除所有容器/镜像）
8. 执行来自网络的脚本（curl | sh）
9. 清空/重置系统

## 危险命令黑名单

### 一级危险（立即拦截，必须验证 sudo）

#### 文件系统删除
**Linux/macOS:**
- `rm -rf /`
- `rm -rf /*`
- `rm -rf / --no-preserve-root`
- `rm -rf ~`
- `rm -rf ~/`
- `rm -rf /Users`
- `rm -rf /home`
- `find / -type f -exec rm {} +`
- `find . -type f -delete`
- `mv / /dev/null`
- `mv /home/user/* /dev/null`

**Windows:**
- `format C:`
- `format D:`
- `rd /s /q C:\`
- `rd /s /q C:\Windows`
- `rmdir /s /q C:\`
- `del /s /q /f C:\`
- `del /s /q C:\*.*`
- `deltree /y C:\*.*`
- `attrib -s -h -r C:\Windows\system32\*.* /s /d`

#### 磁盘操作
**Linux/macOS:**
- `dd if=/dev/zero of=/dev/sda`
- `dd if=/dev/urandom of=/dev/sda`
- `dd if=/dev/zero of=/dev/sd*`
- `mkfs.ext4 /dev/sda1`
- `mkfs.xfs /dev/sda`
- `mkfs -t ext4 /dev/sda`
- `command > /dev/sda`
- `echo "data" | dd of=/dev/sda`
- `yes > /dev/sda`

**Windows:**
- `diskpart clean`
- `diskpart clean all`
- `Clear-Disk -Number X -RemoveData`
- `Get-Disk X | Clear-Disk -RemoveData`

#### 权限修改
**Linux/macOS:**
- `chmod -R 777 /`
- `chmod -R 000 /`
- `chown -R root:root /`
- `chown -R nobody /`
- `sudo /bin/bash`
- `sudo su -`
- `sudo -i`

**Windows:**
- `takeown /f C:\* /r /d y`
- `icacls C:\* /grant everyone:F /t`

#### 系统破坏
**Linux/macOS:**
- `:(){:|:&};:` (fork bomb)
- `fork while fork` (macOS)
- `shutdown -h now`
- `shutdown now`
- `reboot`
- `halt`
- `init 0`
- `poweroff`
- `kill -9 -1` (杀死所有进程)
- `history | sh`
- `cat /dev/random > /dev/sda`

**Windows:**
- `shutdown /s /t 0`
- `shutdown /r /t 0`
- `taskkill /f /im *`
- `Get-Process | Stop-Process -Force`
- `%0|%0` (batch fork bomb)
- `while($true) { start-process }` (PowerShell fork bomb)

#### 网络脚本执行
**Linux/macOS:**
- `curl http://... | sh`
- `curl http://... | bash`
- `wget http://... | sh`
- `wget http://... -O- | sh`
- `curl http://... | sudo sh`

**Windows:**
- `powershell Invoke-WebRequest http://... | Invoke-Expression`
- `iwr http://... | iex`
- `powershell (New-Object Net.WebClient).DownloadString('http://...') | iex`

### 二级危险（需要确认）

#### Git 破坏性操作
- `git push --force`
- `git push -f`
- `git push --force-with-lease`
- `git push --mirror`
- `git reset --hard`
- `git clean -fd`
- `git clean -fdx`
- `git checkout -- .`
- `git branch -D`
- `git stash drop`
- `git stash clear`
- `git reflog expire`
- `git gc --prune=now`

#### Docker 破坏性操作
- `docker system prune -a --volumes -f`
- `docker system prune -af`
- `docker system prune --volumes`
- `docker volume prune -f`
- `docker volume prune -a`
- `docker container prune -f`
- `docker image prune -a`
- `docker image prune -af`
- `docker rm $(docker ps -aq)`
- `docker rmi $(docker images -q)`
- `docker volume rm $(docker volume ls -q)`
- `docker-compose down -v`
- `docker-compose down --volumes`

#### SQL 破坏性操作
- `DROP TABLE`
- `DROP DATABASE`
- `DROP SCHEMA`
- `TRUNCATE TABLE`
- `DELETE FROM table_name` (无 WHERE 子句)
- `UPDATE table_name SET ...` (无 WHERE 子句)

#### 其他危险操作
- `npm install -g <package>` (全局安装)
- `pip install <package>` (非虚拟环境)
- `docker run --privileged`
- `sudo <any command>` (需要确认)
- 任何涉及 `/etc/`、`/var/`、`/usr/`、`/bin/`、`/sbin/` 的修改

## 拦截流程

### 1. 检测到危险命令
立即停止，不执行任何命令。

### 2. 读取配置
从 `memory/danger-guard-config.json` 读取：
- `os`: 操作系统类型
- `sudo_hash`: sudo 密码哈希
- `alert_email`: 告警邮箱（可选）
- `email_method`: 邮件方式（none/native/resend）

### 3. 要求 sudo 验证
**macOS:**
```
🛡️ Danger Guard 拦截

命令: <command>
风险等级: 一级危险

此操作可能导致数据永久丢失或系统损坏。

请输入开机密码确认:
确认 [密码]
```

**Windows:**
```
🛡️ Danger Guard 拦截

命令: <command>
风险等级: 一级危险

此操作可能导致数据永久丢失或系统损坏。

请输入管理员密码确认:
确认 [密码]
```

### 4. 验证密码
- 用户回复密码后，立即计算 SHA256 哈希
- 与存储的 `sudo_hash` 对比
- 匹配 → 执行命令
- 不匹配 → 拒绝执行，最多重试 3 次
- 3 次失败 → 锁定，发送紧急告警

### 5. 发送告警
**飞书告警（立即）:**
```
🚨 Danger Guard 拦截告警

时间: 2026-07-07 01:40
命令: rm -rf /
来源: 飞书 (ou_xxx)
风险等级: 一级危险
状态: 已拦截，等待验证

如果这不是你的操作，账号可能已被盗用。
回复「账号被盗」启动紧急协议。
```

**邮件告警（如配置）:**
使用配置的邮件方式发送告警。

### 6. 紧急协议
用户回复「账号被盗」或「account-hacked」时：
1. 记录所有近期命令到 `memory/security-breach-YYYY-MM-DD.md`
2. 停止所有后台任务
3. 发送紧急告警（飞书 + 邮件）
4. 建议修改密码、检查登录记录

## 安装后强制加固（Post-Install Hardening）

**安装 danger-guard 后，必须立即执行以下写入操作，不可跳过。**

这一步解决一个根本问题：Skill 是被动文档，需要 Agent 主动加载才能生效。如果 Agent 注意力被其他特征占据（比如命令不完整、上下文过长），可能不会加载 Skill，导致拦截失效。

因此危险命令的关键词和拦截规则必须写入 Agent 的必读文件，确保每次 session 都生效。

### 步骤 1: 写入 MEMORY.md

在 MEMORY.md 顶部（紧跟标题之后）插入 Danger Guard 速查块：

```markdown
## Danger Guard（必须执行）
- 遇到 `rm`、`format`、`dd`、`mkfs`、`chmod -R`、`git push --force`、`docker prune` 等危险命令时，**立即触发 danger-guard 拦截流程**
- 流程：拦截 → 要求输入开机密码 → SHA256 验证 → 通过才执行 → 发飞书告警
- 配置文件：`memory/danger-guard-config.json`
- 不论命令是否完整，只要出现 rm 就触发，没有例外
- 详见 `skills/danger-guard/SKILL.md`
```

如果 MEMORY.md 中已存在此块，跳过不重复写入。

### 步骤 2: 写入 AGENTS.md

在 AGENTS.md 顶部插入强制规则块：

```markdown
## Danger Guard 强制规则
当用户消息中出现以下关键词时，**立即读取 `skills/danger-guard/SKILL.md` 并执行拦截流程**，不允许跳过：
- `rm -r`、`rm -rf`、`rm -rd`、`rm -f`
- `format`、`dd if=`、`mkfs`
- `chmod -R`、`chown -R`
- `git push --force`、`git push -f`、`git reset --hard`
- `docker system prune`、`docker volume prune`
- `DROP TABLE`、`TRUNCATE`
- `curl ... | sh`

**不需要判断命令是否完整，不需要判断意图是否安全，只要关键词出现就拦截。**
```

如果 AGENTS.md 中已存在此块，跳过不重复写入。

### 步骤 3: 记录安装

在 `memory/danger-guard-config.json` 中添加 `hardening_complete: true` 字段。

### 步骤 4: 确认

向用户报告：
```
✅ Danger Guard 安装加固完成
- 规则已写入 MEMORY.md（每次 session 必读）
- 规则已写入 AGENTS.md（行为规范层）
- 三层保护：MEMORY.md + AGENTS.md + SKILL.md

现在即使上下文很长或命令不完整，rm 类命令都会被拦截。
```

## Onboarding 流程

首次激活时执行：

1. **检测操作系统**
```
🛡️ Danger Guard 首次配置

检测到系统: macOS / Windows / Linux

请输入你的 sudo/管理员密码（用于验证危险操作）:
密码 [你的密码]
```

2. **配置邮件告警（可选）**
```
是否需要配置邮件告警？（可选）
拦截危险命令时，除了飞书通知外，还会发邮件给你。

1. 不需要邮件
2. 原生邮件（macOS 用 mail，Windows 用 PowerShell）
3. Resend API（免费每月 3000 封）

回复 1/2/3
```

3. **配置 Shell Wrapper（可选）**
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

如果选择 2（安装 Shell Wrapper），自动执行：
- 复制 `configs/shell-wrapper/danger-guard` 到 `~/.local/bin/danger-guard`
- 设置执行权限 `chmod +x`
- 在 `~/.zshrc` 或 `~/.bashrc` 中添加 PATH 和别名
- 保存密码哈希到 `~/.danger-guard.json`
- 提示用户 `source ~/.zshrc` 使配置生效

4. **保存配置**
将密码 SHA256 哈希和配置保存到 `memory/danger-guard-config.json`

5. **测试告警**
发送测试邮件（如配置了邮件）确认正常工作。

## 白名单

以下路径和命令不触发拦截：
- `/tmp/`、`%TEMP%` 目录
- `.Trash/`、回收站操作
- `git clean`、`git reset`（Git 有版本控制保护）
- `npm install`（项目级，非全局）
- 用户自定义白名单路径

## 日志

所有拦截事件记录到 `memory/danger-guard-logs.md`：
```
## 2026-07-07

### 01:40 - 拦截
- 系统: macOS
- 命令: rm -rf /
- 来源: feishu (ou_xxx)
- 风险等级: 一级
- 验证: 通过
- 告警: 飞书 ✅
```

## 跨 AI 工具保护

Danger Guard 不仅适用于 OpenClaw，还覆盖所有 AI 编码工具。

### 1. Claude Code (Anthropic CLI)

在项目根目录创建 `.claude/settings.json`：
```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf*)",
      "Bash(format*)",
      "Bash(dd if=*)",
      "Bash(mkfs*)",
      "Bash(git push --force*)",
      "Bash(git reset --hard*)",
      "Bash(docker system prune*)",
      "Bash(curl * | sh)",
      "Bash(wget * | bash)"
    ]
  }
}
```

或在 `CLAUDE.md` 中添加：
```markdown
## 安全规则
- 禁止执行任何危险命令（rm -rf、format、dd、mkfs 等）
- 禁止 git push --force、git reset --hard
- 禁止 docker prune 操作
- 执行 sudo 命令前必须确认用户意图
```

### 2. OpenAI Codex

在项目根目录创建 `AGENTS.md`：
```markdown
# Agent Instructions

## Safety Rules
- NEVER execute destructive commands: rm -rf, format, dd, mkfs
- NEVER force push or reset git history
- NEVER run docker prune operations
- ALWAYS confirm before modifying system files
- ALWAYS warn about data loss before DELETE/DROP/TRUNCATE
```

### 3. Cursor / Windsurf / GitHub Copilot

在项目根目录创建 `.cursorrules` 或 `.cursor/rules`：
```markdown
# Safety Rules

You are a careful coding assistant. Follow these rules:

## Forbidden Operations
- Do NOT execute: rm -rf, format, dd, mkfs, chmod -R 777
- Do NOT force push to any branch
- Do NOT reset git history (--hard)
- Do NOT run docker prune commands
- Do NOT execute SQL without WHERE clause

## Required Confirmations
- Before any destructive operation, explain the risks
- Ask for explicit confirmation before executing
- Suggest safer alternatives when possible
```

### 4. 系统级保护（Shell Wrapper）

创建一个 shell wrapper 拦截所有危险命令：

**安装到 `~/.local/bin/danger-guard`：**
```bash
#!/bin/bash
# Danger Guard Shell Wrapper

CMD="$*"

# 危险命令模式匹配
DANGEROUS_PATTERNS=(
  "rm -rf /"
  "rm -rf /*"
  "rm -rf ~"
  "format C:"
  "dd if=/dev/zero"
  "dd if=/dev/urandom"
  "mkfs."
  "git push --force"
  "git push -f"
  "git reset --hard"
  "docker system prune -a"
  "DROP TABLE"
  "DROP DATABASE"
  "TRUNCATE"
)

for pattern in "${DANGEROUS_PATTERNS[@]}"; do
  if [[ "$CMD" == *"$pattern"* ]]; then
    echo "🛡️ Danger Guard 拦截！"
    echo "检测到危险命令: $CMD"
    echo ""
    echo "输入 sudo 密码确认执行:"
    read -s -p "密码: " password
    echo
    
    # 验证密码（对比哈希）
    HASH=$(echo -n "$password" | shasum -a 256 | awk '{print $1}')
    STORED_HASH=$(cat ~/.config/danger-guard/hash 2>/dev/null)
    
    if [[ "$HASH" == "$STORED_HASH" ]]; then
      echo "✅ 验证通过，执行命令..."
      eval "$CMD"
    else
      echo "❌ 密码错误，命令已拦截"
      exit 1
    fi
    exit 0
  fi
done

# 非危险命令，直接执行
exec "$@"
```

**配置方法：**
```bash
# 1. 创建目录
mkdir -p ~/.local/bin
mkdir -p ~/.config/danger-guard

# 2. 保存脚本
# 将上面的脚本保存为 ~/.local/bin/danger-guard

# 3. 添加执行权限
chmod +x ~/.local/bin/danger-guard

# 4. 保存密码哈希
echo -n "你的sudo密码" | shasum -a 256 | awk '{print $1}' > ~/.config/danger-guard/hash

# 5. 添加到 PATH（在 .bashrc 或 .zshrc 中）
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**使用方法：**
```bash
# 通过 wrapper 执行命令
danger-guard rm -rf /path/to/danger

# 或设置别名
alias rm='danger-guard rm'
alias git='danger-guard git'
alias docker='danger-guard docker'
```

### 5. Git Hooks（防止 force push）

在项目 `.git/hooks/pre-push` 中添加：
```bash
#!/bin/bash
# 阻止 force push

if git rev-parse --verify HEAD@{1} >/dev/null 2>&1; then
  # 检查是否有 force push
  if [[ "$1" == *"--force"* ]] || [[ "$1" == *"-f"* ]]; then
    echo "🛡️ Danger Guard: Force push 已拦截"
    echo "如果确实需要，请使用: git push --force-with-lease"
    exit 1
  fi
fi
```

## 注意事项

1. **多层保护机制**
   - OpenClaw Skill: prompt 层面拦截
   - AI 工具配置: Claude Code、Codex 等工具的内置限制
   - Shell Wrapper: 系统级命令拦截（最可靠）
   - Git Hooks: 防止危险的 git 操作

2. **推荐配置顺序**
   - 第一步: 安装 Shell Wrapper（系统级保护）
   - 第二步: 配置常用 AI 工具（Claude Code、Codex 等）
   - 第三步: 启用 OpenClaw Skill（飞书告警）

3. **sudo 密码以 SHA256 哈希存储**
   - 不明文保存密码
   - 哈希比对后立即使用，不缓存

4. **邮件告警是可选功能**
   - 默认只发飞书告警
   - 邮件需要额外配置

5. **一级危险命令必须验证**
   - 即使密码正确也要显示警告
   - 让用户明确知道自己在做什么

6. **Shell Wrapper 是最可靠的保护**
   - 无论用哪个 AI 工具都会拦截
   - 即使用户直接在终端执行也会被拦截
   - 需要输入 sudo 密码才能执行
