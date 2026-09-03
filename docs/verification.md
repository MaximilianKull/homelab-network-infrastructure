# Verification Checklist

This checklist records items supported by the captured deployment evidence.

## Host

- [x] Ubuntu 24.04 LTS verified
- [x] UFW active
- [x] default inbound firewall policy is deny
- [x] SSH entry point explicitly allowed
- [x] WireGuard UDP entry point explicitly allowed
- [x] IPv4 forwarding enabled
- [x] BBR congestion control enabled
- [x] `fq` default queueing discipline enabled
- [x] no functional global IPv6 assumed without evidence

## SSH

- [x] remote root login disabled
- [x] public-key authentication enabled
- [x] password authentication disabled
- [x] keyboard-interactive authentication disabled
- [ ] X11 forwarding disabled — not currently verified; effective configuration reported it enabled

## WireGuard

- [x] `wg0` interface present
- [x] UDP listener present
- [x] VPN server address `10.66.66.1/24` observed
- [x] remote client connectivity verified
- [x] Internet routing through VPN verified

## Docker / AdGuard Home

- [x] Docker installed and working
- [x] Docker Compose installed and working
- [x] AdGuard Home container running
- [x] persistent volumes configured
- [x] DNS published on VPN-side port 53 TCP
- [x] DNS published on VPN-side port 53 UDP
- [x] management UI mapped to localhost

## DNS

- [x] container-local DNS resolution verified
- [x] DNS over WireGuard TCP verified
- [x] DNS over WireGuard UDP verified
- [x] successful remote response observed
- [ ] exact final encrypted upstream string documented — intentionally pending because the captured evidence is incomplete

## Network diagnostics

- [x] latency tested
- [x] packet loss tested
- [x] multiple ICMP payload sizes tested
- [x] DF/MTU test performed
- [x] traceroute used during investigation
- [x] no clear MTU/fragmentation failure demonstrated

Unchecked items are intentionally not presented elsewhere as completed work.