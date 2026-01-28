# Media Server Stack

A comprehensive media server setup using Docker containers, featuring automated media management, VPN integration, and a unified dashboard interface.

## Overview

This project sets up a complete media server environment with the following components:

- **VPN Gateway (Gluetun)**: Provides secure VPN connectivity for torrent-related services
- **qBittorrent**: Torrent client with web interface
- **Prowlarr**: Indexer manager for automated media downloads
- **Radarr**: Movie collection manager
- **Sonarr**: TV show collection manager
- **Plex**: Media server
- **Dashboard**: Web interface to monitor and access all services

## Prerequisites

- Docker and Docker Compose installed
- VPN account (example is using Surfshark)
- WireGuard configuration from your VPN provider
- Basic understanding of Docker and networking concepts

## Directory Structure

```
.
├── config/           # Configuration files for all services (git-backed)
├── compose.yml
```

## Services

### VPN Gateway (Gluetun)
- Acts as a VPN gateway for torrent-related services
- Uses WireGuard protocol
- Routes traffic for qBittorrent and Prowlarr through VPN
- Ports: 8081, 6881 (TCP/UDP), 9696

### qBittorrent
- Web interface accessible at `http://localhost:8081`
- Downloads routed through VPN
- Stores completed downloads in `/mnt/ext_ssd/media/downloads`
- Media files stored in `/mnt/ext_ssd/media`

### Prowlarr
- Web interface accessible at `http://localhost:9696`
- Manages indexers for automated downloads
- Traffic routed through VPN

### Radarr
- Web interface accessible at `http://localhost:7878`
- Manages movie collection
- Direct internet connection (no VPN)
- Monitors and organizes movie downloads

### Sonarr
- Web interface accessible at `http://localhost:8989`
- Manages TV show collection
- Direct internet connection (no VPN)
- Monitors and organizes TV show downloads

### Plex
- Web interface accessible at `http://localhost:32400/web`
- Media library at `/mnt/ext_ssd/media`

## Setup Instructions

1. Clone this repository
2. Configure VPN settings in `compose.yml`:
   - Set your WireGuard private key
   - Configure your WireGuard IP address
   - Adjust server location if needed

3. Create necessary directories:
   ```bash
   mkdir -p config
   ```

4. Start the services:
   ```bash
   docker compose -f compose.yml up -d
   ```

5. Access the dashboard by opening `dashboard.html` in your web browser

## Configuration

### VPN Configuration
Edit the following environment variables in `compose.yml`:
- `WIREGUARD_PRIVATE_KEY`: Your WireGuard private key
- `WIREGUARD_ADDRESSES`: Your assigned WireGuard IP
- `SERVER_COUNTRIES`: Your preferred VPN server location

### Service Configuration
Each service's configuration is stored in the `config/` directory:
- `config/gluetun/`: VPN configuration
- `config/qbittorrent/`: Torrent client settings
- `config/prowlarr/`: Indexer settings
- `config/radarr/`: Movie management settings
- `config/sonarr/`: TV show management settings
- `config/plex/`: Plex settings and database

### Storage Layout (Busybee)
- Configs (git-backed): `/home/james/docker/mediaserver/config`
- Media: `/mnt/ext_ssd/media`
- Downloads: `/mnt/ext_ssd/media/downloads`

### Hardlink-Friendly Paths
- Host mount: `/mnt/ext_ssd/media`
- Container path (all apps): `/data`
- Downloads: `/data/downloads`
- Movies: `/data/movies`
- TV: `/data/tv`

## Security Considerations

- VPN credentials are stored in environment variables
- Services behind VPN (qBittorrent, Prowlarr) are protected
- Media management services (Radarr, Sonarr) use direct connection
- All services use non-root user (PUID/PGID: 1000)

## Maintenance

- Regular updates: `docker compose -f compose.yml pull`
- Restart services: `docker compose -f compose.yml restart`
- View logs: `docker compose -f compose.yml logs -f [service_name]`
- Stop all services: `docker compose -f compose.yml down`

## Troubleshooting

1. VPN Connection Issues:
   - Check WireGuard credentials
   - Verify server availability
   - Check Gluetun logs

2. Service Access Issues:
   - Verify ports are not in use
   - Check service logs
   - Ensure proper network configuration

3. Download Issues:
- Verify VPN connection
- Check disk space
- Verify download paths
- Ensure qBittorrent temp/incomplete downloads are on `/mnt/ext_ssd/media/downloads`
