# systemd overrides

Host-level systemd configuration for `lord-of-the-pings`. These files are copies; the live versions are in `/etc/systemd/system/`.

## docker.service.d/tailscale-wait.conf

Makes `docker.service` wait (up to 30 seconds) for the Tailscale IP `100.90.170.107` to appear on `tailscale0` before Docker starts. Without it, containers with ports published on that IP can fail to bind at boot (`cannot assign requested address`).

The leading `-` on `ExecStartPre` makes the wait non-fatal, so Docker still starts if Tailscale never comes up.

### Install

    sudo mkdir -p /etc/systemd/system/docker.service.d
    sudo cp systemd/docker.service.d/tailscale-wait.conf /etc/systemd/system/docker.service.d/
    sudo systemctl daemon-reload

### Verify

    systemctl show docker.service -p ExecStartPre --no-pager | grep -o 'ignore_errors=[a-z]*'

Expected output: `ignore_errors=yes`.

If you edit the file in `/etc`, copy it back into the repo and commit it.
