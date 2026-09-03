# WireGuard Configuration Notes

This directory intentionally does **not** contain a copy of the production WireGuard configuration.

A WireGuard configuration can include private keys, preshared keys, peer identities, endpoints, and other infrastructure details that do not belong in a public portfolio repository.

## Verified structure

The deployed server uses:

```text
Interface: wg0
VPN network: 10.66.66.0/24
Server VPN address: 10.66.66.1/24
Transport: UDP
```

The inspected sanitized configuration contained one interface and two peer sections.

## Key handling

Generate cryptographic keys on the systems that will use them. Do not paste real private keys into documentation, examples, tickets, screenshots, or source control.

Conceptually:

```text
server                         client
------                         ------
private key [local only]       private key [local only]
public key  <--------------->  public key
```

The public repository focuses on architecture and operational verification rather than publishing key-shaped placeholder configurations that could encourage accidental credential handling.

For routing behavior and verification, see [`../docs/wireguard-and-routing.md`](../docs/wireguard-and-routing.md).