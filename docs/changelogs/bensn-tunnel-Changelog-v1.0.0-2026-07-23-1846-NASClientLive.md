---
date_created: 2026-07-23 18:46:00
type: changelog
tags:
  - project
  - changelog
date_modified: 2026-07-23 18:46:00
---

# v1.0.0 — NAS-Client live, Tunnel end-to-end getestet (2026-07-23)
- WireGuard-Client als Docker-Container (`linuxserver/wireguard`, `network_mode: host`) auf
  TrueNAS SCALE eingerichtet unter `/mnt/atlas/wireguard-tunnel/`
- Vier Probleme beim Ersteinrichten gefunden und behoben (Details in `CLAUDE.md` →
  „NAS-Setup"):
  1. Compose-Volume zeigte auf falschen Pfad (`linuxserver/wireguard` braucht
     `/config/wg_confs/wg0.conf`, nicht `/config/wg0.conf`)
  2. `sysctls`-Eintrag in Compose war inkompatibel mit `network_mode: host`, entfernt
  3. TrueNAS-Konflikt zwischen DHCP-Interface und manuell gesetztem Default-Gateway führte
     dazu, dass die NAS gar keine Default-Route hatte — Gateway-Feld in Global Configuration
     geleert
  4. Hetzner Cloud-Firewall (`firewall-1`, separat vom OS-`ufw`) hatte kein Regel für
     UDP 51820 — WireGuard-Pakete wurden auf Infrastruktur-Ebene verworfen, bevor sie den
     Server erreichten. Regel ergänzt.
- WireGuard-Handshake steht, Tunnel bidirektional bestätigt (`wg show` zeigt Traffic auf
  beiden Seiten)
- `https://stream.bensn.me` liefert Jellyfin aus (getestet: `/` redirected korrekt zu `/web/`,
  `200 OK`)
- SSH-Zugriff auf die NAS eingerichtet (Key-Auth + passwordless sudo für `truenas_admin`),
  für zukünftiges Debugging direkt nutzbar
- v1.0.0 damit vollständig abgeschlossen
