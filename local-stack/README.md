# 🎬 Media Automation Stack: NAS + Docker

This guide ensures your Docker containers talk to your NAS efficiently, your permissions stay in sync, and your local network is easy to navigate.

---

## Host Configuration: NAS Mounts
Before starting the containers, we must "link" the NAS folders to the Docker host.

### Create Mount Points
Run these commands on your Linux host to create the local directories:
```bash
sudo mkdir -p /mnt/nas/movies
sudo mkdir -p /mnt/nas/series
sudo mkdir -p /mnt/nas/downloads
```

### Mount the Shares (SMB/CIFS)
Mount the NAS shares using the specific UID/GID of your Docker user (usually `1000`).
```bash
sudo mount -t cifs -o username=NAS_USER,password=NAS_PASS,uid=1000,gid=1000 //NAS/emby/Movies /mnt/nas/movies
sudo mount -t cifs -o username=NAS_USER,password=NAS_PASS,uid=1000,gid=1000 //NAS/emby/Series /mnt/nas/series
```
> **Pro Tip:** To make these permanent, add them to your `/etc/fstab` file so they remount automatically when the server reboots.

---

## Local DNS: Easy Access Names
To access your services via names like `radarr.local` instead of IP addresses, add these to your host machine's `/etc/hosts` file.

**Run:** `sudo nano /etc/hosts` and paste the following at the bottom:

```text
# Media Stack Local Domains
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
*Note: Unless you use a reverse proxy, you will still need to add the port number in your browser (e.g., `http://radarr.local:7878`).*

---

## Docker-Compose: Volume Mapping
This configuration maps your NAS mounts into the containers. Ensure all services use `PUID=1000` and `PGID=1000` to prevent permission errors.

### Radarr (Movies)
```yaml
radarr:
  image: lscr.io/linuxserver/radarr:latest
  network_mode: "bridge"
  environment:
    - PUID=1000
    - PGID=1000
    - TZ=Europe/Bucharest
  volumes:
    - /mnt/nas/movies:/data/movies
    - /mnt/nas/downloads:/downloads
    - radarr_volume:/config
  ports:
    - "7878:7878"
  restart: unless-stopped
```

### Sonarr (Series)
```yaml
sonarr:
  image: lscr.io/linuxserver/sonarr:latest
  network_mode: "bridge"
  environment:
    - PUID=1000
    - PGID=1000
    - TZ=Europe/Bucharest
  volumes:
    - /mnt/nas/series:/data/series
    - /mnt/nas/downloads:/downloads
    - sonarr_volume:/config
  ports:
    - "8989:8989"
  restart: unless-stopped
```

### Bazarr (Subtitles)
```yaml
bazarr:
  image: lscr.io/linuxserver/bazarr:latest
  network_mode: "bridge"
  environment:
    - PUID=1000
    - PGID=1000
    - TZ=Europe/Bucharest
  volumes:
    - /mnt/nas/movies:/data/movies
    - /mnt/nas/series:/data/series
    - bazarr_volume:/config
  ports:
    - "6767:6767"
  restart: unless-stopped
```

### qBittorrent (VPN + Downloads)
```yaml
qbittorrent:
  image: lscr.io/linuxserver/qbittorrent:latest
  network_mode: "service:gluetun" # Routes traffic through VPN
  environment:
    - PUID=1000
    - PGID=1000
    - TZ=Europe/Bucharest
    - WEBUI_PORT=8080
  volumes:
    - /mnt/nas/downloads:/downloads
    - qbittorrent_volume:/config
  restart: unless-stopped
```

---
