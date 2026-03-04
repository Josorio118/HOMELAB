# VLANs & 802.1Q Trunking

## The Problem VLANs Solve

In a flat network, every switch port is in the same broadcast domain. Every ARP request, DHCP request, and broadcast frame reaches every device. As networks grow this creates:

- Increased bandwidth consumption
- Broadcast storms that affect all devices
- Security exposure — all devices see all broadcasts
- Harder troubleshooting as scope grows

The traditional solution was separate physical switches and routers — expensive and unscalable.

> VLANs solve this in software. One physical switch, multiple logical networks.

---

## What a VLAN Is

A VLAN (Virtual Local Area Network) is a logical broadcast domain. Devices in different VLANs:

- Do not receive each other's broadcasts
- Cannot communicate without a router
- Behave as if they are on separate physical switches

VLANs exist at Layer 2. They do not require separate hardware.

---

## VLAN vs Subnet — Important Distinction

| Concept | Layer | Controls |
|---|---|---|
| VLAN | Layer 2 | Broadcast scope, frame forwarding |
| Subnet | Layer 3 | IP addressing, routing |

Often mapped 1:1 (one VLAN = one subnet) but they are not the same thing. A VLAN is not a subnet.

---

## Access Ports

- Belongs to exactly one VLAN
- Sends and receives untagged Ethernet frames
- Used for end devices — PCs, laptops, printers
- The end device is completely unaware a VLAN exists

---

## Trunk Ports

- Carries traffic for multiple VLANs on a single cable
- Uses 802.1Q tags to identify which VLAN each frame belongs to
- Used between: Switch ↔ Switch, Switch ↔ Router, Switch ↔ Firewall

---

## 802.1Q Tagging — What Actually Happens

When a frame leaves a switch on a trunk port, the switch inserts a 4-byte VLAN tag into the Ethernet frame containing the VLAN ID.

When that frame arrives at an access port on the far end, the switch strips the tag. The end device never sees it.

---

## Native VLAN

The one VLAN that is sent **untagged** on a trunk link.

- Default native VLAN = VLAN 1
- Must match on both ends of a trunk
- Native VLAN mismatch = traffic leakage and potential VLAN hopping attacks
- Best practice: change native VLAN to an unused VLAN, never use VLAN 1 for real traffic

> Native VLAN is the junk drawer. Don't leave it open.

---

## VLAN Hopping — Security Tie-In

Attackers can exploit misconfigured trunks by sending double-tagged frames to reach VLANs they shouldn't have access to. Mitigations:

- Configure trunks explicitly — never rely on auto-negotiation
- Change native VLAN away from VLAN 1
- Shut down unused ports and assign them to an unused VLAN

---

## DTP — Dynamic Trunking Protocol

DTP automatically negotiates whether a link should become a trunk.

**Best practice:** Manually configure every port as either trunk or access. Never rely on DTP in a production or security-conscious environment.

---

## Key Rules

- VLANs = logical broadcast domains at Layer 2
- Devices in different VLANs cannot communicate without a router
- Access ports = one VLAN, untagged frames, end devices
- Trunk ports = multiple VLANs, 802.1Q tagged frames, network devices
- Native VLAN = the one untagged VLAN on a trunk — must match on both ends
- Native VLAN mismatch = traffic leakage and security risk
- VLAN ≠ Subnet (though often mapped 1:1)
