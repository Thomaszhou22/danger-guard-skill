# Danger Guard

AI agent safety shield. Intercepts dangerous commands before execution, requires password verification, and sends alerts when someone tries to destroy your system through a compromised account.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![OpenClaw](https://img.shields.io/badge/OpenClaw-compatible-blue)](https://openclaw.ai)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-compatible-purple)](https://claude.ai)
[![Cursor](https://img.shields.io/badge/Cursor-compatible-orange)](https://cursor.sh)
[![Platform](https://img.shields.io/badge/platform-all%20agent%20platforms-green)](https://github.com/Thomaszhou22/danger-guard-skill)

[中文文档](README-CN.md)

---

## The Problem

Your AI assistant has full access to your machine. If someone compromises your messaging account — Feishu, Telegram, Discord, WhatsApp, whatever — they can tell your AI to `rm -rf /`, `format C:`, or `git push --force`. The AI just does it.

Danger Guard stops that.

## How It Works

```
Dangerous command detected
        ↓
Immediate intercept (nothing executes)
        ↓
Warning + risk level displayed
        ↓
Requires sudo/admin password verification
        ↓
3 failed attempts → lockout + alert
        ↓
Password correct → execute
Password wrong → block + alert
```

## Supported Platforms

Danger Guard works with any messaging platform that connects to your AI agent:

| Platform | Alert Method | Notes |
|----------|-------------|-------|
| Feishu / Lark | Direct message | Primary platform |
| Telegram | Bot message | Via OpenClaw |
| Discord | Bot message | Via OpenClaw |
| WhatsApp | Message | Via OpenClaw |
| Signal | Message | Via OpenClaw |
| Slack | Bot message | Via OpenClaw |
| Any OpenClaw channel | Auto-detected | Alerts go to the current conversation |

## Supported AI Tools

Danger Guard provides configs for all major AI coding tools:

| Tool | Config File | Location |
|------|------------|----------|
| **OpenClaw** | `SKILL.md` | `~/.openclaw/skills/danger-guard/` |
| **Claude Code** | `settings.json` + `CLAUDE.md` | `.claude/` + project root |
| **OpenAI Codex** | `AGENTS.md` | Project root |
| **Cursor / Windsurf** | `.cursorrules` | Project root |
| **GitHub Copilot** | `.cursorrules` | Project root |
| **Shell (system-level)** | `danger-guard` script | `~/.local/bin/` |
| **Git Hooks** | `pre-push` | `.git/hooks/` |

## What Gets Blocked

### Tier 1 — Always Blocked (Requires Password)

**Filesystem destruction:**
- `rm -rf /`, `rm -rf /*`, `rm -rf ~`, `rm -rf /Users`
- `find / -exec rm {} +`, `find . -delete`
- `mv / /dev/null`
- `format C:`, `rd /s /q C:\`, `del /s /q /f C:\*.*`

**Disk operations:**
- `dd if=/dev/zero of=/dev/sda`, `dd if=/dev/urandom of=/dev/sda`
- `mkfs.*` (all filesystem formats)
- `diskpart clean`, `Clear-Disk -RemoveData`

**Permission abuse:**
- `chmod -R 777 /`, `chmod -R 000 /`
- `chown -R root:root /`
- `sudo /bin/bash`, `sudo su -`

**System destruction:**
- Fork bombs: `:(){:|:&};:`, `fork while fork`
- `shutdown`, `reboot`, `halt`, `poweroff`
- `kill -9 -1` (kill all processes)
- `history | sh`

**Remote code execution:**
- `curl ... | sh`, `curl ... | bash`
- `wget ... | sh`, `wget ... | bash`
- `powershell iwr ... | iex`

### Tier 2 — Blocked With Confirmation

**Git:**
- `git push --force`, `git push -f`, `git push --mirror`
- `git reset --hard`, `git clean -fd`, `git clean -fdx`
- `git branch -D`, `git stash clear`, `git reflog expire`

**Docker:**
- `docker system prune -a --volumes`
- `docker volume prune -f`
- `docker-compose down -v`

**SQL:**
- `DROP TABLE`, `DROP DATABASE`, `DROP SCHEMA`
- `TRUNCATE TABLE`
- `DELETE` without `WHERE`
- `UPDATE` without `WHERE`

### Semantic Detection

Danger Guard also catches natural language attempts in both English and Chinese:
- "Delete all files" / "删除所有文件"
- "Format the disk" / "格式化硬盘"
- "Clear everything" / "清空整个目录"
- "Reset the system" / "重置系统"

## Installation

### Quick Start (OpenClaw)

```bash
# Clone the repo
git clone https://github.com/Thomaszhou22/danger-guard.git

# Copy to your OpenClaw skills directory
cp -r danger-guard ~/.openclaw/skills/

# Restart OpenClaw gateway (or it auto-detects new skills)
```

Danger Guard activates automatically on first use and walks you through setup.

### Post-Install Hardening (v1.1.0 — Automatic)

After installation, Danger Guard automatically writes interception rules into your agent's mandatory-read files:

| Target File | What Gets Written | Why |
|-------------|-------------------|-----|
| `MEMORY.md` | Danger keyword shortcuts at the top | Loaded every session, highest priority |
| `AGENTS.md` | Full keyword trigger list | Behavior rules layer |
| `danger-guard-config.json` | `hardening_complete: true` | Marks hardening as done |

This ensures interception works even when the agent doesn't actively load the Skill. No manual action needed — it happens automatically on first run after install.

```
✅ Danger Guard 安装加固完成
- 规则已写入 MEMORY.md（每次 session 必读）
- 规则已写入 AGENTS.md（行为规范层）
- 三层保护：MEMORY.md + AGENTS.md + SKILL.md
```

### Onboarding

The first time Danger Guard detects a dangerous command, it runs through setup:

```
🛡️ Danger Guard Setup

Detected system: macOS

Step 1: Enter your sudo password (login password)
  → Stored as SHA256 hash, never plaintext

Step 2: Email alerts? (optional)
  1. No (chat alerts only)
  2. Native mail (macOS mail / Windows PowerShell)
  3. Resend API (free 3000/month)

Step 3: Shell Wrapper? (optional)
  1. No (protect AI-executed commands only)
  2. Yes (protect ALL terminal commands system-wide)
```

### Other AI Tools

Copy the appropriate config from `configs/` to your project:

```bash
# Claude Code
cp configs/claude-code/settings.json .claude/settings.json
cp configs/claude-code/CLAUDE.md ./CLAUDE.md

# OpenAI Codex
cp configs/codex/AGENTS.md ./AGENTS.md

# Cursor / Windsurf / Copilot
cp configs/cursor/.cursorrules ./.cursorrules

# Git Hooks (per-repo)
cp configs/git-hooks/pre-push .git/hooks/pre-push
chmod +x .git/hooks/pre-push
```

Full installation guide: [INSTALL.md](INSTALL.md)

## Protection Layers

Danger Guard provides **three layers** of protection:

```
┌─────────────────────────────────────────────┐
│  Layer 0: Hardened Memory (Post-Install)     │
│  (MEMORY.md + AGENTS.md)                     │
│                                              │
│  Danger command keywords written directly    │
│  into the agent's mandatory-read files.      │
│  Ensures interception even if the agent      │
│  doesn't actively load the Skill.            │
├──────────────────────────────────────────────┤
│  Layer 1: AI Agent Protection                │
│  (OpenClaw Skill / Claude Code / etc.)       │
│                                              │
│  Intercepts dangerous commands when          │
│  sent through AI tools and chat platforms    │
├──────────────────────────────────────────────┤
│  Layer 2: System-Level Protection            │
│  (Shell Wrapper — optional)                  │
│                                              │
│  Intercepts ALL terminal commands,           │
│  regardless of who or what runs them         │
└──────────────────────────────────────────────┘
```

### Why Three Layers?

Skills are passive documents — they only work when the agent actively loads them. If the agent's attention is occupied (long context, incomplete command, unrelated topic), it may not load the Skill, and interception fails.

**Post-Install Hardening** (new in v1.1.0) solves this by writing danger command keywords directly into `MEMORY.md` and `AGENTS.md` — files the agent reads every single session. This guarantees interception regardless of whether the Skill is loaded.

| Layer | What It Protects | Guarantee |
|-------|-----------------|----------|
| **Hardened Memory** | Forces agent to recognize danger keywords | Every session, always loaded |
| **AI Agent** | Commands from chat (Feishu, Telegram, etc.) | Auto-activates with skill install |
| **AI Tools** | Commands from Claude Code, Codex, Cursor | Copy config files |
| **Shell Wrapper** | ALL terminal commands (system-wide) | Optional during onboarding |
| **Git Hooks** | Dangerous git operations (force push) | Copy to `.git/hooks/` |

## Emergency Protocol

If you suspect your account is compromised, reply with:

**"account-hacked"** or **"账号被盗"**

Danger Guard will:
1. Log the last 50 commands to `memory/security-breach-YYYY-MM-DD.md`
2. Stop all background tasks
3. Send emergency alerts (chat + email)
4. Provide security recommendations

## Configuration

Config is stored in `memory/danger-guard-config.json`:

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

### Whitelist

Commands and paths that skip interception:
- `/tmp/`, `%TEMP%` directories
- `.Trash/`, Recycle Bin operations
- `git clean`, `git reset` (version-controlled)
- `npm install` (project-level, not global)

## Project Structure

```
danger-guard/
├── SKILL.md                  # Core skill (blacklist + intercept logic)
├── README.md                 # This file (English)
├── README-CN.md              # Chinese version
├── INSTALL.md                # Detailed installation guide
├── configs/
│   ├── claude-code/          # Claude Code config
│   │   ├── settings.json     # 70+ deny patterns
│   │   └── CLAUDE.md         # Safety rules
│   ├── codex/                # OpenAI Codex config
│   │   └── AGENTS.md         # Agent instructions
│   ├── cursor/               # Cursor/Windsurf config
│   │   └── .cursorrules      # Forbidden operations
│   ├── shell-wrapper/        # System-level protection
│   │   └── danger-guard      # Bash intercept script
│   └── git-hooks/            # Git safety
│       └── pre-push          # Block force push
└── memory/                   # Runtime config (auto-generated)
    └── danger-guard-config.json
```

## Contributing

### Add New Dangerous Commands

Edit `SKILL.md` and add patterns under the appropriate tier.

### Add AI Tool Support

Create a new directory under `configs/` with the tool's config format.

### Report Issues

- Describe the command that wasn't caught (or was falsely caught)
- Include the exact command syntax
- Note which AI tool / platform triggered it

## License

MIT

## Links

- [OpenClaw Docs](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)

---

Your AI's last line of defense. Now with three-layer hardening.
