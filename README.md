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

