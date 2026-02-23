---
title: Homelab - Docker Stack
layout: default
nav_order: 4
---

# Docker Stack
So for learning purposes i decided to set up docker stack on my local server...Then i got addicted.
 Most services are just `docker compose up -d` with a simple compose file and an `.env` for secrets.

<!-- screenshot: portainer dashboard or docker ps output -->
*![](img/docker-portainer.png)*

## Services i use at the moment

| Container | Use Case |
|-----------|----------|
| Nextcloud | File sync, calendar, contacts |
| Plex | Media server, main library |
| Jellyfin | Backup media server |
| open-webui | Local LLM frontend (Ollama) |
| Portainer | Docker management UI |
| cloudflared | Cloudflare tunnel for external access |
| netdata | Server monitoring |
| mcbot | Minecraft server bot |


## External Access

Nextcloud is exposed externally via a Cloudflare tunnel — no port forwarding needed. The `cloudflared` container connects out to Cloudflare and tunnels traffic to `localhost:` and it's free. And if i want to add another service i just create subdomain and boom-done.

Everything else stays LAN-only.

## Notes

- Portainer is useful but I mostly just use `docker compose` directly, also it broke and i haven't gotten to fixing it. who need GUI anyway.
- Ollama runs on the host (not in Docker) because GPU passthrough inside containers is more trouble than it's worth on this setup
- Plex transcodes fine with software — haven't needed to set up hardware transcoding yet

