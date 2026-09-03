# Server Hardening

This is the current hardening baseline on the Ubuntu 24.04 VPS. Real keys, credentials, hostnames and public addresses are not included here.

## SSH

I checked the effective OpenSSH settings with `sshd -T` instead of only reading the config file.

```text
port 22
permitrootlogin no
pubkeyauthentication yes
passwordauthentication no
kbdinteractiveauthentication no
```

So root login is disabled and I use SSH public-key authentication instead of passwords.

When changing SSH authentication, I keep an existing session open and test the new key-based login in a second session before disabling password access. That avoids locking myself out if something is wrong.

No private key or `authorized_keys` file is stored in this repo.

## UFW

Current policy:

```text
Default incoming: deny
Default outgoing: allow
Default routed:   allow
```

Inbound rules are limited to the services I currently need:

```text
SSH       TCP
WireGuard UDP
```

The firewall also allows forwarding from the WireGuard interface toward the VPS network interface so VPN clients can route through the host.

One thing I still want to review is whether the routed default policy can be tightened without breaking the VPN path.

## Service exposure

I tried to avoid exposing application interfaces directly to the Internet.

- SSH is reachable for administration.
- WireGuard is reachable for VPN clients.
- AdGuard DNS listens on the WireGuard-side address.
- The AdGuard web UI is mapped only to loopback:

```text
127.0.0.1:3000 -> container:80
```

That lets me administer AdGuard locally on the VPS without making its web interface public.

## IP forwarding

The VPN needs IPv4 forwarding:

```text
net.ipv4.ip_forward = 1
```

## IPv6

The VPS did not show a usable global IPv6 address or default IPv6 route during the checks I captured, so I currently treat this setup as IPv4-only.

## Remaining hardening work

`sshd -T` also showed:

```text
x11forwarding yes
```

I do not use X11 forwarding on this server, so disabling it is still on my cleanup list.

## What never goes into the repo

- SSH private keys
- WireGuard private or preshared keys
- passwords
- API/bot tokens
- real public IP addresses
- personal domains or hostnames
- provider/account information
