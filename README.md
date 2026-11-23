# homelab

Self-hosted home server infrastructure running on a refurbished HP ProDesk SFF PC (Intel Core i5-9500, 16GB RAM) running Ubuntu Linux, Docker, and containerized microservices.

For a complete visual system topology and data flow map, see the [Architecture Overview](ARCHITECTURE.md).

## Hardware & Architecture

| Component | Specification / Tool | Purpose |
| :--- | :--- | :--- |
| **Host Hardware** | HP ProDesk SFF (Intel Core i5-9500, 16GB DDR4) | Compute node |
| **Storage** | 256GB NVMe (system) + 500GB HDD (data, ext4 at `/home/frodo/hdd`) | OS and containers on NVMe; photos and documents on the HDD |
| **Operating System** | Ubuntu Linux | Base container host |
| **Container Engine** | Docker & Docker Compose (Managed via Portainer) | Microservice orchestration |
| **Networking** | Tailscale | Secure remote access and mesh VPN |
| **Reverse Proxy** | Caddy (`caddy_proxy`) | Internal HTTP routing behind the tunnel (TLS terminates at Cloudflare) |
| **Web Server** | Nginx (`static_website`) | Static portfolio and website hosting |
| **Ingress** | Cloudflare Tunnels (`cloudflare_tunnel`) | Public service exposure without port forwarding |

---

## Repository Structure

```text
homelab/
├── stacks/
│   ├── immich/            # Self-hosted photo and video backup solution
│   ├── paperless/         # Paperless-ngx document management (PostgreSQL + Redis)
│   ├── portainer/         # Portainer CE container management UI
│   ├── stirling-pdf/      # Local PDF utility toolkit
│   ├── uptime-kuma/       # Uptime monitoring with push alerts
│   └── website-tunnel/    # Nginx static server + Cloudflare Tunnel agent stack
└── systemd/               # Host-level systemd overrides (Docker waits for Tailscale at boot)
```

AnythingLLM and its Ollama setup live in a separate repository: [AurelSto/local-ai](https://github.com/AurelSto/local-ai).

## Services & Ports

Published ports are bound to the host's Tailscale address (`<tailscale-ip>`), so they are not reachable from the LAN or the internet. Services with no published port are reachable only inside the Docker network.

| Service | Container Name | Published Binding | Access Method | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **Caddy** | `caddy_proxy` | none | Docker network only, fed by the tunnel | Reverse proxy to the static site |
| **Nginx** | `static_website` | none | Routed via Caddy | Static portfolio & website hosting |
| **Cloudflare Tunnel** | `cloudflare_tunnel` | none (outbound only) | Cloudflare Edge | Secure public ingress without open ports |
| **Portainer** | `portainer` | `<tailscale-ip>:9443` | Tailscale only | Container management UI |
| **Immich** | `immich_server` | `<tailscale-ip>:2283` | Tailscale only | Self-hosted photo and video backup |
| **Paperless-ngx** | `paperless-webserver` | `<tailscale-ip>:8010` | Tailscale only | Document management & OCR indexing |
| **Stirling-PDF** | `stirling-pdf` | `<tailscale-ip>:8080` | Tailscale only | Local PDF utility toolkit |
| **AnythingLLM** | `anythingllm` | `<tailscale-ip>:3001` | Tailscale only | Local AI knowledge base & RAG workspace |
| **Uptime Kuma** | `uptime-kuma` | `<tailscale-ip>:3002` | Tailscale only | Service monitoring and push alerts |

## Security

This stack enforces strict network isolation via Tailscale VPN and container permission hardening. For details on ingress restrictions and storage integrity policies, please see [SECURITY.md](./SECURITY.md).

Summary of the current posture:

- No router port forwards; the public site is reached through an outbound Cloudflare Tunnel.
- Every published container port is bound to the Tailscale address.
- UFW denies incoming traffic by default and allows `tailscale0`; Ollama (`11434`) is also allowed from the AnythingLLM Docker subnet only. Docker bypasses UFW for published ports, so those are restricted by their bind address.
- SSH is key-only, restricted with `AllowUsers`, and reachable only over Tailscale.
- The data drive mount point is immutable while unmounted, so a missing drive makes containers fail instead of writing to the system drive.

## Rebuild From Scratch

If you need to redeploy the entire homelab environment on a fresh Ubuntu host (e.g., lord-of-the-pings), follow these steps:

### 1. Prerequisites & Dependencies
Ensure Docker, Docker Compose, and Git are installed on your Ubuntu host:
```bash
sudo apt update && sudo apt install -y git docker.io docker-compose-v2
sudo usermod -aG docker $USER
```

### 2. Host preparation (before deploying any stack)

1. Install and join Tailscale, and note the host's address (`tailscale ip -4`).
2. Format the data drive as ext4 and add it to `/etc/fstab` with `nofail`. Protect the empty mount point first:
   ```bash
   sudo mkdir -p /home/frodo/hdd && sudo chattr +i /home/frodo/hdd
   echo "UUID=<data-uuid> /home/frodo/hdd ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab
   sudo mount -a && findmnt /home/frodo/hdd
   ```
3. Create the data folders owned by UID 1000: `/home/frodo/hdd/immich` and `/home/frodo/hdd/paperless/{consume,data,export,media}`.
4. Create the external volume used by AnythingLLM: `docker volume create anythingllm_storage`.
5. Create the external volume used by Portainer: `docker volume create portainer_data`.
6. Install the Docker boot-order drop-in so containers bound to the Tailscale address don't fail at boot. See [systemd/README.md](systemd/README.md).
7. Set the firewall: default deny incoming, allow on `tailscale0`, and allow the AnythingLLM subnet to reach port `11434`.
8. Harden SSH with key-only login before closing the firewall to the LAN.

## Known gaps / roadmap

- [ ] **Backups.** Everything lives on single disks, and Immich's database and the Docker volumes are on the system drive. Plan: nightly restic/Kopia to a second drive plus an offsite copy, including database dumps, with a tested restore.
- [ ] **Image pinning.** Pinned by digest: Paperless-ngx, Stirling-PDF, Uptime Kuma, Portainer, Valkey, nginx, cloudflared, Caddy, and AnythingLLM (in the `local-ai` repo). Tag only, not yet pinned: Immich (server, machine-learning and Postgres) and Paperless's Postgres and Redis. Next: add Renovate for update PRs.
- [x] **Tunnel token** moved out of git and rotated (kept in the Portainer-managed stack's environment files on the host).
- [x] **Monitoring.** Uptime Kuma with push alerts is running. Still to do: monitors for Ollama and SSH, an external check that does not depend on the server, and resource monitoring.
- [ ] **Encrypted data drive** (LUKS) with a boot-time unlock strategy.
