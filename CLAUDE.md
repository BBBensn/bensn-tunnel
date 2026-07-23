---
date_created: 2026-07-23
date_modified: 2026-07-23
---

# bensn-tunnel — CLAUDE.md

Projekt-spezifischer Kontext. Ergänzt `~/.claude/CLAUDE.md`.
Ablageort: `~/Documents/Coding/bensn-tunnel/CLAUDE.md`

---

## Projekt-Basics

- **Name:** bensn-tunnel
- **Domain:** stream.bensn.me (v1.0.0), weitere Subdomains folgen (Cloud/Nextcloud geplant)
- **Version:** v1.0.0
- **Status:** active — Hetzner-Seite (WireGuard-Server, nginx, Zertifikat) live. NAS-Client
  (Ort: direkt auf TrueNAS via Docker-Compose) noch vom Nutzer vor Ort einzurichten.
- **Stack:** WireGuard (VPN-Tunnel Hetzner ↔ NAS) + nginx (Reverse Proxy). Kein klassisches
  Frontend/Backend — reines Infra-/Netzwerk-Projekt.

---

## Lokale Struktur

~/Documents/Coding/bensn-tunnel/
├── wireguard/
│ ├── hetzner-wg0.conf.example
│ ├── nas-wg0.conf.example
│ ├── nas-docker-compose.yml.example  ← WireGuard-Client als Docker-Container auf TrueNAS
│ ├── hetzner-wg0.conf   ← echte Config mit Keys, NICHT committed (siehe .gitignore)
│ └── nas-wg0.conf       ← echte Config mit Keys, NICHT committed (siehe .gitignore)
├── nginx/
│ ├── stream.bensn.me
│ └── websocket-upgrade.conf  ← $connection_upgrade Map, geht nach /etc/nginx/conf.d/
├── docs/
│ └── changelogs/
├── CLAUDE.md
└── .gitignore ← echte *-wg0.conf (mit Keys) ausschließen, nur .example committen


---

## Remote-Struktur

Hetzner (178.104.133.228):
/etc/wireguard/wg0.conf ← WireGuard-Server-Config
/etc/nginx/sites-enabled/stream.bensn.me
/etc/nginx/conf.d/websocket-upgrade.conf

NAS "atlas" (192.168.0.63, zuhause hinter NAT):
WireGuard-Client via Docker-Compose (linuxserver/wireguard, network_mode: host),
direkt auf TrueNAS SCALE. Config-Pfad auf dem NAS: vom Nutzer selbst festzulegen
(z. B. via Custom App "Install via YAML" oder manuell per docker compose up -d).


---

## Services & Ports

| Dienst | Port | Wo |
|--------|------|-----|
| WireGuard | 51820/udp | Hetzner (einzig neu geöffneter Port) |
| Jellyfin intern | 30013 (TrueNAS-App-NodePort) | NAS, über Tunnel (10.10.0.2:30013) |
| Tunnel-Subnetz | 10.10.0.0/24 | Hetzner = .1, NAS = .2 |

⚠️ Hetzner-Firewall (`ufw`) ist aktuell **inaktiv** — kein Port ist dort gesperrt, 51820/udp
war also ohnehin offen. Falls eine Firewall gewünscht ist, muss der Nutzer sie selbst
einrichten (Claude Code fasst Security-Settings nicht an).

---

## Deploy

```bash
# WireGuard-Config auf Hetzner
scp wireguard/hetzner-wg0.conf bensn:/etc/wireguard/wg0.conf
ssh bensn systemctl restart wg-quick@wg0

# nginx-Vhost + Websocket-Map
scp nginx/websocket-upgrade.conf bensn:/etc/nginx/conf.d/websocket-upgrade.conf
scp nginx/stream.bensn.me bensn:/etc/nginx/sites-enabled/stream.bensn.me
ssh bensn nginx -t && ssh bensn systemctl reload nginx

# Zertifikat (einmalig — certbot editiert die Vhost-Config automatisch,
# danach lokale Kopie in nginx/stream.bensn.me mit dem Server-Stand abgleichen)
ssh bensn certbot --nginx -d stream.bensn.me
```

