# Homelab Network Infrastructure

Hands-on Linux infrastructure project documenting a hardened Ubuntu VPS used for secure remote access and private DNS services.

The goal of this repository is to show the architecture, implementation decisions, verification process, and troubleshooting behind a real self-hosted environment rather than only presenting installation commands.

> **Security:** All configuration examples are sanitized. Private keys, credentials, public IP addresses, personal domains, production hostnames, and provider/account data are intentionally excluded.

## Current verified deployment

- Ubuntu 24.04 LTS VPS
- SSH public-key authentication with password authentication disabled
- UFW host firewall with default-deny inbound policy
- WireGuard remote-access VPN on `10.66.66.0/24`
- IPv4 forwarding for VPN client routing
- Docker + Docker Compose
- Containerized AdGuard Home
- DNS exposed specifically on the WireGuard-side address
- AdGuard management interface bound to localhost
- BBR congestion control with `fq` queueing
- Remote VPN client verified for tunnel connectivity, Internet routing, and DNS resolution

## Architecture

```text
Remote client
     |
     | WireGuard UDP
     v
+--------------------------------------+
| Ubuntu 24.04 VPS                     |
|                                      |
|  SSH administration                  |
|  - public-key authentication         |
|  - password authentication disabled |
|                                      |
|  UFW                                 |
|  - default deny incoming             |
|  - SSH + WireGuard explicitly open  |
|                                      |
|  WireGuard                           |
|  - VPN network: 10.66.66.0/24        |
|  - server: 10.66.66.1                |
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

A broader homelab diagram is available in [`diagrams/`](diagrams/).

## Security model

The deployment follows a small-exposure approach:

1. Administrative SSH access uses public-key authentication; password and keyboard-interactive authentication are disabled.
2. UFW denies unsolicited inbound traffic by default.
3. Only the required SSH and WireGuard entry points are explicitly allowed through the host firewall.
4. AdGuard DNS is published on the WireGuard-side address rather than intentionally operating as a public DNS resolver.
5. The AdGuard management interface is mapped to localhost rather than a public interface.
6. No private key or production credential is stored in this repository.

See [`docs/server-hardening.md`](docs/server-hardening.md) for the verified hardening baseline.

## WireGuard and DNS

The WireGuard server uses the private VPN network `10.66.66.0/24`, with the server observed at `10.66.66.1`. A remote client was tested successfully for:

- tunnel connectivity
- access to the VPN-side DNS service
- Internet routing through the VPN
- DNS resolution over both TCP and UDP during troubleshooting

AdGuard Home runs in Docker. Its DNS listener is published on the VPN-side address on port 53 TCP/UDP, while the management UI is mapped to `127.0.0.1:3000` on the host.

This separation is intentional: VPN clients need DNS, but the administration interface does not need to be publicly exposed.

See [`docs/wireguard-and-routing.md`](docs/wireguard-and-routing.md) and [`docs/adguard-home.md`](docs/adguard-home.md).

## Troubleshooting case study

One of the useful parts of this project was diagnosing a DNS-over-WireGuard failure instead of treating the deployment as a black box.

The investigation separated the path into layers:

```text
remote client
   -> WireGuard tunnel
   -> host listener
   -> Docker port publishing
   -> AdGuard container
   -> upstream DNS
```

Tools used included `dig`, `nslookup`, `ss`, Docker status/log commands, `ping`, and `traceroute`.

Container-local DNS worked while the original remote query timed out. Testing then confirmed Docker's port listener, DNS over TCP, and finally successful remote UDP DNS resolution with `NOERROR`.

A second issue involved the AdGuard web interface: the first-launch interface used its setup port, while the configured web service subsequently listened on container port 80. The host-to-container mapping was corrected accordingly.

See [`docs/troubleshooting/dns-over-wireguard.md`](docs/troubleshooting/dns-over-wireguard.md).

## Network diagnostics and tuning

The VPS was also used for practical network diagnostics:

- BBR + `fq` configured and verified
- latency and packet-loss testing
- multiple ICMP payload sizes
- DF/MTU testing with a 1372-byte payload
- traceroute investigation
- IPv6 capability inspection

No clear MTU/fragmentation failure was demonstrated. VPS-side testing to a public resolver was stable during the recorded test, while the remote access network showed greater latency/jitter and intermittent ICMP loss. This is documented as a troubleshooting observation, not a permanent performance benchmark.

See [`docs/network-diagnostics.md`](docs/network-diagnostics.md).

## Repository structure

```text
.
├── README.md
├── diagrams/
├── docker/
│   └── adguard-home/
│       └── compose.example.yml
├── docs/
│   ├── server-hardening.md
│   ├── wireguard-and-routing.md
│   ├── adguard-home.md
│   ├── network-diagnostics.md
│   ├── verification.md
│   └── troubleshooting/
│       └── dns-over-wireguard.md
└── wireguard/
    └── README.md
```

## Skills demonstrated

- Linux server administration
- SSH hardening and public-key authentication
- Host firewall configuration
- WireGuard VPN deployment
- IPv4 routing and forwarding
- DNS infrastructure
- Docker and Docker Compose
- Service exposure and port-binding design
- Network troubleshooting
- MTU, latency, and packet-loss diagnostics
- Operational verification
- Security-conscious technical documentation

## Scope

This repository intentionally documents conventional defensive infrastructure: Linux administration, secure remote access, DNS, containers, routing, monitoring, and troubleshooting. It does not publish unrelated sensitive infrastructure or credentials.

## Status

This is an evolving lab. Documentation distinguishes verified implementation from future work; planned features are not presented as completed.