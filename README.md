# Local ARR Stack - Media Automation Docker Project

A comprehensive, self-hosted media automation stack designed to streamline your home media library management, automated downloading, and organization using Docker containers.

## 🎬 What Is This Project?

This project provides a complete "Local ARR" (Auto-Release Requester) stack that automates the entire process of finding, downloading, and organizing movies and TV series into your media collection. It's built on top of the popular localNAS stack configuration and enhanced with Docker networking for simplified access.

## ✨ Features

### Core Media Management
- **Radarr** - Automated movie management and download
- **Sonarr** - Automated TV series management and download
- **Bazarr** - Subtitle management for movies and TV shows
- **Trailarr** - Movie trailer collection automation
- **Prowlarr** - Universal indexer aggregator and controller
- **Jackett** - Torrent tracker indexer support

### Download Clients
- **qBittorrent** - BitTorrent client with VPN support
- **SABnzbd** - NZB file downloader
- **NZBGet** - Alternative NZB download client

### Infrastructure & Networking
- **Portainer** - Docker management UI for container control
- **Nginx** - Reverse proxy with virtual host routing
- **Gluetun** - VPN-based secure downloads (ProtonVPN configured)
- **Deunhealth** - Docker health check monitoring

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     External Network                         │
│              (Port 80/443 via Nginx Proxy)                   │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    Local Stack (Docker)                      │
│                                                              │
│   ┌──────────────┐  ┌─────────────────────────────────────┐ │
│   │   Nginx      │→│          Gluetun (VPN Container)    │ │
│   └──────────────┘  ├─────────────────────────────────────┤ │
│                     │                                     │ │
│         ┌───────────▼─────────────┐     ┌───────────────┐ │
│         │    Docker Services      │←────│   Portainer   │ │
│         │  (via gluetun service)  │     │               │ │
│         ├─────────────────────────┤     └───────────────┘ │
│         │                         │                      │
│         │    Prowlarr (9696)    ↓   Jackett (9117)      │
│         │    Sonarr  (8989)     ↓                       │
│         │    Radarr   (7878)    ↓                       │
│         │    Bazarr   (6767)    ↓                       │
│         │    Trailarr (7889)    ↓                       │
│         │    qBittorrent(8080)  ↓   SABnzbd (8085)      │
│         │    NZBGet    (6789)     ↓   Gluetun Ports...   │
│         └─────────────────────────┘                      │
└─────────────────────────────────────────────────────────────┘
                              ↓
                    NAS Mounts (/data/*)
```

## 📋 System Requirements

- Docker Engine 20.10+
- Linux-based host (Ubuntu, Debian, Arch, etc.)
- At least 4GB RAM recommended
- NAS mount accessible via SMB/CIFS or direct path
- Valid VPN account (currently configured for ProtonVPN)

## 🚀 Quick Start Guide

### Prerequisites

1. **Create Docker Volume Directories**
   ```bash
   sudo mkdir -p /mnt/nas/downloads/{qbittorrent,sabnzbd,nzbget}
   sudo mkdir -p /data/media/{movies,series}
   ```

2. **Configure VPN Credentials**
   
   Create `.env` file in `local-stack/` directory:
   ```bash
   cd local-stack
   nano .env
   ```
   
   Add your ProtonVPN credentials:
   ```
   WIREGUARD_PRIVATE_KEY=your_wireguard_private_key_here
   WIREGUARD_ADDRESSES=10.64.20.4/32
   TZ=Europe/Bucharest
   SERVER_COUNTRIES=Romania
   SERVER_CITIES=Bucharest
   ```

3. **Configure Hosts File** (for local DNS access)
   
   Add to `/etc/hosts`:
   ```
   127.0.0.1 prowlarr.local
   127.0.0.1 jackett.local
   127.0.0.1 sonarr.local
   127.0.0.1 radarr.local
   127.0.0.1 bazarr.local
   127.0.0.1 trailarr.local
   127.0.0.1 qbittorrent.local
   127.0.0.1 sabnzbd.local
   127.0.0.1 nzbget.local
   ```

4. **Set Permissions**
   All containers use `PUID=1000` and `PGID=1000` for consistent permissions across your system.

### Deploy the Stack

```bash
# From the project root: local-arr-stack/

# Navigate to stack directory
cd local-stack

# Pull all required images
docker-compose pull

# Start all services
docker-compose up -d
```

## 🔌 Service Ports Reference

| Container   | Port  | Protocol | Service               |
|-------------|-------|----------|----------------------|
| nginx       | 80    | TCP/UDP  | External proxy access |
| prowlarr    | 9696  | TCP      | Indexer aggregator    |
| jackett     | 9117  | TCP      | Tracker manager       |
| sonarr      | 8989  | TCP      | TV series manager     |
| radarr      | 7878  | TCP      | Movie manager         |
| bazarr      | 6767  | TCP      | Subtitle manager      |
| trailarr    | 7889  | TCP      | Trailer collector     |
| qbittorrent | 8080  | TCP      | Web UI                |
| qbittorrent | 6881  | TCP      | Torrent port          |
| sabnzbd     | 8085  | TCP      | NZB downloader        |
| nzbget      | 6789  | TCP      | NZB downloader        |
| gluetun     | 9696+ | Various  | VPN proxy (all ports) |

## 📁 Directory Structure

```
local-arr-stack/
├── README.md                          # This file
├── local-portainer/                   # Portainer management UI
│   └── docker-compose.yaml            # Portainer configuration
└── local-stack/                       # Main application stack
    ├── docker-compose.yaml            # Service definitions
    ├── nginx/
    │   └── nginx.conf                 # Nginx reverse proxy config
    ├── .env                           # Environment variables (VPN)
    └── README.md                      # Additional stack docs
```

## ⚙️ Configuration Options

### Environment Variables

All services support standard LinuxServer environment variables:

- `PUID` - User ID (default: 1000)
- `PGID` - Group ID (default: 1000)
- `TZ` - Timezone (currently Europe/Bucharest)
- `WEBUI_PORT` - qBittorrent web interface port

### Nginx Configuration

The nginx.conf uses server blocks with virtual host routing:
- Each service gets its own `.local` domain
- Automatic HTTPS can be added via Let's Encrypt (requires additional config)

## 🔧 Maintenance Commands

```bash
# Check status
docker-compose ps

# View logs
docker-compose logs -f

# Restart services
docker-compose restart <service>

# Stop all services
docker-compose down

# Remove volumes (WARNING: destroys all data!)
docker-compose down -v
```

## 🛠️ Troubleshooting

### Common Issues

1. **Permission errors on NAS mounts**
   - Ensure PUID=1000 and PGID=1000 are set in environment variables
   - Run `chown` commands to fix ownership

2. **Containers won't start**
   - Check `.env` file is present with valid VPN credentials
   - Review logs: `docker-compose logs`

3. **Cannot access services via browser**
   - Verify `/etc/hosts` entries are correct
   - Check nginx configuration syntax: `nginx -t`
   - Ensure firewall allows port 80 traffic

4. **VPN connection fails**
   - Validate WireGuard private key in `.env` file
   - Check ProtonVPN server availability
   - Review Gluetun logs for specific errors

## 📚 Additional Resources

- [Portainer Documentation](https://www.portainer.io/docs)
- [LinuxServer Container Images](https://docs.linuxserver.io/general/environment-variable-documentation)
- [Radarr Documentation](https://radarr.video/documentation/)
- [Sonarr Documentation](https://sonarr.tv/)
- [ProtonVPN Setup Guide](https://protonvpn.com/help/clients/proton-vpn-cli)

## 📄 License

This project is provided as-is for personal use. All included container images are subject to their respective licenses.

---

Built with ❤️ for self-hosted media enthusiasts!