# Security Policy

This document outlines the security architecture and hardening measures implemented for this self-hosted deployment.

## Network Isolation

All public and unsecured local network exposure is disabled for core services. 

* **Tailscale VPN Binding:** Services do not listen on public interfaces (`0.0.0.0`) or unauthenticated local network adapters. Ingress is restricted exclusively to the private Tailscale virtual private network interface (`100.90.170.107`).
* **Configured Endpoints:**
  * **Immich:** Bound exclusively to `100.90.170.107:2283`
  * **Paperless-ngx:** Bound exclusively to `100.90.170.107:8010`

## Container Permissions & Storage Security

* **User Mapping:** Containers enforce non-root execution contexts where applicable (utilizing explicit `USERMAP_UID: 1000` and `USERMAP_GID: 1000` configurations).
* **Storage Integrity:** Persistent storage volumes (such as `/mnt/immich-storage`) enforce strict directory ownership (`100.90.170.107` host-level UID/GID mapping) and utilize system sentinel validation checks (`.immich` files) to verify read/write integrity on startup.

## Reporting a Vulnerability

Since this is a self-hosted personal infrastructure stack, please report any security concerns or misconfigurations directly via private communication rather than public issue trackers.
