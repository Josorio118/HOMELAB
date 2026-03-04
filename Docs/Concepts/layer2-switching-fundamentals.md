# Layer 2 Switching Fundamentals

## What a Switch Actually Does

A switch has one core job: forward Ethernet frames inside a local network based on MAC addresses. It does not understand IP addresses, websites, or applications. All intelligence comes from watching traffic and remembering where it came from.

> Switching is reactive, not predictive. A switch never looks for devices — it only learns from incoming traffic.

---

## Ethernet Frames — The Only Thing a Switch Sees

Every communication on a local network is broken into Ethernet frames. Each frame contains:

- **Source MAC address** — who sent it
- **Destination MAC address** — who should receive it
- **Payload** — the data (the switch ignores this)

The switch reads the source MAC, checks the destination MAC, and makes a forwarding decision.

---

## MAC Address Learning — How the Table is Built

The switch maintains a MAC address table. It is empty when the switch boots and is built entirely from observed traffic.

**Learning process:**
1. A frame enters a switch port
2. Switch reads the source MAC address
3. Switch records: Source MAC + Incoming Port + VLAN + Aging Timer
4. Process repeats for every frame, constantly

**Verification command (HP ProCurve):**
```
show mac-address
```

---

## Forwarding vs Flooding

| Situation | Behavior |
|---|---|
| Destination MAC is **known** | Frame forwarded only out the correct port — efficient |
| Destination MAC is **unknown** | Frame flooded out all ports in the VLAN except the source port |

Unknown unicast flooding is normal and required — it is how the switch discovers where devices live. Once the destination replies, the switch learns its port and flooding stops.

> Networks start noisy, become quiet, and stay efficient as the MAC table fills in.

---

## MAC Address Aging

MAC address table entries are not permanent. Each entry has an aging timer of approximately 300 seconds. If a device stops sending traffic, is unplugged, or moves to another port, the entry is removed.

**Why aging exists:**
- Devices move
- Networks change
- The table must stay accurate

**Real-world implications:**
- Unplugging and replugging a device forces MAC re-learning — explains why this sometimes fixes connectivity issues
- A device moving to a new port causes temporary flooding until the old entry ages out
- "Waiting a few minutes" sometimes resolves issues because stale entries expire

---

## Why Loops Break Switching

Layer 2 frames have no TTL. Without STP, a flooded frame on a looped network circulates forever. The results:

- Broadcast storms
- MAC addresses appearing on multiple ports simultaneously
- Constant MAC table rewrites
- Network collapse

This is why Spanning Tree Protocol exists.

---

## Key Rules

- Switches care about MAC addresses — routers care about IP addresses
- A switch never looks for devices — it only learns from incoming traffic
- Unknown unicast flooding is normal behavior, not a fault
- MAC entries age out (~300 seconds) if traffic stops
- Loops at Layer 2 are fatal without STP
