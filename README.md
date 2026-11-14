# lord-of-the-pings

Self-hosted home server infrastructure running on a refurbished HP ProDesk SFF PC (Intel Core i5-9500, 16GB RAM) running Ubuntu Linux, Docker, and containerized microservices.

For a complete visual system topology and data flow map, see the [Architecture Overview](ARCHITECTURE.md).

## Hardware & Architecture

| Component | Specification / Tool | Purpose |
| :--- | :--- | :--- |
| **Host Hardware** | HP ProDesk SFF (Intel Core i5-9500, 16GB DDR4) | Compute node |
| **Operating System** | Ubuntu Linux | Base container host |
| **Container Engine** | Docker & Docker Compose (Managed via Portainer) | Microservice orchestration |
| **Networking** | Tailscale | Secure remote access and mesh VPN |
| **Reverse Proxy** | Caddy (`caddy_proxy`) | Automated TLS termination and internal routing |
| **Web Server** | Nginx (`static_website`) | Static portfolio and website hosting |
| **Ingress** | Cloudflare Tunnels (`cloudflare_tunnel`) | Public service exposure without port forwarding |

---

## Repository Structure

```text
homelab/
├── stacks/
│   ├── anythingllm/       # Local AI knowledge base & RAG workspace
│   ├── caddy/             # Caddy reverse proxy configuration & data volumes
│   ├── immich/            # Self-hosted photo and video backup solution
│   ├── paperless/         # Paperless-ngx document management (PostgreSQL + Redis)
│   ├── stirling-pdf/      # Local PDF utility toolkit
│   └── website-tunnel/    # Nginx static server + Cloudflare Tunnel agent stack
```
## Services & Ports

| Service | Container Name | Internal Port | Access Method | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **Caddy** | `caddy_proxy` | `80`, `443` | Public / Direct | Reverse proxy & automated TLS termination |
| **Nginx** | `static_website` | `80` | Routed via Caddy | Static portfolio & website hosting |
| **Cloudflare Tunnel** | `cloudflare_tunnel` | N/A | Cloudflare Edge | Secure public ingress without open ports |
| **Portainer** | `portainer` | `9443` | Tailscale / Local | Container management UI |
| **Immich** | `immich-server` | `2283` | Tailscale / Local | Self-hosted photo and video backup |
| **Paperless-ngx** | `paperless-webserver` | `8000` | Tailscale / Local | Document management & OCR indexing |
| **Stirling-PDF** | `stirling-pdf` | `8080` | Tailscale / Local | Local PDF utility toolkit |
| **AnythingLLM** | `anythingllm` | `3001` | Tailscale / Local | Local AI knowledge base & RAG workspace |

## Rebuild From Scratch

If you need to redeploy the entire homelab environment on a fresh Ubuntu host (e.g.,lord-of-the-pings), follow these steps:

### 1. Prerequisites & Dependencies
Ensure Docker, Docker Compose, and Git are installed on your Ubuntu host:
```bash
sudo apt update && sudo apt install -y git docker.io docker-compose-v2
sudo usermod -aG docker $USER
