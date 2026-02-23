---
title: Docker Stack
layout: default
parent: Homelab
nav_order: 1
---

# Docker Stack

Running Docker on an Ubuntu server (192.168.1.10). Most services are just `docker compose up -d` with a simple compose file and an `.env` for secrets.

<!-- screenshot: portainer dashboard or docker ps output -->
*![](img/docker-portainer.png)*

## Services

| Container | Port | Notes |
|-----------|------|-------|
| Nextcloud | 8080 | File sync, calendar, contacts |
| Plex | 32400 | Media server, main library |
| Jellyfin | — | Backup media server |
| open-webui | 3000 | Local LLM frontend (Ollama) |
| Portainer | 9443 | Docker management UI |
| cloudflared | — | Cloudflare tunnel for external access |
| netdata | — | Server monitoring |

## External Access

Nextcloud is exposed externally via a Cloudflare tunnel — no port forwarding needed. The `cloudflared` container connects out to Cloudflare and tunnels traffic to `localhost:8080`. Domain is `makonis.qqcyber.com`.

Everything else stays LAN-only.

## Notes

- Portainer is useful but I mostly just use `docker compose` directly
- Ollama runs on the host (not in Docker) because GPU passthrough inside containers is more trouble than it's worth on this setup
- Plex transcodes fine with software — haven't needed to set up hardware transcoding yet

```bash
# quick status check
docker ps --format "table {% raw %}{{.Names}}\t{{.Status}}\t{{.Ports}}{% endraw %}"
```
