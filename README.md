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

No extra container needed — just mount `index.html` into the existing Caddy.

### 1. Clone

```bash
cd /root
git clone <repo-url> zynerio-redirect
```

### 2. Mount into Caddy

Add a volume to the Caddy service in Remnawave's `docker-compose.yml`:

```yaml
volumes:
  - /root/zynerio-redirect/index.html:/srv/connect/index.html:ro
```

### 3. Update Caddyfile

Replace the `https://zynerio.com` block:

```caddyfile
https://zynerio.com {
    handle /add_sub/config {
        rewrite * /assets/.app-config-v2.json
        reverse_proxy remnawave-subscription-page:3010
    }
    handle /add_sub* {
        root * /srv/connect
        try_files {path} /index.html
        file_server
    }
    respond "OK" 200
}
```

The `/add_sub/config` route proxies to the Remnawave subscription page to fetch
the latest app deep-link schemes and store links dynamically.

### 4. Restart Caddy

```bash
docker compose restart caddy
```

### 5. Verify

```bash
curl https://zynerio.com/health
curl "https://zynerio.com/add_sub?sub=test"
curl https://zynerio.com/add_sub/config | head -c 200
```

## Updating

```bash
cd /root/zynerio-redirect && git pull
docker compose restart caddy
```

## Stack

- **Single HTML file** — no build step, no dependencies
- Served by the existing Caddy instance
