# AdGuard Home

I run AdGuard Home in Docker Compose on the Ubuntu VPS.

## How I expose it

The main thing I cared about here was not making the DNS service or admin interface more public than necessary.

```text
WireGuard clients
      |
      +--> 10.66.66.1:53 TCP/UDP
                 |
                 v
            AdGuard Home

VPS localhost
      |
      +--> 127.0.0.1:3000
                 |
                 v
          AdGuard web UI :80
```

DNS is reachable from the VPN side. The web UI stays on localhost.

The sanitized Compose file is here:
[`../docker/adguard-home/compose.example.yml`](../docker/adguard-home/compose.example.yml)

## Persistent data

The container uses persistent mounts for its working data and configuration so rebuilding the container does not wipe the setup.

## DNS upstreams

I tested encrypted upstream DNS and ended up using Cloudflare after the Yandex endpoint forms I tried failed validation.

I have not added the exact final upstream string yet because I want to re-check the live config first instead of copying an old or failed value into the repo.

## IPv6

The VPS did not have working global IPv6 in the checks I captured, so I treated DNS behavior with that limitation in mind instead of assuming dual-stack connectivity.

## Port mapping issue I hit

During first setup, AdGuard used its setup interface on one internal port. After configuration, the web service listened internally on port 80.

That meant the Docker mapping had to become:

```text
127.0.0.1:3000 -> container:80
```

The issue was the host-to-container port mapping, not the web service itself.

## Commands I used while checking it

```bash
docker compose ps
docker logs adguardhome --tail 30
ss -lntup | grep ':53 '
```

I also tested DNS from inside the container before moving outward to host and remote-client tests.
