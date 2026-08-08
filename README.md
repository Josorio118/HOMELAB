# CCNA Home Lab — Study Journal & Lab Documentation

> **HP ProCurve 2524 · pfSense CE 2.8.1 · VMware Fusion · LibreNMS · Cisco Packet Tracer**
> January 2026 – Present

A hands-on CCNA study lab built on real hardware and a virtualized firewall. This repo documents the full journey — physical setup, every major mistake, all configurations, and Packet Tracer labs — organized as a working reference and study log.

---

## Lab Environment

| Component | Details |
|---|---|
| **Switch** | HP ProCurve 2524 (J4813A) — Firmware F.01.08 |
| **Firewall/Router** | pfSense CE 2.8.1 (VMware Fusion VM) |
| **Host Machine** | 2017 iMac (macOS 13.7) — VMware Fusion hypervisor |
| **Monitoring** | LibreNMS (Docker via Colima) — SNMP polling + Discord alerting |
| **Upstream Router** | Xfinity ISP router (WAN) |
| **NICs** | Built-in Ethernet (WAN) + USB-to-Ethernet Adapter (trunk, bridged as LAN/em0) |
| **Test Device** | MacBook Pro — used as lab endpoint |

---

## Network Topology

```
Internet
    |
Xfinity Router (WAN)
    |
iMac built-in Ethernet (em1 / WAN)
    |
pfSense
    |
USB Ethernet Adapter (em0 / LAN — 802.1Q Trunk)
    |
HP ProCurve 2524 — Port 5 (Trunk, only active connectivity port)
```

Router-on-a-stick design: a single trunk (Port 5) carries all VLAN traffic between pfSense and the switch.

### VLAN Design

| VLAN | Name | Subnet | Gateway | Tagging on Trunk |
|---|---|---|---|---|
| VLAN 99 | MGMT | 192.168.99.0/24 | 192.168.99.1 | Native / untagged |
| VLAN 10 | USERS | 192.168.10.0/24 | 192.168.10.1 | Tagged |
| VLAN 20 | TEST | 192.168.20.0/24 | 192.168.20.1 | Tagged |

- ProCurve management IP: `192.168.99.27`
- iMac (host): `192.168.99.100`

### Switch Port Reference

| Port | Role |
|---|---|
| Port 5 | 802.1Q Trunk — only active connectivity port, to pfSense via USB Ethernet |
| Ports 1–4, 6–9, 12–24 | Unused — administratively disabled to reduce attack surface |
| Ports 25–26 | SFP transceiver slots — unpopulated |

*(Port 4 was originally used as a dedicated VLAN 99 management port; VLAN 99 now rides natively/untagged on the Port 5 trunk and Port 4 is unused.)*

### pfSense Interface Assignments

| Interface | Assignment |
|---|---|
| WAN (em1) | DHCP from Xfinity |
| LAN / VLAN 99 (em0) | 192.168.99.1/24 — static |
| VLAN10_USERS (em0.10) | 192.168.10.1/24 — static |
| VLAN20_TEST (em0.20) | 192.168.20.1/24 — static |

---

## Current Lab State

