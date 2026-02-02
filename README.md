# Media Automation Stack

A Docker Compose setup for automated media management running on TrueNAS, deployed via Dockge for easy stack management.

## Services

- **Gluetun** - VPN client (Surfshark) that routes all traffic for the *arr stack
- **qBittorrent** - Torrent client with WebUI on port 8080
- **Prowlarr** - Indexer manager on port 9696
- **Radarr** - Movie collection manager on port 7878
- **Sonarr** - TV show collection manager on port 8989
- **FlareSolverr** - Cloudflare bypass proxy on port 8191

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

## Access

Once running, access the services at:
- qBittorrent: http://truenas-ip:8080
- Prowlarr: http://truenas-ip:9696
- Radarr: http://truenas-ip:7878
- Sonarr: http://truenas-ip:8989
- FlareSolverr: http://truenas-ip:8191