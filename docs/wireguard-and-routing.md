# WireGuard and Routing

## Purpose

WireGuard provides legitimate encrypted remote access to the VPS environment and a private path to the DNS service.

## Verified addressing

```text
VPN network: 10.66.66.0/24
VPN server:  10.66.66.1
Listen port: 51820/UDP
```

Two peer sections were present in the sanitized server configuration at the time of inspection. Their keys and identifying endpoint information are intentionally not reproduced.

## Traffic flow

```text
remote client
    |
    | encrypted WireGuard tunnel
    v
10.66.66.1 (VPS)
    |
    +--> AdGuard DNS on VPN-side :53
    |
    +--> IPv4 forwarding -> Internet
```

IPv4 forwarding was verified as enabled on the server.

## Firewall relationship

The host firewall explicitly allows the WireGuard UDP entry point. Routed traffic from the WireGuard interface toward the external VPS interface was also present in the verified firewall state.

This is a useful distinction: establishing the encrypted tunnel and successfully routing client traffic are separate layers that both need verification.

## Client verification

A remote macOS client was used to verify:

- WireGuard connectivity
- reachability of the VPN-side DNS service
- Internet routing through the tunnel
- DNS resolution over the tunnel

DNS troubleshooting additionally verified both TCP and UDP behavior.

## Configuration safety

A production WireGuard configuration is deliberately not committed because it contains cryptographic identity material.

The public repository documents structure and behavior rather than distributing real peer configurations. When reproducing the environment, generate new keys on the systems that will use them and keep private material outside source control.

## Troubleshooting model

When a client cannot use a service over WireGuard, test the layers separately:

1. Is the WireGuard interface active?
2. Is the expected UDP listener present?
3. Can the client reach the VPN-side host address?
4. Is IP forwarding configured when routing is required?
5. Does the firewall permit the intended path?
6. Is the application listening on the correct host/interface?
7. Does the application work locally before testing remotely?

This layer-by-layer method was used during the DNS incident documented in [`troubleshooting/dns-over-wireguard.md`](troubleshooting/dns-over-wireguard.md).