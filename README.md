# CCNA Home Lab — Study Journal & Lab Documentation

> **HP ProCurve 2524 · pfSense CE 2.8.1 · VMware Fusion · Cisco Packet Tracer**
> January – February 2026

A hands-on CCNA study lab built on real hardware and a virtualized firewall. This repo documents the full journey — physical setup, every major mistake, all configurations, and Packet Tracer labs — organized as a working reference and study log.

---

## Lab Environment

| Component | Details |
|---|---|
| **Switch** | HP ProCurve 2524 (J4813A) — Firmware F.01.08 |
| **Firewall/Router** | pfSense CE 2.8.1 (VMware Fusion VM) |
| **Host Machine** | iMac — VMware Fusion hypervisor |
| **Upstream Router** | ISP Router (192.168.1.0/24) |
| **NICs** | Built-in Ethernet (management) + USB-to-Ethernet Adapter (trunk) |
| **Test Device** | Laptop via USB-C Ethernet dock |

---

## Network Topology

```
Internet
    |
ISP Router (192.168.1.0/24)
    |
pfSense WAN (em1) — DHCP from ISP
    |
pfSense LAN (em0) — 802.1Q Trunk
    |
HP ProCurve 2524 — Port 5 (Trunk)
    |__________________________|_________________________|
    |                          |                         |
Port 3 (ISP uplink)     Port 4 (MGMT/VLAN 99)    Port 10/11 (VLAN 10/20)
```

### VLAN Design

| VLAN | Name | Subnet | Gateway |
|---|---|---|---|
| VLAN 10 | USERS | 192.168.10.0/24 | 192.168.10.1 |
| VLAN 20 | TEST | 192.168.20.0/24 | 192.168.20.1 |
| VLAN 99 | MGMT | 192.168.99.0/24 | 192.168.99.1 |

### Switch Port Reference

| Port | Role |
|---|---|
| Port 3 | Access — ISP router uplink |
| Port 4 | Access — Management (VLAN 99 untagged) |
| Port 5 | 802.1Q Trunk — to pfSense USB Ethernet |
| Port 10 | Access — VLAN 10 test device |
| Port 11 | Access — VLAN 20 test device |

### pfSense Interface Assignments

| Interface | Assignment |
|---|---|
| WAN (em1) | DHCP from ISP |
| LAN / VLAN 99 (em0) | 192.168.99.1/24 — static |
| VLAN10_USERS (em0.10) | 192.168.10.1/24 — static |
| VLAN20_TEST (em0.20) | 192.168.20.1/24 — static |

---

## Current Lab State

- ✅ HP ProCurve 2524 — STP enabled, root bridge at priority 4096
- ✅ 802.1Q trunk on Port 5 — VLAN 10, 20, 99 carrying correctly
- ✅ pfSense VM — WAN DHCP, three VLAN interfaces routed
- ✅ DHCP per VLAN — devices receive correct IP, gateway, DNS
- ✅ Firewall rules — VLAN 10/20 blocked from MGMT VLAN 99
- ✅ Dual-homed management — USB adapter on VLAN 99, built-in on ISP
- ✅ pfSense GUI accessible at https://192.168.99.1
- ✅ STP loop test — loop blocked correctly, convergence observed
- ✅ Wireshark verified — BPDUs, ARP, TCP sessions, VLAN-tagged traffic

---

## Repository Structure

```
ccna-homelab/
├── README.md
├── docs/
│   ├── journal/               # Chronological lab session logs
│   ├── concepts/              # CCNA concept notes by topic
│   └── topology/              # Diagrams and VLAN reference tables
├── configs/
│   ├── hp-procurve-2524/      # Switch VLAN, STP, trunk configs
│   └── pfsense/               # Interface assignments, firewall rules
└── mistakes-and-lessons.md   # Real failures, diagnoses, and fixes
```

---

## Lab Journal — Session Index

| Date | Session | Key Topics |
|---|---|---|
| Jan 4 | Physical setup, first Layer 2 observations | MAC learning, Layer 2 loops |
| Jan 5 | MAC aging and port behavior verification | Aging timer, dynamic learning |
| Jan 6 | Unknown unicast flooding study | Flooding behavior, Wireshark |
| Jan 9 | VLAN creation and access port assignment | Layer 2 segmentation, VLAN isolation |
| Day 6 | pfSense VM installation (VMware Fusion) | Virtualization networking, VMware adapter modes |
| Day 6+ | pfSense + VLAN integration | Architecture constraints, dual NIC design |
| Feb 1 | Router-on-a-stick full build | 802.1Q trunk, inter-VLAN routing, DHCP per VLAN |
| Feb 1 | STP configuration and root bridge control | Root election, STP timers, loop test |
| Feb 1 | Firewall segmentation | pfSense rules, VLAN isolation enforcement |

---

## CCNA Topics Covered

- OSI Model — Layers 1–3 focus
- Layer 2 Switching — MAC learning, aging, flooding, forwarding
- ARP — broadcast discovery, ARP cache, local vs routed traffic
- VLANs — broadcast domains, access vs trunk ports, 802.1Q tagging, native VLAN
- Inter-VLAN Routing — router-on-a-stick, subinterfaces, encapsulation dot1q
- STP Classic (802.1D) — root election, port roles, port states, timers
- RSTP (802.1w) — convergence improvements, port roles, link types
- PVST+ and Rapid-PVST+ — per-VLAN STP instances, load balancing
- STP Guard Features — PortFast, BPDU Guard, Root Guard, Loop Guard, BPDU Filter
- pfSense — VLAN interfaces, DHCP server, firewall rules, NAT

## Topics Still Ahead

- IP addressing and subnetting
- IPv6
- Routing protocols — OSPF, static routes, administrative distance
- ACLs
- NAT
- DHCP on Cisco IOS
- Wireless networking
- WAN concepts
- Network security fundamentals

---

## Notable Mistakes (The Real Learning)

Full write-ups in [`mistakes-and-lessons.md`](./mistakes-and-lessons.md). Short version:

- **Layer 2 loop on Day 1** — duplicate cable from router to switch caused broadcast storm before STP was even configured. Diagnosed via MAC address table showing same MAC on multiple ports.
- **pfSense installer loop** — repeated cycling through installer screens was VMware adapter instability, not pfSense misconfiguration. Changing adapter modes mid-install made it worse.
- **Trying to route before securing management access** — pfSense LAN set to VLAN 99 subnet while iMac was on ISP subnet with no path between them. Couldn't reach WebGUI to fix it.
- **Single NIC architecture constraint** — discovered that a VM router cannot share one physical NIC between untagged management traffic and a tagged VLAN trunk. Required adding a USB Ethernet adapter as a dedicated trunk interface.

---

*CCNA Home Lab · January–February 2026*
