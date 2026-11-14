graph TD
    subgraph Internet
        User[External User / Browser]
        CF_Edge[Cloudflare Edge]
    end

    subgraph Tailscale Network
        TS_Client[Tailscale Remote Client]
    end

    User -->|HTTPS / Domain| CF_Edge
    CF_Edge -->|Cloudflare Tunnel| Tunnel[cloudflare_tunnel container]
    TS_Client -->|Secure Mesh VPN| Host[Ubuntu Host: lord-of-the-pings]

    subgraph Host ["Ubuntu Host (HP ProDesk SFF - i5-9500)"]
        subgraph Docker ["Docker Engine & web_net Bridge Network"]
            Tunnel -->|HTTP / Internal| Caddy[caddy_proxy / Caddy]
            Caddy -->|TLS Termination & Routing| Nginx[static_website / Nginx]
            
            subgraph Services ["Containerized Services"]
                Nginx
                Immich[Immich Photo Stack]
                Paperless[Paperless-ngx Stack]
                Stirling[Stirling-PDF]
                AnythingLLM[AnythingLLM Workspace]
                Portainer[Portainer CE]
            end
        end
    end
