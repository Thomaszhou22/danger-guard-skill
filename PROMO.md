# Danger Guard - Promotional Content

## Hacker News

**Title:**
```
Show HN: Safety shield for AI agents that intercepts dangerous commands before execution
```

**URL:** `https://github.com/Thomaszhou22/danger-guard-skill`

**Text:**
```
Your AI assistant has full access to your machine. If someone compromises your messaging account (Feishu, Telegram, Discord, etc.), they can tell your AI to run rm -rf /, format C:, or git push --force. The AI just does it.

Danger Guard solves this by detecting 100+ dangerous commands across Linux, macOS, and Windows, requiring sudo/admin password verification, and sending alerts to your messaging platform and email.

Works with OpenClaw, Claude Code, Cursor, Codex, and any AI tool. Optional system-level shell wrapper for full terminal protection.

The core idea: AI agents need a safety layer between them and the system, especially when they can be controlled remotely through potentially compromised channels.

Feedback welcome on command patterns I might have missed.
```

## Reddit

### r/selfhosted
**Title:** I built a safety shield for AI agents that prevents destructive commands even if your account is hacked

**Body:**
```
Been running an AI assistant that has full access to my machine through OpenClaw. Realized if my Feishu/Telegram account gets compromised, someone could tell my AI to rm -rf / and it would just do it.

So I built Danger Guard - a safety layer that:

- Detects 100+ dangerous commands (rm -rf, format, dd, mkfs, git force push, etc.)
- Requires sudo password verification before execution
- Sends alerts to your messaging platform + email
- Works with OpenClaw, Claude Code, Cursor, Codex, and other AI tools
- Optional system-level shell wrapper

It's basically a firewall between your AI agent and your system. The AI can still do everything it needs to, but destructive operations require explicit human verification.

Open source, MIT licensed: https://github.com/Thomaszhou22/danger-guard-skill

Anyone else worried about AI agents having unrestricted system access?
```

### r/ChatGPT
**Title:** Built a security layer for AI assistants that stops them from executing dangerous commands

**Body:**
```
If you're giving your AI assistant access to your computer (like through OpenClaw, Claude Code, or similar tools), you might want to check this out.

I built Danger Guard - it intercepts dangerous commands before they execute:

- Filesystem destruction (rm -rf /, format C:)
- Disk operations (dd, mkfs)
- Git force pushes
- Docker system prune
- And 100+ more patterns

Requires password verification and sends alerts. Works with any messaging platform (Feishu, Telegram, Discord, WhatsApp, etc.) and multiple AI tools.

GitHub: https://github.com/Thomaszhou22/danger-guard-skill

Thought the community might find this useful as more people give AI agents system access.
```

### r/LocalLLaMA
**Title:** Safety shield for local AI agents with system access - intercepts dangerous commands

**Body:**
```
Been running local AI agents with full system access and got paranoid about security. Built Danger Guard to add a safety layer:

- Detects dangerous commands (rm -rf, format, dd, git force push, etc.)
- Requires sudo password before execution
- Sends alerts via messaging platform + email
- Works with OpenClaw, Claude Code, Cursor, Codex
- Optional shell wrapper for system-wide protection

Open source: https://github.com/Thomaszhou22/danger-guard-skill

For anyone running local AI agents with system access, this might save your data someday.
```

## Twitter/X

**Post:**
```
Your AI assistant can rm -rf / if someone hacks your messaging account.

Built Danger Guard to fix this:
- Intercepts 100+ dangerous commands
- Requires password verification
- Sends alerts
- Works with OpenClaw, Claude Code, Cursor, Codex

Open source: https://github.com/Thomaszhou22/danger-guard-skill

#AI #Security #OpenSource
```

## Product Hunt

**Tagline:**
Safety shield for AI agents - stops dangerous commands before they execute

**Description:**
```
Danger Guard is a safety layer for AI assistants that have system access. It intercepts dangerous commands (rm -rf, format, git force push, etc.) before execution, requires password verification, and sends alerts to your messaging platform.

Works with OpenClaw, Claude Code, Cursor, Codex, and any AI tool. Supports Feishu, Telegram, Discord, WhatsApp, and all major messaging platforms.

Two layers of protection:
1. AI Agent Protection - intercepts commands from chat platforms
2. System-Level Protection - optional shell wrapper for all terminal commands

Open source, MIT licensed.
```
