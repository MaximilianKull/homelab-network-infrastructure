# Network Diagnostics and Tuning

## TCP congestion control

The VPS was verified using:

```text
net.ipv4.tcp_congestion_control = bbr
net.core.default_qdisc = fq
```

These values were checked with `sysctl` after configuration.

## Latency and packet-loss investigation

Network testing included `ping` and `traceroute` from different points in the path.

A recorded VPS-side test to a public resolver was normally around 25 ms. A DF test using a 1372-byte ICMP payload returned 20/20 packets with no loss and approximately 29 ms average latency during that sample.

These are troubleshooting observations, not advertised benchmarks. Network conditions vary by route, access network, time, and destination.

## MTU / fragmentation test

Linux testing included:

```bash
ping -s 1372 -M do TARGET
```

`-M do` requests that packets not be fragmented. This is useful when investigating whether a path MTU problem may be contributing to connectivity issues.

Payload sizes tested during the investigation included:

- 56 bytes
- 1000 bytes
- 1372 bytes

No clear MTU/fragmentation failure was demonstrated in the recorded testing.

## Client vs server path

The remote client's access network showed substantially higher latency/jitter and intermittent ICMP loss compared with the VPS-side test.

The evidence therefore suggested instability somewhere on the client/access path rather than demonstrating a basic VPS or WireGuard failure. This conclusion is intentionally cautious because the measurements were point-in-time diagnostics rather than controlled long-duration benchmarks.

## IPv6 capability

IPv6 interface and route state were inspected. No usable global IPv6 address/default route was demonstrated in the verified output.

The infrastructure is therefore documented as IPv4-based unless global IPv6 is later configured and independently verified.

## Tools used

- `ping`
- `traceroute`
- `dig`
- `nslookup`
- `ss`
- `sysctl`
- Docker logs and status commands

The broader lesson was to measure each layer independently before changing configuration.