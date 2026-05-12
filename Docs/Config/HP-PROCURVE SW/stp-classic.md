# Spanning Tree Protocol — Classic 802.1D

## Why STP Exists

Redundant physical links at Layer 2 create loops. Switches flood broadcasts and unknown unicasts out all ports. Without a TTL at Layer 2, flooded frames on a looped network circulate forever, causing:

- Broadcast storms
- MAC address table instability — same MAC seen on multiple ports
- High CPU utilization on all switches
- Complete network collapse

> STP does not remove cables. STP does not disable switches. STP selectively blocks specific ports to create a loop-free logical topology.

---

## Root Bridge Election

STP elects one root bridge per VLAN. The root bridge is the reference point for the entire topology — all path calculations are based on distance to root.

**Bridge ID (BID)** = Bridge Priority + MAC Address (tie-breaker)

- Default priority: 32768
- Priority must be configured in increments of 4096
- Lowest Bridge ID wins the election
- If priorities are equal, lowest MAC address wins

> If you don't configure priority, the switch with the lowest MAC becomes root — usually not what you want.

**There is always exactly one root bridge per VLAN.**

---

## STP Port Roles

| Role | Description |
|---|---|
| **Root Port (RP)** | Port with lowest cost path to root. One per non-root switch. Always forwarding. |
| **Designated Port (DP)** | Forwards traffic away from root. One per network segment. Always forwarding. |
| **Blocking Port** | Prevents loops. Does not forward traffic. Still receives BPDUs. |

**Quick rule:** Root bridge has no root ports. Every non-root switch has exactly one root port.

---

## STP Port States

| State | Forwards Traffic | Learns MAC | Receives BPDUs | Notes |
|---|---|---|---|---|
| **Blocking** | ✗ | ✗ | ✓ | Stable — loop prevention |
| **Listening** | ✗ | ✗ | ✓ | Transitional — 15 seconds |
| **Learning** | ✗ | ✓ | ✓ | Transitional — 15 seconds |
| **Forwarding** | ✓ | ✓ | ✓ | Stable — normal operation |
| **Disabled** | ✗ | ✗ | ✗ | Administratively shut down |

---

## STP Timers

| Timer | Value | Purpose |
|---|---|---|
| **Hello** | 2 seconds | How often root bridge sends BPDUs |
| **Forward Delay** | 15 seconds | Time spent in Listening AND Learning states |
| **Max Age** | 20 seconds | How long a port waits after BPDUs stop before assuming topology change |

**Worst case convergence:** Max Age (20) + Listening (15) + Learning (15) = **50 seconds**

> STP timers are set on the root bridge and apply to the entire network. This is another reason controlling root bridge placement matters.

---

## Path Cost — How STP Chooses Paths

STP selects the lowest-cost path to the root bridge. Cost is based on link speed — faster links have lower cost.

| Link Speed | Classic STP Cost |
|---|---|
| 10 Mbps | 100 |
| 100 Mbps | 19 |
| 1 Gbps | 4 |
| 10 Gbps | 2 |

> STP does not care about hop count. It cares about path cost.

---

## STP Tie-Breaker Order

When path costs are equal, STP breaks ties in this order:
1. Lowest root path cost
2. Lowest neighbor bridge ID
3. Lowest neighbor port ID

---

## PVST+ (Cisco)

Cisco's Per-VLAN Spanning Tree Plus runs a separate STP instance per VLAN. This allows:

- Different root bridges per VLAN
- Different blocked ports per VLAN
- Load balancing across redundant links

BPDU MAC address for PVST+: `0100.0ccc.cccd`
BPDU MAC address for IEEE standard STP: `0180.c200.0000`

---

## This Lab — HP ProCurve 2524

STP was disabled by default on this switch. When enabled, the ISP router initially won the root election because it was sending BPDUs with a lower bridge ID.

**Fix:** Set switch priority to 4096 — lowest in the topology — to take root.

```
spanning-tree
spanning-tree priority 4096
write memory
```

**Result:** Switch confirmed as root, timers normalized to 802.1D defaults (Hello 2, Max Age 20, Forward Delay 15).

---

## Key Rules

- STP prevents Layer 2 loops by selectively blocking ports
- One root bridge per VLAN — lowest Bridge ID wins
- Root ports → lowest cost path to root (one per non-root switch)
- Designated ports → forward traffic away from root (one per segment)
- Blocking ports → prevent loops, still receive BPDUs
- Classic STP worst case convergence = 50 seconds
- STP timers on root bridge determine timers for the entire network
