# WireGuard and Routing

WireGuard gives me remote access to the VPS and a private path to the DNS service.

## Addressing

```text
VPN network: 10.66.66.0/24
VPN server:  10.66.66.1
Listen port: 51820/UDP
```

The server config currently has two peer sections. I do not publish real peer keys or endpoint details.

## Traffic flow

```text
remote client
    |
    | WireGuard tunnel
    v
10.66.66.1 (VPS)
    |
    +--> AdGuard DNS on :53
    |
    +--> IPv4 forwarding -> Internet
```

`net.ipv4.ip_forward` is enabled so VPN clients can route through the VPS.

## Firewall

UFW allows the WireGuard UDP port and forwarding from the WireGuard interface toward the external network interface.

A working tunnel and working routed traffic are two different checks. I tested both instead of assuming that a successful handshake meant everything behind the tunnel was reachable.

## Client tests

From a remote macOS client I confirmed:

- the tunnel connects
- the VPN-side DNS service is reachable
- Internet traffic can be routed through the VPS
- DNS works over the tunnel

During the DNS issue I also tested both TCP and UDP separately.

## Config handling

I do not commit the production WireGuard config because it contains key material and peer details. New systems should generate their own keys locally and keep private keys out of source control.

## How I troubleshoot it

If something over WireGuard is broken, I check the path in order:

1. interface up?
2. UDP listener present?
3. VPN-side host reachable?
4. IP forwarding enabled?
5. firewall path allowed?
6. application listening on the right interface?
7. application working locally?

That same approach is what I used in the DNS incident: [`troubleshooting/dns-over-wireguard.md`](troubleshooting/dns-over-wireguard.md)
