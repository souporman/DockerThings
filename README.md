# DockerThings

Portainer-managed Git stacks for a small homelab: **SABnzbd, Sonarr, Radarr, Bazarr, Heimdall, Pi-hole, Cloudflared**.

> This repo stores the **Compose files** only. Secrets/tokens stay out of Git.

---

You can deploy stacks individually in Portainer by pointing to each file under `stacks/`.

---

## Prereqs

- Docker Engine + Portainer (**Standalone**)
- Host folders created with correct ownership/permissions, e.g.
  - `/srv/docker/<app>/config` for app configs
  - `/mnt/plex` (and subfolders) for media/downloads
- A private GitHub repo (this one), with a read-only auth method for Portainer:
  - **Fine-grained PAT** (Repository access → _Only select repositories_ → this repo; Permission **Contents: Read**) _or_
  - **SSH deploy key** (read-only)

---

## Deploy from Git in Portainer (one stack at a time)

1. **Stacks → Add stack → Repository** tab
2. **Repository URL:** `https://github.com/<you>/DockerThings.git`
3. **Repository reference:** `main` (or `refs/heads/main`)
4. **Compose path:** one of:
   - `stacks/sabnzbd.yml`
   - `stacks/radarr.yml`
   - `stacks/sonarr.yml`
   - `stacks/bazarr.yml`
   - `stacks/pihole.yml` 
   - `stacks/cloudflared.yml` (add TUNNEL_TOKEN env variable)
   - `stacks/heimdall.yml` )
5. **Authentication:** your GitHub username + token _or_ SSH deploy key
6. (Optional) **Auto-update:** enable polling or configure a webhook (Repo → Settings → Webhooks → push event)
7. **Deploy the stack**

---

## Stack Overview & Interactions

### SABnzbd
- **Role:** Usenet downloader; fetches requested content.
- **Communicates with:**
  - Radarr & Sonarr (via API at `http://<host-ip>:8888`).
  - Shares completed files through `/downloads`.

### Radarr (Movies)
- **Role:** Automates movie acquisition and library management.
- **Communicates with:**
  - SABnzbd (to request downloads and monitor status).
  - Shared `/downloads` folder (to import completed movies).
  - Writes organized files into `/movies`.

### Sonarr (TV Shows)
- **Role:** Automates TV show acquisition and library management.
- **Communicates with:**
  - SABnzbd (to request downloads and monitor status).
  - Shared `/downloads` folder (to import completed episodes).
  - Writes organized files into `/tv`.

### Bazarr (Subtitles)
- **Role:** Fetches and manages subtitles for existing media.
- **Communicates with:**
  - Radarr & Sonarr libraries directly (`/movies`, `/tv`).
  - Does not interact with SABnzbd or `/downloads`.

### Heimdall
- **Role:** Web dashboard / homepage for all services.
- **Communicates with:**
  - None directly — provides quick links to other stack UIs.

### Watchtower
- **Role:** Monitors running containers and auto-updates images when new versions are released.
- **Communicates with:**
  - Docker Engine API (manages all stacks).

### Uptime Kuma
- **Role:** Self-hosted monitoring tool for services, sites, and endpoints.
- **Communicates with:**
  - Periodically queries Radarr, Sonarr, SABnzbd, etc. to check uptime and status.
  - Provides its own dashboard for alerts/notifications.

### Overseerr
- **Role:** Request management system for media (front-end for Radarr/Sonarr).
- **Communicates with:**
  - Radarr & Sonarr APIs (to create and manage requests).
  - SABnzbd indirectly via Radarr/Sonarr.

### Tautulli
- **Role:** Analytics and monitoring for Plex usage.
- **Communicates with:**
  - Plex Media Server API (not directly tied to Radarr/Sonarr/SAB).

---

## How They Fit Together

- **SABnzbd** is the download engine.  
- **Radarr & Sonarr** act as the brains: they decide what to download, instruct SABnzbd, then import finished content.  
- **Bazarr** adds subtitles once movies/TV are in place.  
- **Overseerr** provides a user-friendly interface for requesting media, passing requests to Radarr/Sonarr.  
- **Heimdall** is the user-facing dashboard for quick access.  
- **Watchtower** keeps all stacks up-to-date automatically.  
- **Uptime Kuma** monitors all services for availability.  
- **Tautulli** reports on Plex usage and viewing activity.

---

## Service notes

### SABnzbd
- UI: `http://<host-ip>:8888`
- Volumes (example):
  - `/srv/docker/sabnzbd/config:/config`
  - `/mnt/plex/Downloading:/incomplete`
  - `/mnt/plex:/downloads`
- In SAB **Folders**:
  - _Temporary Download Folder_ = `/incomplete`
  - _Completed Download Folder_ = `/downloads`

### Radarr & Sonarr
- Must see the **same inside-container path** as SAB: `/downloads`
- Example volumes:
  - Radarr: `/srv/docker/radarr/config:/config`, `/mnt/plex:/downloads`, `/mnt/plex/Movies:/movies`
  - Sonarr: `/srv/docker/sonarr/config:/config`, `/mnt/plex:/downloads`, `/mnt/plex/TV:/tv`
- If deployed as separate stacks, set the download client URL to `http://<host-ip>:8888` (host port).

### Bazarr
- Reads libraries directly (`/movies`, `/tv`). No `/downloads` needed.

### Pi-hole (DNS only)
- UI: `http://<host-ip>:8088`
- Bind DNS ports to a specific LAN IP to avoid host conflicts:
  ```yaml
  ports:
    - "<host-ip>:53:53/tcp"
    - "<host-ip>:53:53/udp"
    - "8088:80"

### cloudflared
- Don't forget to add your TUNNEL TOKEN env variable when you create the stack.  

