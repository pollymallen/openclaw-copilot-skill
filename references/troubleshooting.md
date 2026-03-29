# Troubleshooting Reference

## Universal Diagnostic Flow

Run these in order — they cover 90% of issues:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
openclaw security audit
```

---

## Common Issues

### Bot not responding to DMs
```bash
openclaw pairing list <channel>
openclaw pairing approve <channel> <CODE>
```
Or change `dmPolicy` to `allowlist` and add your number/username to `allowFrom`.

### Bot silent in groups
- Check `groupPolicy` in config — may be set to `disabled`
- Set `requireMention: false` if you want it to respond without being @mentioned
- For Telegram: BotFather → `/setprivacy` → disable, then remove/re-add bot to group

### "All models failed" error
```bash
# Check your API key is set
cat ~/.openclaw/.env
# Verify the service sees it
openclaw models status
# Restart after adding key
systemctl restart openclaw
```
Also check: have you hit your Anthropic usage limit? Check console.anthropic.com.

### Service not starting / port 18789 busy
```bash
openclaw gateway --force    # force-kill existing listener, then start
# or
openclaw doctor --repair
```

### Daemon not starting on boot (Linux VPS)
```bash
sudo loginctl enable-linger <username>
systemctl --user enable openclaw
```

### Config changes not taking effect
- The service hot-reloads most changes automatically
- For critical changes (gateway port, auth mode): full restart required
- Use `openclaw config set <key> <value>` for small changes — safer than editing JSON directly

### WhatsApp disconnects / QR loops
```bash
openclaw channels login --channel whatsapp
# Re-scan QR code
systemctl restart openclaw
```

### Discord bot not seeing messages
Go to Discord Developer Portal → your app → Bot → Privileged Gateway Intents → enable **Message Content Intent**. This is the #1 Discord issue.

### Slack Socket Mode connected but not replying
You need BOTH tokens:
- `appToken` starting with `xapp-` (App-Level Token, scope: `connections:write`)
- `botToken` starting with `xoxb-` (Bot User OAuth Token)

Missing either one = silent failure.

### Signal bot silent
Verify `signal-cli` is running in daemon mode (not just linked). Check `cliPath` in config points to the actual binary.

### Gateway accessible but Control UI not loading
```bash
openclaw status
# Check controlUi.enabled: true in config
# Try: http://127.0.0.1:18789/
# Auth token required — paste from: openclaw config get gateway.auth.token
```

### SSH only (can't reach dashboard URL)
Options:
1. SSH tunnel: `ssh -N -L 18789:127.0.0.1:18789 user@<droplet-ip>` then open `http://127.0.0.1:18789`
2. Use the CLI directly on the server
3. Configure Tailscale if you want persistent remote access

---

## Log Analysis Tips

```bash
# Follow live
journalctl -u openclaw -f

# Last 100 lines
journalctl -u openclaw -n 100

# Since last boot
journalctl -u openclaw -b

# Filter for errors only
journalctl -u openclaw -p err
```

Key phrases to search for in logs:
- `"error"` or `"Error"` — general errors
- `"auth"` — authentication failures
- `"reconnect"` — channel disconnections
- `"model"` or `"provider"` — AI provider issues
- `"rate limit"` — API rate limiting

---

## Nuclear Options

**Reset everything and start fresh:**
```bash
openclaw reset --scope full --yes --non-interactive
```
⚠️ This wipes all sessions, credentials, and config. Back up `~/.openclaw/workspace/` first if you've customized your soul/agent files.

**Reinstall:**
```bash
npm install -g openclaw@latest
openclaw doctor --repair
openclaw onboard --install-daemon
```

---

## Version and Update Issues

```bash
# Check current version
openclaw --version

# Check what's available
openclaw update --dry-run

# Roll back to specific version
npm i -g openclaw@<version>
openclaw doctor && openclaw gateway restart
```

Update channels: `stable` (default), `beta`, `dev`. Switch with:
```bash
openclaw update --channel stable
```
