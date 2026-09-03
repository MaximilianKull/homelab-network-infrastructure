# Server Hardening Baseline

This document records the hardening controls that were verified on the Ubuntu 24.04 VPS. It deliberately avoids publishing production credentials, hostnames, public addresses, or key material.

## SSH

The effective OpenSSH server configuration was inspected with `sshd -T` rather than relying only on comments in configuration files.

Verified settings:

```text
port 22
permitrootlogin no
pubkeyauthentication yes
passwordauthentication no
kbdinteractiveauthentication no
```

This means remote root login is disabled and interactive SSH authentication is based on public-key authentication rather than account passwords.

### Safe key workflow

Private keys are never copied into this repository. The operational model is:

```text
administrator device
  private key -> remains private
  public key  -> installed for the server account
```

A safe deployment workflow is to establish and test public-key access in a separate session before removing password-based access. This avoids turning an authentication hardening change into an administrative lockout.

This repository does not contain an `authorized_keys` file or any real SSH key material.

## Host firewall

UFW was verified as active with logging enabled at the low level.

Observed policy:

```text
Default incoming: deny
Default outgoing: allow
Default routed:   allow
```

The required inbound services were explicitly allowed:

```text
SSH       TCP
WireGuard UDP
```

Forwarding from the WireGuard interface toward the VPS network interface was also present for VPN client routing.

Exact production addresses are intentionally omitted.

## Service exposure

The objective was not simply to make services reachable, but to make them reachable only where required.

### Public-facing entry points

The host requires an SSH entry point for administration and a WireGuard UDP entry point for VPN clients.

### VPN-side service

AdGuard Home DNS is published on the WireGuard-side host address on port 53 TCP and UDP. It is intended for VPN clients, not as an intentionally public open resolver.

### Local-only administration

The AdGuard management interface is mapped to loopback on the VPS:

```text
127.0.0.1:3000 -> container:80
```

Binding the administration interface to localhost reduces unnecessary network exposure.

## Routing

IPv4 forwarding was verified:

```text
net.ipv4.ip_forward = 1
```

This is required for the VPS to route traffic for the remote-access VPN design.

## IPv6

No usable global IPv6 configuration was demonstrated during verification. The host should therefore not be described as providing working global IPv6 connectivity until that is separately configured and tested.

## Additional observations

The effective SSH configuration also reported X11 forwarding enabled. It was not required for the documented infrastructure. This repository records the verified state rather than pretending every possible hardening control was already applied.

That distinction is intentional: a security portfolio should document what was actually implemented and identify remaining review items instead of claiming an idealized configuration.

## Secrets policy

Never commit:

- SSH private keys
- WireGuard private or preshared keys
- passwords
- API/bot tokens
- production public IP addresses
- personal domains or hostnames
- provider/account information

Example configuration uses descriptive placeholders instead.