- ✅ HP ProCurve 2524 — STP enabled, VLAN trunking on Port 5 confirmed
- ✅ 802.1Q trunk on Port 5 — VLAN 10, 20, 99 carrying correctly (99 native)
- ✅ pfSense VM — WAN DHCP, three VLAN interfaces routed
- ✅ DHCP per VLAN — devices receive correct IP, gateway, DNS
- ✅ Firewall rules — VLAN 10/20 isolated from MGMT (VLAN 99)
- ✅ pfSense GUI accessible at `https://192.168.99.1`
- ✅ Wireshark verified on iMac's trunk NIC — ARP, TCP sessions, VLAN-tagged traffic (preferred over unreliable pfSense packet capture GUI)
- ✅ Port security — access port lockdown completed and tested
- ✅ Unused switch ports — administratively disabled (Ports 1–4, 6–9, 12–24)
- ✅ SNMP monitoring — LibreNMS deployed via Docker/Colima, polling pfSense and switch
- ✅ SNMP trap forwarding — switch traps to LibreNMS, verified via live port flap test
- ✅ Discord alerting — LibreNMS alert rules wired to Discord webhook, ignore-tags tuned for disabled ports
- ✅ Switch clock — manually set with Eastern DST offset (ProCurve's `ip timep dhcp` is incompatible with pfSense NTP)
- ✅ Static routing lab — full verify → break → restore cycle using a phantom next-hop and loopback alias to simulate a remote network
- ✅ Root-caused intermittent SNMP flapping to Spotlight indexing VM/Docker disk files — resolved via Spotlight Privacy exclusions
- 🔄 Second-hand Cisco gear acquired (2950-24 switches, 2600-series routers, ASA 5505/5545-X firewalls) — being parted out and integrated for HSRP/EtherChannel redundancy labs
- ⏭ OSPF via pfSense FRR — planned after static routing is fully documented
- ⏭ ProCurve ACL lab, OpenCanary honeypot, Suricata IDS — planned

---

## Repository Structure

```
HOMELAB/
├── README.md
├── Docs/
│   ├── Lab Session Logs/       # Dated, kebab-case session logs
│   ├── mistakes-and-lessons.md # Numbered real failures, diagnoses, fixes (19+ entries)
│   └── maintenance_log.md      # Ongoing maintenance notes
├── configs/
│   ├── hp-procurve-2524/       # Switch VLAN, STP, trunk configs
│   └── pfsense/                # Interface assignments, firewall rules
└── photos/                     # Hardware teardown / build photos (in progress)
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
| Jul 14 | Firewall rules / ACLs | VLAN20_TEST firewall rules on pfSense |
| Jul 15 | Port security | Switch port security configuration and testing |
| Jul 19 | LibreNMS deployment | Docker/Colima, SNMP polling, Discord alerting |
| Jul 25 | SNMP trap forwarding, port hardening, alert tuning | Trap receiver config, unused port disable, LibreNMS ignore-tags, switch clock fix |
| — | VLAN 99 ARP/hairpin troubleshooting | Root-caused to VMware Fusion bridged adapter "no carrier" state |
| — | Static routing lab | Phantom next-hop, loopback alias, verify/break/restore methodology |
| — | SNMP flapping root cause investigation | Spotlight indexing I/O contention on VM/Docker disk files |
| — | Second-hand Cisco gear acquisition | 2950s, 2600s, ASA 5505/5545-X evaluated for redundancy lab build-out |

---

## CCNA Topics Covered

- OSI Model — Layers 1–3 focus
- Layer 2 Switching — MAC learning, aging, flooding, forwarding
- ARP — broadcast discovery, ARP cache, local vs. routed traffic
- VLANs — broadcast domains, access vs. trunk ports, 802.1Q tagging, native VLAN
- Inter-VLAN Routing — router-on-a-stick, subinterfaces, encapsulation dot1q
- STP Classic (802.1D) — root election, port roles, port states, timers
- Static Routing — administrative distance, default-route fallback behavior, route verification
- pfSense — VLAN interfaces, DHCP server, firewall rules, NAT
- Port Security — intrusion alerts, MAC-based restrictions
- SNMP — polling, trap forwarding, event levels, network monitoring (LibreNMS)

## Topics Still Ahead

- OSPF (via pfSense FRR)
- IP addressing and subnetting deep-dive
- IPv6
- Extended ACLs (physical, on ProCurve)
- NAT
- Wireless networking
- WAN concepts
- HSRP / EtherChannel (new Cisco gear)
- VPN (OpenVPN/WireGuard) as prerequisite for SSH hardening on pfSense

---

## Notable Mistakes (The Real Learning)

Full write-ups in [`mistakes-and-lessons.md`](./Docs/mistakes-and-lessons.md) — currently 19+ entries. Highlights:

- **Layer 2 loop on Day 1** — duplicate cable from router to switch caused a broadcast storm before STP was even configured. Diagnosed via MAC address table showing the same MAC on multiple ports.
- **VLAN 99 ARP/hairpin issue** — VMware Fusion reported "no carrier" on the bridged adapter (em0); host-originated frames weren't looping back correctly through the virtual bridge into the pfSense VM. Fixed by reconnecting the adapter in Fusion's network settings.
- **Single NIC architecture constraint** — a VM router cannot share one physical NIC between untagged management traffic and a tagged VLAN trunk. Required adding a dedicated USB Ethernet adapter as the trunk interface.
- **SFP slots don't take standard port-disable syntax** — ports 25–26 on the 2524 are transceiver slots, not copper, and reject bulk `interface disable` commands the same way the other 24 ports accept them.
- **Disabling a port on the switch doesn't stop LibreNMS from alerting on it** — needed the separate `Ignore alert` tag per port in LibreNMS to quiet a stale "port down" alert.
- **Switch clock has no persistent save** — `write memory` doesn't cover the clock; `reload`/`boot` always resets it to January 1990. `ip timep dhcp` is also incompatible with pfSense's NTP implementation (ProCurve uses the older Timep protocol) — set manually instead.
- **Intermittent SNMP flapping traced to Spotlight** — macOS Spotlight indexing the VMware and Docker VM disk files caused I/O contention severe enough to flap SNMP polling. Not a sleep/display issue — resolved via Spotlight Privacy exclusions on both folders.
- **Default-route fallback confirmed the hard way** — during a static route break test, a missing specific route didn't fail locally; traffic fell through to the default route and got an ICMP "Communication prohibited by filter" from the Comcast edge instead.
