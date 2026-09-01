# homelab-network-infrastructure
Self-hosted Linux network environment using Docker, Pi-hole and WireGuard
# Self-Hosted Network Infrastructure & Homelab

A self-hosted Linux environment built to gain hands-on experience with Linux system administration, networking, containerization, DNS, secure remote access, monitoring and automation.

The environment runs several network and monitoring services using Docker and is maintained as an ongoing personal infrastructure project.

> **Security note:** All examples and documentation in this repository are sanitized. No production credentials, private keys, public IP addresses, personal domains or other sensitive configuration data are included.

---

## Architecture

The environment consists of a Linux-based host running containerized network and monitoring services.

A visual overview of the infrastructure can be found in the `diagrams` directory.

---

## Technologies

### Operating System & Infrastructure

- Linux
- Docker
- Docker Compose
- TCP/IP
- Network routing
- SSH
- Persistent Docker volumes

### DNS

- Pi-hole
- AdGuard Home
- DNS configuration
- DNS filtering
- DNS troubleshooting
- Upstream DNS resolvers

### Remote Access

- WireGuard
- Encrypted VPN tunnels
- Secure remote access
- Network routing

### Monitoring & Automation

- Linux system monitoring
- Network connectivity monitoring
- DNS health checks
- Service availability checks
- Automated notifications
- Telegram Bot integration

---

## DNS Infrastructure

DNS services are hosted locally to provide centralized DNS resolution and filtering for devices on the network.

Pi-hole and AdGuard Home are used to manage and experiment with:

- Network-wide DNS filtering
- DNS query handling
- Custom DNS configuration
- Upstream DNS resolvers
- DNS troubleshooting
- Service availability

Running DNS services locally has provided practical experience with DNS resolution and troubleshooting network-wide DNS issues.

---

## Containerization

Network services are deployed and managed using Docker.

Containerization keeps individual services isolated while simplifying deployment, maintenance and upgrades.

Areas covered include:

- Container deployment and management
- Docker networking
- Persistent volumes
- Container configuration
- Service updates
- Container troubleshooting
- Docker Compose

Example configurations published in this repository contain placeholder values and do not contain sensitive data from the live environment.

---

## WireGuard Remote Access

WireGuard provides encrypted remote access to resources within the home network.

The implementation provided practical experience with:

- VPN configuration
- Public/private key authentication
- IP addressing
- Network routing
- Peer configuration
- Firewall configuration
- VPN connectivity troubleshooting

Private keys and production configuration files are never stored in this repository.

---

## Monitoring

The environment includes monitoring for both the Linux host and important network services.

Monitored information includes:

- System uptime
- CPU and memory usage
- Disk usage
- System temperature
- Internet connectivity
- DNS availability
- Service status

This makes it possible to identify system, service or connectivity problems without manually checking each component.

---

## Automated Notifications

Monitoring events can trigger automated notifications through a Telegram bot.

This allows important system and network events to be reported automatically.

The public example implementation contains placeholder credentials only. Real bot tokens, chat IDs and other credentials are not included.

---

## Dynamic DNS

Dynamic DNS is used to maintain remote connectivity when the public IP address changes.

Only the general implementation is documented in this repository. The actual hostname and production configuration are intentionally not published.

---

## Security

Security was considered throughout the design and maintenance of the environment.

This public repository therefore contains only sanitized documentation and example configurations.

The following information is intentionally not published:

- Private keys
- Passwords
- API tokens
- Bot tokens
- Public IP addresses
- Personal domains
- Internal hostnames
- Production configuration files
- Logs containing identifying information

---

## Repository Structure

The repository is organized into separate sections for documentation and sanitized example configurations:

- `diagrams/` — Infrastructure and network architecture diagrams
- `docker/` — Sanitized Docker and Docker Compose examples
- `wireguard/` — Sanitized WireGuard configuration examples
- `monitoring/` — Monitoring and notification examples

---

## What I Learned

Building and maintaining this environment has provided hands-on experience beyond theoretical networking concepts.

Key areas include:

- Linux system administration
- Docker and containerization
- Computer networking
- DNS infrastructure
- VPN configuration
- Network routing
- Network troubleshooting
- System and service monitoring
- Automation
- Maintaining self-hosted services

The environment continues to evolve as I experiment with new technologies and improve the reliability and maintainability of the infrastructure.

---

## Disclaimer

This repository documents a personal homelab and learning environment.

Configuration files are provided as sanitized examples for documentation and educational purposes and do not represent the complete production configuration.
