# Architecture Overview

## System Topology & Data Flow

```mermaid
graph TD
    Internet((Internet)) -->|Cloudflare Tunnel| WT[website-tunnel / Nginx]
    TailscaleUser((Tailscale / LAN)) -->|Secure Access| Host[lord-of-the-pings Ubuntu Host]

    subgraph Host [Ubuntu Host: ~/homelab/stacks/]
        direction TB
        WT --> LocalWeb[Static Portfolio]
        
        subgraph Stacks [Modular Infrastructure Stacks]
            Immich[immich + postgres]
            Paperless[paperless-ngx + redis]
            PDF[stirling-pdf]
        end
    end

    subgraph Storage [Hardware Layers]
        NVMe[Host NVMe SSD <br/> OS, Configs, Git Repos]
        ExtHDD[(500GB External HDD <br/> /run/media/frodo/New Volume1/immich)]
    end

    Host -.-> NVMe
    Immich -->|UPLOAD_LOCATION| ExtHDD
