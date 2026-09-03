# AdGuard Home

## Deployment

AdGuard Home is deployed with Docker Compose on the Ubuntu VPS.

Verified runtime versions during inspection included Docker 29.1.3 and Docker Compose 2.40.3. AdGuard Home had previously been observed running version 0.107.79; the public Compose example intentionally avoids pretending that `latest` is a fixed version.

Persistent data is stored under the service directory and mounted into the container for AdGuard work and configuration data.

## Exposure model

The important design decision is where the service is published.

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

DNS is therefore reachable through the VPN-side address, while the management UI is bound locally.

## Sanitized Compose example

See [`../docker/adguard-home/compose.example.yml`](../docker/adguard-home/compose.example.yml).

The example uses placeholders and does not contain credentials or public infrastructure identifiers.

## DNS upstreams

Encrypted upstream DNS was explored and configured. Cloudflare was selected after tested Yandex endpoint forms failed validation.

The exact final production upstream string was not captured in the verified handoff, so it is intentionally not claimed or reproduced here. This prevents a suggested or failed value from being presented as a known-good production configuration.

## IPv6 behavior

The VPS did not demonstrate usable global IPv6 connectivity during inspection. IPv6-related DNS behavior was therefore reviewed with that host limitation in mind.

## Web UI port troubleshooting

During first launch, AdGuard exposed its setup interface on its initial setup port. After configuration, the web service listened internally on port 80.

The Docker mapping therefore needed to represent the final application listener:

```text
host 127.0.0.1:3000 -> container 80/tcp
```

This was a host-versus-container port mapping issue, not a failure of the web application itself.

## Operational checks

Useful commands used during the deployment included:

```bash
docker compose ps
docker logs adguardhome --tail 30
ss -lntup | grep ':53 '
```

Application-local DNS was also tested from inside the container before remote client testing.