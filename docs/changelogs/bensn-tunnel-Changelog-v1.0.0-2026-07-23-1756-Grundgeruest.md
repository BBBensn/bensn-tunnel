---
date_created: 2026-07-23 17:56:00
type: changelog
tags:
  - project
  - changelog
date_modified: 2026-07-23 17:56:00
---

# v1.0.0 — Grundgerüst WireGuard-Tunnel + Jellyfin (2026-07-23)
- WireGuard-Server auf Hetzner eingerichtet (`/etc/wireguard/wg0.conf`), Keypaare für
  Hetzner und NAS generiert, Service via `wg-quick@wg0` aktiv und enabled
- nginx-Vhost `stream.bensn.me` deployed: Reverse Proxy auf `10.10.0.2:30013` (Jellyfin
  NodePort), Websocket-Support über eigene `$connection_upgrade`-Map in
  `/etc/nginx/conf.d/websocket-upgrade.conf`, Buffering für Streaming deaktiviert
- Let's-Encrypt-Zertifikat für `stream.bensn.me` via `certbot --nginx` bezogen (Auto-Renew
  eingerichtet)
- DNS A-Record `stream.bensn.me → 178.104.133.228` vom Nutzer bereits vorab angelegt
- Domain-Namen im Projekt vereinheitlicht: `stream.bensn.me` statt ursprünglich geplantem
  `jellyfin.bensn.me` (DNS-Eintrag existierte schon für `stream`)
- Docker-Compose-Template für WireGuard-Client auf TrueNAS erstellt
  (`wireguard/nas-docker-compose.yml.example`, `network_mode: host`), Einrichtung vor Ort
  durch Nutzer noch offen
- Festgestellt: Hetzner-`ufw` ist inaktiv, `51820/udp` war daher ohnehin offen — keine
  Firewall-Änderung vorgenommen (Security-Settings fasst Claude Code nicht an)
- Git-Repo lokal initialisiert und mit `git@github.com:BBBensn/bensn-tunnel.git` verbunden
