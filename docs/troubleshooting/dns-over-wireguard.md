# Incident: DNS Resolution over WireGuard

## Symptom

A remote client initially timed out when querying the VPN-side DNS service.

Sanitized test form:

```bash
dig @VPN_DNS google.com
```

The goal was to determine whether the problem was the VPN, the host listener, Docker publishing, AdGuard itself, or upstream DNS.

## Diagnostic strategy

Instead of changing multiple components at once, the path was tested layer by layer:

```text
remote client
    |
WireGuard tunnel
    |
host :53 listener
    |
Docker port publishing
    |
AdGuard container
    |
upstream DNS
```

## 1. Inspect listeners

```bash
ss -lntup | grep ':53 '
```

This checks which process is listening, on which address, and for which transport.

## 2. Check container state

```bash
docker compose ps
docker logs adguardhome --tail 30
```

This distinguishes a dead/unhealthy container from a network publication problem.

## 3. Test DNS inside the container

```bash
docker exec adguardhome nslookup google.com 127.0.0.1
```

Container-local DNS succeeded. That was important evidence: the DNS application itself could resolve queries, so the investigation could move outward toward Docker/host/client connectivity.

## 4. Test host-side behavior

A host-local `dig` test was used as another boundary check. The final deployment intentionally did not publish AdGuard DNS on host localhost; it was bound to the VPN-side address instead.

This matters because `localhost:53` and `VPN_ADDRESS:53` are different sockets and should not be treated as interchangeable tests.

## 5. Verify Docker publishing

Docker's proxy was confirmed listening on the VPN-side address on port 53 for both TCP and UDP.

That demonstrated that the container port was being published where the VPN design expected it.

## 6. Compare TCP and UDP DNS

A remote TCP query was tested:

```bash
dig @VPN_DNS google.com +tcp
```

DNS over VPN TCP worked. Remote UDP DNS was then retested and ultimately returned a successful `NOERROR` response.

## Result

The final verified state was:

- AdGuard resolved DNS from inside its container
- Docker published port 53 TCP/UDP on the VPN-side host address
- DNS over WireGuard worked with TCP
- DNS over WireGuard worked with UDP
- remote client resolution succeeded

## What this incident demonstrated

The useful part of the incident was not a single command. It was narrowing the fault domain without assuming that "DNS is broken" identified the failing component.

The troubleshooting model separated:

1. client/access network
2. VPN tunnel
3. Linux host
4. Docker publishing
5. application/container
6. upstream Internet service

That same approach is reusable for other containerized network services.