WireGuard-Config auf dem NAS wird direkt vor Ort eingerichtet (TrueNAS UI oder Shell),
kein SCP-Deploy von hier aus — NAS sitzt hinter Heim-NAT und ist von hier aus nicht erreichbar.

---

## Git

- **Repo:** `https://github.com/BBBensn/bensn-tunnel`
- **Remote:** `git remote add origin git@github.com:BBBensn/bensn-tunnel.git`

```bash
git add .
git commit -m "Add WireGuard tunnel + stream.bensn.me nginx vhost"
git push origin main
```

⚠️ Nur `.example`-Configs (ohne Keys/Secrets) committen. Echte `wg0.conf` mit Private Keys
bleibt lokal/auf dem Server, nie im Repo.

---

## Auth

- [ ] Auth via `auth.bensn.me` (nginx `auth_request`)
- [x] Öffentlich erreichbar, aber Auth via Jellyfin-eigenes Login (kein bensn-auth davor,
      wegen mobiler Jellyfin-Apps)

---

## Projekt-spezifische Konventionen

- WireGuard-Client-Seite braucht `PersistentKeepalive = 25` (NAS sitzt hinter Heim-NAT,
  sonst bricht der Tunnel nach Inaktivität ab)
- Am Heimrouter wird **kein** Port geöffnet — der Tunnel wird immer vom NAS aus initiiert
- Am Hetzner-Server wird **nur** `51820/udp` neu geöffnet, sonst nichts
- Neue Subdomain für weiteren Dienst über den Tunnel (z. B. später `cloud.bensn.me`):
  einfach neuer nginx-Vhost mit `proxy_pass http://10.10.0.2:<port>`, kein neuer Tunnel nötig
- Vaultwarden ist explizit NICHT Teil dieses Projekts (läuft direkt auf Hetzner, ohne Tunnel)

---

## Roadmap

| Version | Feature | Status |
|---------|---------|--------|
| v1.0.0 | WireGuard-Tunnel Hetzner↔NAS + Jellyfin über stream.bensn.me erreichbar | Hetzner-Seite ✅ done, NAS-Client offen (Nutzer) |
| v1.1.0 | Cloud-Dienst (Nextcloud o.ä.) auf NAS über selben Tunnel erreichbar machen | geplant |
| v1.2.0 | Monitoring/Alerting für Tunnel-Verfügbarkeit (z. B. via Grafana/weather-stack-Muster) | geplant |

---

## NAS-Setup (noch zu erledigen, vor Ort)

1. Ordner auf dem NAS anlegen, z. B. `/mnt/<pool>/wireguard-tunnel/`
2. `wireguard/nas-docker-compose.yml.example` dorthin kopieren (ohne `.example`) und
   `nas-wg0.conf` (echte Datei, liegt lokal unter `wireguard/nas-wg0.conf`, NICHT im Git-Repo)
   nach `<ordner>/config/wg_confs/wg0.conf` legen — **wichtig:** `linuxserver/wireguard`
   erwartet die Interface-Config genau in `/config/wg_confs/`, nicht direkt in `/config/`
3. `docker compose -f nas-docker-compose.yml up -d` (SSH-Zugriff auf TrueNAS nötig) —
   läuft mit `network_mode: host`, damit Tunnel-Traffic auf `10.10.0.2:30013` direkt beim
   Jellyfin-NodePort auf dem NAS-Host ankommt
4. Prüfen: von Hetzner aus `ping 10.10.0.2` sollte antworten
5. Test: `https://stream.bensn.me` im Browser öffnen

---

## Obsidian-Doku

- Projekt-MD: `03_Projects/Coding PC/bensn-tunnel/bensn-tunnel.md`
- Changelogs: `03_Projects/Coding PC/bensn-tunnel/Changelogs/`
- Changelog-All: `03_Projects/Coding PC/bensn-tunnel/bensn-tunnel-Changelog-All.md`