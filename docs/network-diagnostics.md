# Network Diagnostics and Tuning

## BBR and queueing

The VPS is using:

```text
net.ipv4.tcp_congestion_control = bbr
net.core.default_qdisc = fq
```

I checked both values with `sysctl` after configuring them.

## Latency and packet loss

I used `ping` and `traceroute` from different points in the path to separate VPS-side behavior from the remote client's access network.

One VPS-side sample to a public resolver was around 25 ms. A 1372-byte DF test returned 20/20 packets with no loss and about 29 ms average latency for that run.

I treat those numbers as troubleshooting samples, not performance benchmarks.

## MTU test

```bash
ping -s 1372 -M do TARGET
```

I also tested smaller payloads (56 and 1000 bytes). I did not reproduce a clear MTU/fragmentation failure.

The remote client path showed more latency/jitter and intermittent ICMP loss than the VPS-side path, which made the access network a more likely source of the instability I was seeing at that time.

## IPv6

I checked the IPv6 addresses and routes on the VPS. There was no usable global IPv6 address or default IPv6 route in the captured output, so I currently treat the deployment as IPv4-only.

## Tools used

- `ping`
- `traceroute`
- `dig`
- `nslookup`
- `ss`
- `sysctl`
- Docker status/log commands

The main habit I took from this troubleshooting was to test each layer separately before changing configuration.
