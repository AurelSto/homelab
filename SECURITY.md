# Security Policy

Security architecture, known limitations and reporting process for this self-hosted deployment. Last verified against the running host: 2026-11-20. See [ARCHITECTURE.md](ARCHITECTURE.md) for the full topology and verification commands.

## Scope and threat model

A single personal server on a home network. The only intentionally public service is a static portfolio site. Everything else is for private use and should be reachable only from devices on my Tailscale network. Protection against a compromised tailnet device or a compromised host is out of scope.

## Network isolation

- **No inbound ports.** The router has no port forwards. The public site is reached through an outbound Cloudflare Tunnel (`cloudflared` → Caddy → Nginx) on a private Docker network; Caddy and Nginx publish no host ports.
- **Tailscale-bound services.** The published ports of Immich (`2283`), Paperless-ngx (`8010`), Stirling-PDF (`8080`), AnythingLLM (`3001`), Uptime Kuma (`3002`) and Portainer (`9443`) are bound to the host's Tailscale address, not to `0.0.0.0`. They are not reachable from the LAN or the internet.
- **Host firewall (UFW).** Default deny for incoming traffic; allow on the `tailscale0` interface. Ollama (`11434`) is also allowed from the AnythingLLM Docker subnet only.
- **Docker bypasses UFW** for published container ports, so those ports are protected by their bind address in the compose files, not by firewall rules.
- **Native services on all interfaces.** Ollama (`11434`) and the remote desktop service (`3389`) listen on all interfaces and rely on UFW; RDP is allowed only on `tailscale0`.

## Host hardening

- **SSH:** key-only authentication, no root login, `AllowUsers` restricted to one account, `MaxAuthTries 3`. Port 22 has no firewall rule of its own and is reachable only over Tailscale.
- **Updates:** `unattended-upgrades` installs security updates automatically (Ubuntu security and ESM security pockets only). Other package updates are applied manually. Kernel and some library updates take effect only after a reboot.
- **Drive mount safety:** the data drive is mounted via `/etc/fstab` with `nofail`, and its mount point is immutable while unmounted (`chattr +i`), so a missing drive makes containers fail instead of silently writing to the system drive.

## Containers and storage

- **Unprivileged users where configured.** Paperless-ngx runs its application processes as UID/GID 1000 (`USERMAP_UID`/`USERMAP_GID`). Stirling-PDF's image uses UID/GID 1000 by default. Data directories on the data drive are owned by `1000:1000`.
- **Immich startup checks.** Immich verifies marker files (`.immich`) in its upload folders on startup and refuses to start if they are missing or unreadable. This catches an unmounted or replaced storage volume; it is not an integrity check of the stored files.
- **Secrets** are kept in git-ignored `.env` files; each stack documents its variables in `.env.example`. The Cloudflare tunnel token for the Portainer-managed `website-tunnel` stack lives in the environment files next to that stack's compose file on the host.

## Known limitations

- **Portainer** has access to the Docker socket, which is equivalent to root on the host. It is reachable only over Tailscale.
- **AnythingLLM** runs with the `SYS_ADMIN` capability.
- **Stirling-PDF** has its built-in login disabled (`DOCKER_ENABLE_SECURITY=false`); the tailnet is its only access control.
- **Image tags.** Pinned by digest in this repository: Paperless-ngx, Stirling-PDF, Uptime Kuma, Valkey, and the `website-tunnel` images (nginx, cloudflared, Caddy). Still floating: AnythingLLM (`latest`) and Portainer (`latest`); Immich follows its own release tag. The live `website-tunnel` stack in Portainer currently runs floating tags (`nginx:alpine`, `caddy:latest`, `cloudflared:latest`).
- **No Cloudflare Access** policy is configured; the only public service is the static site.
- **Alerts** are delivered through the public `ntfy.sh` service. The topic name is the only secret, so alerts contain service names only.
- **No backups yet.** Data lives on single disks, and Immich's database and the Docker volumes sit on the system drive; see the roadmap in [README.md](README.md).

## Incident history

- **2026-11:** A Cloudflare tunnel token was committed to this repository by mistake (in an example env file) and stayed public for several days. It was rotated in Cloudflare on 2026-11-19, replaced in the running stack, and scrubbed from the Git history. Earlier commits of the Paperless compose file also contained credentials; the database password and secret key were regenerated.

## Reporting a vulnerability

This is a personal infrastructure repository. If you find a security issue or misconfiguration, please use GitHub's private vulnerability reporting (Security tab → Report a vulnerability) instead of a public issue.
