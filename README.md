# Homelab Network Infrastructure

This repository documents a Linux networking lab I built around an Ubuntu 24.04 VPS.

The current setup uses WireGuard for remote access and runs AdGuard Home in Docker for DNS filtering. Most of the interesting work was not the installation itself, but getting routing, service exposure, firewall rules and DNS through Docker/WireGuard working cleanly.

> Public examples are sanitized. No real private keys, credentials, public IP addresses, personal domains, production hostnames or provider data are stored here.

## Current setup

- Ubuntu 24.04 LTS VPS
- SSH with public-key authentication
- password and keyboard-interactive SSH authentication disabled
- UFW with default-deny incoming policy
- WireGuard on `10.66.66.0/24`
- IPv4 forwarding for VPN traffic
- Docker + Docker Compose
- AdGuard Home in Docker
- DNS published on the WireGuard-side address only
- AdGuard web UI bound to localhost
- BBR with `fq`

## Architecture

```text
Remote client
     |
     | WireGuard UDP
     v
+--------------------------------------+
| Ubuntu 24.04 VPS                     |
|                                      |
|  SSH                                 |
|  UFW                                 |
|  WireGuard 10.66.66.1/24             |
|       |                              |
|       +----> Docker                  |
|              |                       |
|              +--> AdGuard Home       |
|                   DNS :53 TCP/UDP    |
|                                      |
|  AdGuard UI -> localhost only        |
+------------------+-------------------+
                   |
                   v
              Upstream DNS
```

A broader homelab diagram is in [`diagrams/`](diagrams/).

## Hardening choices

I kept externally reachable services to the minimum I needed for this setup: SSH and WireGuard.

For SSH, root login is disabled and authentication is done with public keys instead of passwords. UFW denies unsolicited inbound traffic by default and only allows the services I need.

For AdGuard, I did not want to expose either a public DNS resolver or the management UI. DNS is bound to the WireGuard address, while the web UI is mapped to `127.0.0.1:3000`.

More details: [`docs/server-hardening.md`](docs/server-hardening.md)

## WireGuard and DNS

The VPN network is `10.66.66.0/24`, with the server on `10.66.66.1`.

I tested the setup from a remote macOS client and confirmed:

- the WireGuard tunnel comes up
- the client can use the VPN-side DNS service
- Internet traffic can be routed through the VPS
- DNS works over the tunnel

AdGuard Home runs in Docker and listens on port 53 TCP/UDP through the WireGuard-side host address. The web interface stays local to the VPS.

Details: [`docs/wireguard-and-routing.md`](docs/wireguard-and-routing.md) and [`docs/adguard-home.md`](docs/adguard-home.md)

## Something that broke: DNS over WireGuard

At one point the remote client timed out on DNS even though the tunnel itself was up.

Instead of changing everything at once, I worked through the path one layer at a time:

```text
client
  -> WireGuard
  -> Linux host listener
  -> Docker port publishing
  -> AdGuard container
  -> upstream DNS
```

I used `dig`, `nslookup`, `ss`, `docker compose ps`, Docker logs, `ping` and `traceroute` while narrowing it down.

The important clue was that DNS worked inside the container. From there I checked the host listener and Docker publishing, tested DNS over TCP, and then retested remote UDP DNS until it returned `NOERROR`.

I also hit a separate AdGuard web UI issue. The setup interface and the final web interface used different container ports, so I had to correct the Docker host-to-container mapping.

Full write-up: [`docs/troubleshooting/dns-over-wireguard.md`](docs/troubleshooting/dns-over-wireguard.md)

## Network diagnostics

I also used the VPS to test a few lower-level network issues:

- BBR + `fq`
- latency and packet loss
- different ICMP payload sizes
- DF/MTU testing with a 1372-byte payload
- traceroute
- IPv6 capability

I did not find a clear MTU/fragmentation failure in the recorded tests. The VPS-side path was comparatively stable, while the remote access network showed more latency/jitter and intermittent ICMP loss.

More details: [`docs/network-diagnostics.md`](docs/network-diagnostics.md)

## Files worth looking at

- [`docker/adguard-home/compose.example.yml`](docker/adguard-home/compose.example.yml) — sanitized Compose example based on the live deployment
- [`docs/server-hardening.md`](docs/server-hardening.md) — SSH, UFW and service exposure
- [`docs/troubleshooting/dns-over-wireguard.md`](docs/troubleshooting/dns-over-wireguard.md) — real DNS troubleshooting case
- [`docs/network-diagnostics.md`](docs/network-diagnostics.md) — BBR, MTU, latency and IPv6 checks
- [`docs/verification.md`](docs/verification.md) — what is working and what is still open

## Things I would improve next

This setup works, but there are still things I want to tighten up:

- disable X11 forwarding because I do not need it on this VPS
- review whether the current routed-firewall policy can be made more restrictive
- pin the AdGuard container image instead of relying on `latest`
- document the final encrypted DNS upstream once I re-check the live config
- revisit global IPv6 only if the VPS provider actually supplies usable IPv6 connectivity

I prefer leaving those visible instead of presenting the lab as finished or perfect.
