# DNS over WireGuard: Troubleshooting Notes

## Problem

A remote client could connect to WireGuard, but DNS queries to the VPN-side DNS service timed out.

```bash
dig @VPN_DNS google.com
```

I wanted to figure out whether the problem was the tunnel, the Linux host, Docker, AdGuard or the upstream resolver.

## What I checked

```text
client
  -> WireGuard
  -> host :53 listener
  -> Docker port publishing
  -> AdGuard container
  -> upstream DNS
```

### 1. Is anything listening on port 53?

```bash
ss -lntup | grep ':53 '
```

### 2. Is the container actually running?

```bash
docker compose ps
docker logs adguardhome --tail 30
```

### 3. Does DNS work inside the container?

```bash
docker exec adguardhome nslookup google.com 127.0.0.1
```

It did. That ruled out a basic AdGuard/upstream failure and pushed the problem further out toward Docker publishing or the client path.

### 4. Is Docker publishing the right socket?

I confirmed Docker was listening on the WireGuard-side host address on port 53 for both TCP and UDP.

The deployment does **not** publish AdGuard DNS on host localhost, so `127.0.0.1:53` and `VPN_ADDRESS:53` are not equivalent tests.

### 5. Does TCP DNS work?

```bash
dig @VPN_DNS google.com +tcp
```

TCP worked over the VPN. I then retested normal UDP DNS remotely, which returned `NOERROR`.

## Result

At the end of the checks:

- DNS worked inside the AdGuard container
- Docker published port 53 TCP/UDP on the VPN address
- DNS over WireGuard worked over TCP
- DNS over WireGuard worked over UDP
- the remote client resolved DNS successfully

## Takeaway

The useful part for me was narrowing the failure down one layer at a time instead of treating "DNS is broken" as one problem. The same pattern applies to most containerized network services: prove the application locally first, then move outward through container networking, host networking, firewall/routing and finally the client.
