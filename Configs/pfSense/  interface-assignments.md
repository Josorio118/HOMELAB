# pfSense CE 2.8.1 — Interface Assignments & VLAN Configuration

## Interface Assignments

| Interface | Role | Adapter | Address |
|---|---|---|---|
| em1 | WAN | Built-in Ethernet (en0) | DHCP from ISP |
| em0 | LAN / VLAN 99 | USB Ethernet Adapter | 192.168.99.1/24 (static) |
| em0.10 | VLAN10_USERS | USB Ethernet Adapter | 192.168.10.1/24 (static) |
| em0.20 | VLAN20_TEST | USB Ethernet Adapter | 192.168.20.1/24 (static) |

---

## VLAN Configuration

VLANs are configured under **Interfaces → Assignments → VLANs** in the pfSense WebGUI.

| VLAN Tag | Parent Interface | Description |
|---|---|---|
| 10 | em0 | VLAN10_USERS |
| 20 | em0 | VLAN20_TEST |

VLAN 99 is handled as the native/untagged traffic on em0 — it does not require a separate VLAN tag entry.

---

## DHCP Server — Per VLAN

DHCP is enabled on each VLAN interface under **Services → DHCP Server**.

| Interface | Range | Gateway | DNS |
|---|---|---|---|
| VLAN 99 (LAN) | 192.168.99.100 – 192.168.99.200 | 192.168.99.1 | 192.168.99.1 |
| VLAN 10 | 192.168.10.100 – 192.168.10.200 | 192.168.10.1 | 192.168.10.1 |
| VLAN 20 | 192.168.20.100 – 192.168.20.200 | 192.168.20.1 | 192.168.20.1 |

---

## VMware Fusion — Network Adapter Setup

| Adapter | VMware Mode | Physical NIC | Role |
|---|---|---|---|
| em1 (WAN) | Bridged | Built-in Ethernet (en0) | ISP-facing WAN |
| em0 (LAN) | Bridged | USB Ethernet Adapter | VLAN trunk to switch |

> Both adapters must be bridged to separate physical NICs. Sharing one NIC between untagged management traffic and a VLAN trunk is an architecture constraint — it does not work.
