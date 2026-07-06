# Danger Guard 安装指南

本文档说明如何为不同的 AI 开发工具配置 Danger Guard 安全保护。

## 目录

1. [Claude Code (Anthropic CLI)](#claude-code)
2. [OpenAI Codex](#openai-codex)
3. [Cursor / Windsurf / GitHub Copilot](#cursor-windsurf)
4. [系统级 Shell Wrapper](#shell-wrapper)（可选）
5. [Git Hooks](#git-hooks)

---

## Claude Code

为 Claude Code 配置安全规则，防止执行危险命令。

### 安装步骤

1. **创建配置目录**（如果不存在）：
   ```bash
   mkdir -p .claude
   ```

2. **复制 settings.json**：
   ```bash
   cp /path/to/danger-guard/configs/claude-code/settings.json .claude/settings.json
   ```

3. **复制 CLAUDE.md**（如果项目根目录还没有）：
   ```bash
   cp /path/to/danger-guard/configs/claude-code/CLAUDE.md ./CLAUDE.md
   ```
   
   或者将安全规则追加到现有的 CLAUDE.md：
   ```bash
   cat /path/to/danger-guard/configs/claude-code/CLAUDE.md >> ./CLAUDE.md
   ```

### 配置说明

- **settings.json**: 使用 Claude Code 的 permissions.deny 功能，通过正则表达式匹配阻止危险命令
- **CLAUDE.md**: 提供详细的安全规则说明，包括禁止操作、Git 安全、Docker 安全和数据库安全

---

## OpenAI Codex

为 OpenAI Codex 配置安全指令。

### 安装步骤

1. **复制 AGENTS.md**（如果项目根目录还没有）：
   ```bash
   cp /path/to/danger-guard/configs/codex/AGENTS.md ./AGENTS.md
   ```
   
   或者将安全规则追加到现有的 AGENTS.md：
   ```bash
   cat /path/to/danger-guard/configs/codex/AGENTS.md >> ./AGENTS.md
   ```

### 配置说明

- **AGENTS.md**: OpenAI Codex 的标准配置文件，定义 agent 的行为规则和安全约束

---

## Cursor / Windsurf

为 Cursor 和 Windsurf 编辑器配置安全规则。

### 安装步骤

1. **复制 .cursorrules**：
   ```bash
   cp /path/to/danger-guard/configs/cursor/.cursorrules ./.cursorrules
   ```

### 配置说明

- **.cursorrules**: Cursor 和 Windsurf 使用的项目级配置文件，定义 AI 助手的行为规则
- 包含详细的禁止操作列表、安全替代方案表格和确认要求

---

## Shell Wrapper

系统级保护，拦截所有 shell 中的危险命令（可选，推荐高级用户使用）。

### 安装步骤

1. **创建本地 bin 目录**（如果不存在）：
   ```bash
   mkdir -p ~/.local/bin
   ```

2. **复制 danger-guard 脚本**：
   ```bash
   cp /path/to/danger-guard/configs/shell-wrapper/danger-guard ~/.local/bin/danger-guard
   chmod +x ~/.local/bin/danger-guard
   ```

3. **添加到 PATH**：
   ```bash
   # 对于 bash
   echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
   source ~/.bashrc
   
   # 对于 zsh
   echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
   source ~/.zshrc
   ```

4. **配置命令别名**（保护特定命令）：
   ```bash
   # 对于 bash
   cat >> ~/.bashrc << 'EOF'
   # Danger Guard 保护
   alias rm='danger-guard rm'
   alias mv='danger-guard mv'
   alias find='danger-guard find'
   alias dd='danger-guard dd'
   alias mkfs='danger-guard mkfs'
   alias chmod='danger-guard chmod'
   alias chown='danger-guard chown'
   alias shutdown='danger-guard shutdown'
   alias reboot='danger-guard reboot'
   alias git='danger-guard git'
   alias docker='danger-guard docker'
   alias docker-compose='danger-guard docker-compose'
   EOF
   
   # 对于 zsh
   cat >> ~/.zshrc << 'EOF'
   # Danger Guard 保护
   alias rm='danger-guard rm'
   alias mv='danger-guard mv'
   alias find='danger-guard find'
   alias dd='danger-guard dd'
   alias mkfs='danger-guard mkfs'
   alias chmod='danger-guard chmod'
   alias chown='danger-guard chown'
   alias shutdown='danger-guard shutdown'
   alias reboot='danger-guard reboot'
   alias git='danger-guard git'
   alias docker='danger-guard docker'
   alias docker-compose='danger-guard docker-compose'
   EOF
   ```

5. **重新加载配置**：
   ```bash
   source ~/.bashrc  # 或 source ~/.zshrc
   ```

### 配置说明

- **工作原理**: Shell wrapper 拦截指定的命令，检查是否匹配危险模式，如果匹配则要求输入 sudo 密码验证
- **密码验证**: 使用 SHA-256 哈希比对，密码存储在 `~/.danger-guard.json`（由 OpenClaw 配置）
- **日志记录**: 所有拦截和执行的命令记录在 `~/.danger-guard.log`
- **最大尝试次数**: 密码验证失败 3 次后拦截命令

### 注意事项

⚠️ **重要**: Shell wrapper 会拦截所有通过别名保护的命令，即使不是危险操作。例如：
- `rm file.txt` 会被拦截（即使只是删除单个文件）
- `git push` 会被拦截（即使不是 force push）

如果这影响了你的工作流程，可以：
1. 减少别名数量，只保护最危险的命令
2. 使用 `command rm file.txt` 绕过 wrapper
3. 临时禁用：`unalias rm`

---

## Git Hooks

为 Git 仓库配置 pre-push hook，阻止 force push。

### 安装步骤

#### 方法 1: 单个仓库

1. **复制 pre-push hook**：
   ```bash
   cp /path/to/danger-guard/configs/git-hooks/pre-push .git/hooks/pre-push
   chmod +x .git/hooks/pre-push
   ```

#### 方法 2: 所有新仓库（Git 模板）

1. **创建 Git 模板目录**：
   ```bash
   mkdir -p ~/.git-templates/hooks
   ```

2. **复制 pre-push hook**：
   ```bash
   cp /path/to/danger-guard/configs/git-hooks/pre-push ~/.git-templates/hooks/pre-push
   chmod +x ~/.git-templates/hooks/pre-push
   ```

3. **配置 Git 使用模板**：
   ```bash
   git config --global init.templatedir '~/.git-templates'
   ```

4. **对于已存在的仓库**：
   ```bash
   cd /path/to/repo
   git init  # 这会重新应用模板，但不会丢失数据
   ```

### 配置说明

- **工作原理**: Pre-push hook 在每次 push 前检查是否修改了远程历史（即 force push）
- **检测逻辑**: 使用 `git merge-base --is-ancestor` 检查远程 SHA 是否是本地 SHA 的祖先
- **绕过方法**: 如果确实需要 force push，可以临时重命名 hook：
  ```bash
  mv .git/hooks/pre-push .git/hooks/pre-push.bak
  git push --force
  mv .git/hooks/pre-push.bak .git/hooks/pre-push
  ```

---

## 验证安装

### 测试 Claude Code

在 Claude Code 中尝试执行危险命令，应该看到权限被拒绝的错误。

### 测试 OpenAI Codex

在 Codex 中询问如何执行危险命令，应该看到安全警告和建议。

### 测试 Cursor / Windsurf

在编辑器中让 AI 执行危险命令，应该看到拒绝执行的响应。

### 测试 Shell Wrapper

在终端中执行：
```bash
rm -rf /tmp/test
```

应该看到 Danger Guard 警告并要求输入 sudo 密码。

### 测试 Git Hooks

尝试 force push：
```bash
git push --force
```

应该看到 Danger Guard 拦截消息。

---

## 卸载

### 卸载 Claude Code 配置

```bash
rm .claude/settings.json
# 如果 CLAUDE.md 只包含 Danger Guard 规则：
rm CLAUDE.md
```

### 卸载 OpenAI Codex 配置

```bash
# 如果 AGENTS.md 只包含 Danger Guard 规则：
rm AGENTS.md
```

### 卸载 Cursor / Windsurf 配置

```bash
rm .cursorrules
```

### 卸载 Shell Wrapper

```bash
# 删除脚本
rm ~/.local/bin/danger-guard

# 从 shell 配置中移除 PATH 和别名
# 编辑 ~/.bashrc 或 ~/.zshrc，删除相关行
nano ~/.bashrc  # 或 nano ~/.zshrc

# 重新加载
source ~/.bashrc  # 或 source ~/.zshrc
```

### 卸载 Git Hooks

```bash
# 单个仓库
rm .git/hooks/pre-push

# Git 模板
rm ~/.git-templates/hooks/pre-push
git config --global --unset init.templatedir
```

---

## 故障排除

### Shell Wrapper 不工作

1. **检查 PATH**：
   ```bash
   echo $PATH | grep local/bin
   ```
   应该看到 `~/.local/bin` 或 `/Users/username/.local/bin`

2. **检查别名**：
   ```bash
   alias | grep danger-guard
   ```
   应该看到配置的别名

3. **检查脚本权限**：
   ```bash
   ls -l ~/.local/bin/danger-guard
   ```
   应该有执行权限（`-rwxr-xr-x`）

4. **检查配置文件**：
   ```bash
   cat ~/.danger-guard.json
   ```
   应该包含 `sudo_hash` 字段

### Git Hook 不工作

1. **检查 hook 权限**：
   ```bash
   ls -l .git/hooks/pre-push
   ```
   应该有执行权限（`-rwxr-xr-x`）

2. **检查 hook 内容**：
   ```bash
   head -5 .git/hooks/pre-push
   ```
   应该看到 shebang 行 `#!/bin/bash`

3. **手动测试 hook**：
   ```bash
   .git/hooks/pre-push
   ```
   应该正常运行或显示错误信息

### AI 工具配置不生效

1. **检查文件位置**：确保配置文件在项目根目录
2. **重启 AI 工具**：某些工具需要重启才能加载新配置
3. **检查文件语法**：使用 JSON 验证器检查 settings.json

---

## 更新

当 Danger Guard 发布新版本时：

1. **下载更新**：
   ```bash
   git pull origin main  # 如果使用 Git
   # 或重新下载压缩包
   ```

2. **重新安装配置**：按照上述步骤重新复制配置文件

3. **重启相关工具**：确保新配置生效

---

## 支持

如有问题，请：
1. 查看 [README.md](./README.md) 了解更多信息
2. 查看 [SKILL.md](./SKILL.md) 了解技术细节
3. 在 GitHub Issues 中报告问题
