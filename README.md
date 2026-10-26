# Media Automation Stack

A Docker Compose setup for automated media management running on TrueNAS, deployed via Dockge for easy stack management.

## Services

- **Gluetun** - VPN client (Surfshark) that routes all traffic for the *arr stack
- **qBittorrent** - Torrent client with WebUI on port 8080
- **Prowlarr** - Indexer manager on port 9696
- **Radarr** - Movie collection manager on port 7878
- **Sonarr** - TV show collection manager on port 8989
- **FlareSolverr** - Cloudflare bypass proxy on port 8191
- **Doplarr** - Discord requesting bot
- **Cleanuparr** - Queue cleaner on port 11011; blocklists malware/failed-import releases and searches for replacements

## Network Architecture

All *arr services and qBittorrent run through Gluetun's network stack, ensuring all traffic is routed through the VPN. FlareSolverr runs on the bridge network to handle Cloudflare-protected indexers.

## Configuration Required

Before deploying, replace these placeholders in the compose file:

- `OPENVPN_USER` - Your Surfshark username
- `OPENVPN_PASSWORD` - Your Surfshark password
- `SERVER_HOSTNAMES` - Surfshark server hostname
- `REPLACE_WITH_VPN_TIMEZONE` - Timezone for FlareSolverr (e.g., America/New_York)

## Deployment via Dockge

This stack is managed through Dockge on TrueNAS:

1. Open Dockge web interface
2. Create a new stack
3. Paste the compose.yaml content
4. Update the configuration placeholders above
5. Click "Start" to deploy the stack

Dockge provides an easy way to start/stop the entire stack and monitor container logs from a single interface.

## Cleanuparr Setup

Cleanuparr shares Gluetun's network, so every service is reachable on `localhost`. In the Cleanuparr web UI:

1. **Download Clients** - add qBittorrent at `http://localhost:8080` with your WebUI username/password
2. **Arrs** - add Sonarr at `http://localhost:8989` and Radarr at `http://localhost:7878` with their API keys (Settings > General in each app)
3. **Malware Blocker** - enable it and point Sonarr and Radarr at their blocklists (see below). Make sure "delete known malware" / remove-when-nothing-left is on so bad torrents are blocklisted
4. **Queue Cleaner** - enable "Failed Imports" with a low strike count (1-3) so anything Sonarr/Radarr refuses to import is removed and blocklisted
5. **Search** - make sure searching for a replacement after removal is enabled

### Blocklists

[blocklists/](blocklists/) holds the Malware Blocker blacklists, one per arr: `sonarr.txt` and `radarr.txt`. One pattern per line, no comments. `*.exe` matches names ending in `.exe`; `regex:<pattern>` is also supported. Matching files are skipped in qBittorrent, and a torrent with nothing left is removed, blocklisted and re-searched.

Copy them to TrueNAS under Cleanuparr's config folder (`/mnt/crucialssd/cleanuparr/blocklists/`, which is `/config/blocklists/` inside the container):

```sh
# From this repo on your PC
scp blocklists/*.txt truenas_admin@192.168.1.202:/tmp/

# On TrueNAS
sudo mkdir -p /mnt/crucialssd/cleanuparr/blocklists
sudo mv /tmp/sonarr.txt /tmp/radarr.txt /mnt/crucialssd/cleanuparr/blocklists/
sudo chown -R 568:568 /mnt/crucialssd/cleanuparr/blocklists
```

In Cleanuparr > Malware Blocker, set for each arr:

| Arr    | Blocklist Path                   | Blocklist Type |
|--------|----------------------------------|----------------|
| Sonarr | `/config/blocklists/sonarr.txt`  | Blacklist      |
| Radarr | `/config/blocklists/radarr.txt`  | Blacklist      |

After editing a list, copy it over again and restart Cleanuparr.

## Access

Once running, access the services at:
- qBittorrent: http://truenas-ip:8080
- Prowlarr: http://truenas-ip:9696
- Radarr: http://truenas-ip:7878
- Sonarr: http://truenas-ip:8989
- Cleanuparr: http://truenas-ip:11011
- FlareSolverr: http://truenas-ip:8191