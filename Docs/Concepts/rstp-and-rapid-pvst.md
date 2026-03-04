# RSTP, Rapid PVST+ & STP Guard Features

## Why RSTP Exists

Classic 802.1D is timer-based. Worst case convergence is 50 seconds — unacceptable in modern networks. RSTP (IEEE 802.1w) replaces the timer-based approach with a bridge-to-bridge handshake/negotiation mechanism so ports can move to forwarding as soon as it is confirmed safe to do so.

> Instead of "wait the timers and hope it's safe," RSTP says "let's negotiate and confirm it's safe, then go forward immediately."

---

## What Stays the Same

RSTP is an evolution of STP, not a replacement of its logic:

- Same purpose: prevent Layer 2 loops
- Same root bridge election: lowest Bridge ID wins
- Same root port selection: lowest root cost, then tie-breakers
- Same designated port logic: best BPDU on a segment wins

---

## What Changes

### Faster Neighbor Loss Detection
- Classic STP: waits 10 hello intervals (20 seconds) before assuming a neighbor is lost
- RSTP: considers a neighbor lost after **3 missed BPDUs (~6 seconds)**
- After loss detected, RSTP flushes MAC addresses learned on that interface so traffic relearns on the new path quickly

### All Switches Generate BPDUs
- Classic STP: only the root bridge originates BPDUs, others forward them
- RSTP: every switch generates and sends its own BPDUs from designated ports

### Protocol Version
- Classic STP = version 0
- RSTP = version 2

---

## RSTP Port States (Simplified)

Classic STP had 5 states. RSTP collapses them to 3:

| RSTP State | Replaces |
|---|---|
| **Discarding** | Blocking + Listening + Disabled |
| **Learning** | Learning (unchanged) |
| **Forwarding** | Forwarding (unchanged) |

> CLI may still show BLK even though the RSTP state name is Discarding.

---

## RSTP Port Roles

| Role | Description |
|---|---|
| **Root Port** | Same as classic STP — lowest cost path to root |
| **Designated Port** | Same as classic STP — forwards away from root |
| **Alternate Port** | Discarding port receiving superior BPDU from another switch — backup for root port, can transition immediately if root port fails |
| **Backup Port** | Discarding port receiving superior BPDU from another interface on the **same** switch — only occurs on shared/hub segments, rare in modern networks |

---

## RSTP Link Types

| Link Type | Description | Detection |
|---|---|---|
| **Point-to-Point** | Full-duplex switch-to-switch links — enables rapid handshake convergence | Auto-detected (full duplex) |
| **Shared** | Half-duplex hub connections — slower behavior | Auto-detected (half duplex) |
| **Edge** | End-host facing ports — jumps straight to Forwarding | Configured with PortFast |

---

## RSTP Port Costs (Updated for Modern Speeds)

| Speed | RSTP Cost |
|---|---|
| 10 Mbps | 2,000,000 |
| 100 Mbps | 200,000 |
| 1 Gbps | 20,000 |
| 10 Gbps | 2,000 |
| 100 Gbps | 200 |
| 1 Tbps | 20 |

---

## STP Versions — Standard vs Cisco

| Standard | Description |
|---|---|
| IEEE 802.1D | Classic STP — one instance for all VLANs, no load balancing |
| IEEE 802.1w | RSTP — faster convergence, still one instance for all VLANs |
| IEEE 802.1s | MSTP — groups VLANs into instances for load balancing at scale |

| Cisco | Description |
|---|---|
| PVST+ | One STP instance per VLAN — enables per-VLAN load balancing |
| Rapid PVST+ | PVST+ with RSTP speed — **Cisco default on modern switches** |

---

## STP Guard Features

| Feature | Purpose | Behavior |
|---|---|---|
| **PortFast** | Skip Listening and Learning on access ports | Port moves straight to Forwarding — use on host-facing ports only |
| **BPDU Guard** | Protect PortFast ports from rogue switches | Any BPDU received = immediate err-disable. Recovery: `shutdown` then `no shutdown` |
| **Root Guard** | Prevent a different switch from becoming root | Superior BPDU received = port placed in root-inconsistent state |
| **Loop Guard** | Protect against unidirectional link failures | If BPDUs stop on a non-designated port, port moves to loop-inconsistent state instead of forwarding |
| **BPDU Filter** | Stop sending/receiving BPDUs on a port | Interface-level: disables STP entirely (dangerous). Global-level: stops sending BPDUs but reverts if a BPDU is received |

> Root Guard and Loop Guard are mutually exclusive on an interface.
> BPDU Guard and Loop Guard serve different purposes — do not confuse them.
> Global BPDU Filter default is safer than interface-level BPDU Filter.

---

## STP Load Balancing with PVST+

Because PVST+ runs a separate STP instance per VLAN, different VLANs can use different root bridges and block different links. This prevents the same redundant link from sitting idle for all VLANs simultaneously.

**Example:**
- VLAN 1 root = SW1, VLAN 2 root = SW2
- Each VLAN uses a different active path
- Both redundant links carry traffic — no wasted bandwidth

---

## Key Rules

- RSTP converges much faster than classic STP via negotiation, not timers
- RSTP considers neighbor lost after 3 missed hellos (~6 seconds vs 20 in classic STP)
- Alternate port = backup for root port — can transition immediately
- Backup port = backup for designated port — rare, only on hub segments
- Rapid PVST+ is the Cisco default on modern switches
- PortFast + BPDU Guard is the standard combination for access ports
