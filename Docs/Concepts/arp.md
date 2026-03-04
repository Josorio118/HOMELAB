# ARP — Address Resolution Protocol

## The Problem ARP Solves

Ethernet requires a destination MAC address to deliver a frame. But applications only know IP addresses. ARP exists solely to bridge this gap.

- Applications use IP
- Ethernet uses MAC
- Switches only understand MAC
- Routers understand IP but still send Ethernet frames locally

> ARP's only job is to find the MAC address of the next-hop IP.

---

## What ARP Actually Is

ARP is a discovery protocol that operates exclusively inside a broadcast domain. It is:

- Completely trust-based — no authentication
- Required before any meaningful IP communication can occur
- Local only — it does not route, does not leave the network, and does not care about ports or switches

---

## ARP Request — Why It Must Be a Broadcast

When a device wants to reach an IP it doesn't have a MAC for, it sends an ARP Request:

> "Who has IP X.X.X.X? Tell me your MAC address."

- The sender does not know the destination MAC, so it cannot unicast
- Destination MAC = `FF:FF:FF:FF:FF:FF` (broadcast)
- Switch floods it to all ports in the VLAN
- Only the device with the matching IP replies

This is not inefficient — it is logically required.

---

## ARP Reply — Why It Is Unicast

Once the destination receives the ARP Request, it already knows the requester's MAC (it was in the frame). It replies directly — no broadcast, no flooding.

ARP is noisy once, then silent. This mirrors switch MAC learning: flood to discover, learn, become efficient.

---

## ARP Cache

Every device maintains an ARP cache — a local table mapping IP addresses to MAC addresses.

- Entries expire over time
- Expiration causes new ARP broadcasts
- Clearing the cache causes temporary flooding

**This explains:**
- "It worked after waiting" — stale ARP entry expired and resolved correctly
- "First ping fails, second works" — first ping triggers ARP, second ping uses cached entry
- "It broke, then fixed itself" — ARP cache updated after topology change

---

## Local Traffic vs Internet Traffic

| Destination | Behavior |
|---|---|
| Same subnet (e.g. 192.168.10.50) | PC checks subnet → local → ARPs for that device directly |
| Different subnet / internet (e.g. 8.8.8.8) | PC checks subnet → not local → ARPs for the default gateway instead |

> Your computer never ARPs for Google. It ARPs for the router that will forward the traffic on its behalf.

---

## Why Routers Kill Broadcasts

Routers do not forward Layer 2 broadcasts. This means:

- ARP stays local to its broadcast domain
- ARP storms cannot collapse the internet
- VLANs and routers together create broadcast boundaries

Without this behavior, a single ARP broadcast would propagate across every network on the internet.

---

## End-to-End Flow — Typing a Website

1. DNS resolves the hostname to an IP address
2. PC checks whether the destination is local or remote
3. Destination is not local
4. PC ARPs for the default gateway (router)
5. Switch floods the ARP request
6. Router replies with its MAC
7. PC caches the router MAC
8. Ethernet frames are sent to the router
9. Router routes the packet onward toward the destination
