# lord-of-the-pings

Self-hosted home server infrastructure running on a refurbished HP ProDesk SFF PC (Intel Core i5-9500, 16GB RAM) running Ubuntu Linux, Docker, and containerized microservices.

## Hardware & Architecture

| Component | Specification / Tool | Purpose |
| :--- | :--- | :--- |
| **Host Hardware** | HP ProDesk SFF (Intel Core i5-9500, 16GB DDR4) | Compute node |
| **Operating System** | Ubuntu Linux | Base container host |
| **Container Engine** | Docker & Docker Compose (Managed via Portainer) | Microservice orchestration |
| **Networking** | Tailscale | Secure remote access and mesh VPN |
| **Ingress** | Cloudflare Tunnels | Public service exposure without port forwarding |

---

## Repository Structure

```text
homelab/
├── stacks/
│   ├── anythingllm/       # Local AI knowledge base & RAG workspace
│   ├── immich/            # Self-hosted photo and video backup solution
│   ├── paperless/         # Paperless-ngx document management (PostgreSQL + Redis)
│   ├── stirling-pdf/      # Local PDF utility toolkit
│   └── website-tunnel/    # Nginx static portfolio server + Cloudflare Tunnel agent
