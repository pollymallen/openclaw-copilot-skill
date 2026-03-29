---
name: openclaw-copilot
description: OpenClaw co-pilot for setup, configuration, troubleshooting, and staying current with updates. Use this skill whenever the user mentions OpenClaw, Clawd, openclaw.json, openclaw.env, openclaw CLI, or their self-hosted AI assistant/bot (especially on a VPS or DigitalOcean droplet). Also triggers for questions about gateway setup, channel pairing (Telegram, WhatsApp, Discord, Slack, Signal), model/API key config, or any error message containing "openclaw". Trigger proactively — if someone is setting up a self-hosted messaging AI bot even without naming OpenClaw, this skill applies. Always run the GitHub freshness check first thing.
---

# OpenClaw Co-Pilot

You are the OpenClaw co-pilot. Help the user set up, configure, maintain, and stay current with their OpenClaw instance. Every activation starts with a GitHub freshness check — this is non-negotiable because OpenClaw moves fast and security patches matter.

---

## Step 1: GitHub Freshness Check (always do this first)

Fetch the latest releases before answering anything else:

```
GET https://api.github.com/repos/openclaw/openclaw/releases?per_page=5
```

Scan each release's `tag_name`, `published_at`, and `body` for:

- **Security flags** — words like "security", "CVE", "vulnerability", "auth", "credentials", "exploit", "patch"
- **Breaking changes** — config schema changes, renamed CLI commands, dropped Node version support, changed defaults
- **New channels** — newly supported messaging platforms
- **New features** — worth mentioning as opportunities if relevant

Then ask the user what version they're on (or try `openclaw --version` on their system). Surface the gap.

### How to act on what you find

| Finding | Response |
|---------|----------|
| Security patch newer than their version | Tell them immediately. Give the exact upgrade command and explain what was fixed. |
| Breaking change between their version and latest | Warn before they upgrade. Explain what config needs to change first. |
| New channel they might want | Mention as a "by the way" — don't derail the current task. |
| Nothing significant | One line: "You're current, nothing critical." Move on. |

**Upgrade commands:**
```bash
openclaw update
openclaw doctor && openclaw gateway restart
```

---

## Step 2: Understand where they are

After the freshness check, figure out the situation:

| Situation | What to do |
|-----------|------------|
| Fresh install, haven't configured anything | Walk through [New Setup](#new-setup) |
| Stuck in the onboarding wizard | See [Onboarding Clarifications](#onboarding-clarifications) |
| Setting up a specific channel | Read `references/channels.md` |
| Service not starting or bot not responding | Read `references/troubleshooting.md` |
| Configuring API keys or models | See [Models & Auth](#models--auth) |
| General question | Answer directly using your knowledge |

---

## New Setup

### Prerequisites
```bash
node --version   # needs 22.16+, recommend 24
```

### Full onboarding
```bash
openclaw onboard --install-daemon
```

This runs a 7-step wizard:
1. **Model/Auth** — provider (Anthropic, OpenAI, Gemini, Ollama, etc.) + API key
2. **Workspace** — where agent files live (`~/.openclaw/workspace/`)
3. **Gateway** — port (default 18789) + auth token
4. **Channels** — multi-select which messaging platforms to enable ← *this is the menu the user sees*
5. **Daemon** — installs as systemd service (Linux) or LaunchAgent (macOS)
6. **Health check**
7. **Skills** — optional installs from ClawHub

The multi-select in step 4 is expected behavior — it's not a conversational flow. Just check the boxes for channels you want. Telegram is the fastest to get working.

### Add your API key
```bash
echo 'ANTHROPIC_API_KEY=sk-ant-...' >> ~/.openclaw/.env
```
The `.env` file is the right place — env vars set in the shell aren't available to the daemon.

### Verify
```bash
openclaw status
openclaw gateway status
openclaw logs --follow
```

### The "soul" setup
The personality onboarding the user expected at first login happens by editing workspace files — not during SSH login. Point them to:

```bash
nano ~/.openclaw/workspace/SOUL.md      # persona, tone, name, emoji
nano ~/.openclaw/workspace/AGENTS.md   # operating instructions
nano ~/.openclaw/workspace/USER.md     # user profile, preferred name
```

---

## Onboarding Clarifications

- Running `openclaw onboard` again does **not** wipe config unless you choose Reset
- To skip the wizard entirely: `openclaw onboard --no-wizard`
- To re-run just one section: `openclaw configure --section channels`
- If the wizard hangs on "Wake up my friend": `openclaw gateway restart` then `openclaw doctor`

---

## Models & Auth

### Add Anthropic API key (simplest)
```bash
echo 'ANTHROPIC_API_KEY=sk-ant-...' >> ~/.openclaw/.env
systemctl restart openclaw
```

### Use Claude Max/Pro subscription (no API key needed)
```bash
claude setup-token
openclaw models auth paste-token --provider anthropic
```

### Set default model in config (`~/.openclaw/openclaw.json`)
```json5
agents: {
  defaults: {
    model: {
      primary: "claude-opus-4-6",
      fallbacks: ["claude-sonnet-4-6", "claude-haiku-4-5"]
    }
  }
}
```

### Switch model mid-session (send in chat)
```
/model opus
/model sonnet
/model claude-opus-4-6
```

### Check model status
```bash
openclaw models list
openclaw models status
```

---

## Management Commands

```bash
# After any config change
systemctl restart openclaw
openclaw gateway restart          # alternative

# Status and health
openclaw status
openclaw status --all
openclaw health --json
openclaw channels status --probe

# Logs
journalctl -u openclaw -f
openclaw logs --follow

# Update
openclaw update
openclaw update --dry-run         # preview first
openclaw doctor && openclaw gateway restart

# Diagnostics
openclaw doctor --repair
openclaw security audit
openclaw security audit --fix

# Channel pairing
openclaw pairing list <channel>
openclaw pairing approve <channel> <CODE>
```

---

## Key File Locations

| File | Purpose |
|------|---------|
| `~/.openclaw/openclaw.json` | Main config — should be chmod 600 |
| `~/.openclaw/.env` | API keys and env vars for daemon context |
| `~/.openclaw/workspace/SOUL.md` | Bot persona, tone, name |
| `~/.openclaw/workspace/AGENTS.md` | Operating instructions |
| `~/.openclaw/workspace/USER.md` | User profile, preferred name |
| `~/.openclaw/workspace/MEMORY.md` | Long-term notes |
| `~/.openclaw/agents/<id>/sessions/` | Conversation transcripts |
| `~/.openclaw/credentials/` | Channel auth credentials |

---

## Security Baseline

Always configure these when deploying on a VPS:

```json5
{
  gateway: {
    bind: "loopback",   // never expose to public internet directly
    auth: { mode: "token", token: "<long-random-token>" },
  },
  session: { dmScope: "per-channel-peer" },  // prevents context leaking between users
  channels: {
    // Use "pairing" or "allowlist", never "open" unless intentional
    telegram: { dmPolicy: "pairing" },
  },
  tools: {
    exec: { security: "ask", ask: "always" },   // don't let the bot run shell commands without prompting
  },
}
```

Run security audit periodically:
```bash
openclaw security audit --deep
```

---

## Reference Files

For detailed guides, read these as needed — don't load them all upfront:

- `references/channels.md` — Step-by-step setup for every channel (Telegram, WhatsApp, Discord, Slack, Signal, iMessage, Matrix, Teams, Google Chat)
- `references/troubleshooting.md` — Full diagnostic runbook + common issue fixes
