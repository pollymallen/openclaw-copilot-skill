# Channel Setup Reference

## Telegram (fastest — start here)

1. Message `@BotFather` on Telegram → `/newbot` → follow prompts → copy the token
2. Add to `~/.openclaw/openclaw.json`:
```json5
channels: {
  telegram: {
    enabled: true,
    botToken: "123456:ABCdef...",
    dmPolicy: "pairing",
    groups: { "*": { requireMention: true } },
  }
}
```
3. Restart: `systemctl restart openclaw`
4. DM your bot on Telegram — it replies with a pairing code
5. Approve: `openclaw pairing approve telegram <CODE>`

**Group chats not responding?** BotFather → `/setprivacy` on your bot → disable privacy mode. Then remove and re-add the bot to the group.

---

## WhatsApp

Plugin required first:
```bash
openclaw plugins install @openclaw/whatsapp
```

Config:
```json5
channels: {
  whatsapp: {
    dmPolicy: "allowlist",
    allowFrom: ["+15551234567"],  // E.164 format, your number
  }
}
```

Link account (QR scan):
```bash
openclaw channels login --channel whatsapp
```

Restart: `systemctl restart openclaw`

**Important:** Use a dedicated separate number. Don't link your personal WhatsApp — it's a shared session and can cause issues.

---

## Discord

1. Discord Developer Portal → New Application → Bot tab
2. Enable under Privileged Gateway Intents: **Message Content Intent** + **Server Members Intent**
3. Copy the bot token
4. Add bot to your server: OAuth2 → URL Generator → scopes: `bot` + `applications.commands`
5. Config:
```json5
channels: {
  discord: {
    enabled: true,
    token: { source: "env", id: "DISCORD_BOT_TOKEN" },
    dmPolicy: "pairing",
    guilds: {
      "YOUR_SERVER_ID": {
        requireMention: false,
        users: ["YOUR_USER_ID"],
      }
    }
  }
}
```
6. Add to `~/.openclaw/.env`: `DISCORD_BOT_TOKEN=your-token`
7. Restart, DM the bot, approve pairing: `openclaw pairing approve discord <CODE>`

**Bot not seeing messages?** Almost always the Message Content Intent — go back to Developer Portal and enable it.

---

## Slack

**Socket Mode (easier — no public URL needed):**

1. Create Slack app at api.slack.com → Enable Socket Mode
2. Generate App Token (`xapp-...`) with scope `connections:write`
3. Install app to workspace → copy Bot Token (`xoxb-...`)
4. Subscribe to events: `app_mention`, `message.im`, `message.channels`, `message.groups`
5. Enable App Home → Messages Tab
6. Config:
```json5
channels: {
  slack: {
    enabled: true,
    mode: "socket",
    appToken: "xapp-...",
    botToken: "xoxb-...",
    dmPolicy: "pairing",
  }
}
```

**Connected but no replies?** Verify you have BOTH `appToken` (xapp-) AND `botToken` (xoxb-) — both are required.

---

## Signal

Requires `signal-cli` (Java 25+ for JVM build).

**Link existing Signal account:**
```bash
signal-cli link -n "OpenClaw"
# Scan the QR code with your Signal app
```

Config:
```json5
channels: {
  signal: {
    enabled: true,
    account: "+15551234567",
    cliPath: "signal-cli",
    dmPolicy: "pairing",
    allowFrom: ["+15557654321"],
  }
}
```

---

## iMessage (via BlueBubbles — recommended)

1. Install BlueBubbles server on a Mac: bluebubbles.app
2. Enable the web API in BlueBubbles, set a password
3. Config:
```json5
channels: {
  bluebubbles: {
    enabled: true,
    serverUrl: "http://192.168.1.100:1234",
    password: "your-password",
    webhookPath: "/bluebubbles-webhook",
  }
}
```
4. In BlueBubbles, set webhooks to point to: `https://your-gateway/bluebubbles-webhook?password=<password>`

---

## Matrix

Plugin required: `openclaw plugins install @openclaw/matrix`

```json5
channels: {
  matrix: {
    enabled: true,
    homeserver: "https://matrix.example.org",
    accessToken: "syt_xxx",
    dm: { policy: "pairing" },
  }
}
```

---

## Microsoft Teams

Plugin required: `openclaw plugins install @openclaw/msteams`

1. Create Azure Bot → get App ID, Client Secret, Tenant ID
2. Enable Teams channel in Azure Bot
3. Config:
```json5
channels: {
  msteams: {
    enabled: true,
    appId: "<APP_ID>",
    appPassword: "<CLIENT_SECRET>",
    tenantId: "<TENANT_ID>",
    webhook: { port: 3978, path: "/api/messages" },
  }
}
```
4. Set Azure Bot messaging endpoint to `https://<host>:3978/api/messages`

---

## DM Policies (applies to all channels)

| Policy | Behavior |
|--------|----------|
| `pairing` (default) | Unknown senders get a code. Bot ignores them until you approve. Codes expire in 1 hour. |
| `allowlist` | Only senders in `allowFrom` array get through. Everyone else silently blocked. |
| `open` | Anyone can message. Requires explicit `allowFrom: ["*"]`. |
| `disabled` | No DMs accepted. |

**Approve pending pairings:**
```bash
openclaw pairing list <channel>
openclaw pairing approve <channel> <CODE>
```
