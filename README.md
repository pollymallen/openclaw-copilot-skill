# openclaw-copilot

A Claude Code skill that acts as an AI co-pilot for [OpenClaw](https://github.com/openclaw/openclaw) — the self-hosted personal AI assistant.

## What it does

Every time it activates, it:
1. **Checks GitHub** for new OpenClaw releases — surfaces security patches, breaking changes, and new features before doing anything else
2. **Guides setup** — onboarding, API keys, daemon configuration
3. **Handles channel setup** — Telegram, WhatsApp, Discord, Slack, Signal, iMessage, Matrix, Teams, Google Chat
4. **Troubleshoots** — diagnostics, common fixes, log analysis

## Install on any OpenClaw machine

```bash
git clone https://github.com/pollymallen/openclaw-copilot-skill.git ~/.openclaw/skills/openclaw-copilot
```

Then add to `~/.openclaw/openclaw.json`:
```json5
skills: {
  load: {
    extraDirs: ["~/.openclaw/skills/"]
  }
}
```

Restart: `systemctl restart openclaw` (or `openclaw gateway restart`)

## Keep it updated

```bash
cd ~/.openclaw/skills/openclaw-copilot && git pull
systemctl restart openclaw
```

Or automate with OpenClaw's built-in cron:
```bash
openclaw cron add \
  --name "Update openclaw-copilot skill" \
  --cron "0 6 * * *" \
  --tz "America/Los_Angeles" \
  --message "run: cd ~/.openclaw/skills/openclaw-copilot && git pull && systemctl restart openclaw"
```

## Structure

```
SKILL.md                    ← main skill instructions (auto-loads into Claude context)
references/
  channels.md               ← step-by-step setup for all supported channels
  troubleshooting.md        ← full diagnostic runbook
```

## Contributing

PRs welcome. Future roles being considered:
- CTO persona — architecture decisions, scaling, infrastructure planning
- Security auditor — periodic hardening reviews
- Channel specialist skills — deeper per-platform expertise

## License

MIT
