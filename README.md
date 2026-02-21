# Zynerio Connect

1-click connect redirect service for Zynerio VPN.

When a user taps the "Connect" button in the Telegram bot, the bot sends an HTTPS link to this service. The page sequentially tries deep-link URI schemes for supported VPN apps (Happ, FLClash, Hiddify, V2RayNG, Sing-Box). If none are detected, it shows a download button and a manual subscription key.

## How it works

```
User taps button → opens HTTPS URL → JS tries deep-link URIs → VPN app opens with subscription
```

Query parameters:
| Param | Description |
|-------|-------------|
| `sub` | Full subscription URL (required) |
| `app` | App slug: `happ`, `flclash`, `v2raytun`, or omit for default order |

Example: `https://zynerio.com/add_sub?sub=https://panel.example.com/api/sub/abc123&app=happ`

## Supported apps

| App | URI Scheme |
|-----|-----------|
| Happ | `happ://add/{url}` |
| FLClash / Clash | `clash://install-config?url={encoded}` |
| Hiddify | `hiddify://import/{url}` |
| V2RayNG | `v2rayng://install-config?url={encoded}` |
| Sing-Box | `sing-box://import-remote-profile?url={encoded}` |

## Deployment

### 1. Clone & configure

```bash
git clone <repo-url> && cd ZynerioConnect
cp .env.example .env
# Edit .env if you need a different port
```

### 2. Start

```bash
docker compose up -d
```

The service listens on port **8070** (configurable via `PORT` in `.env`).

### 3. Reverse proxy (main Caddy)

Add to your site's Caddyfile:

```caddyfile
handle /add_sub* {
    reverse_proxy localhost:8070
}
```

Then `caddy reload`.

## Stack

- **Caddy 2** (Alpine) — static file server
- **Single HTML file** — no build step, no dependencies
- **Docker Compose** — deploy and forget
