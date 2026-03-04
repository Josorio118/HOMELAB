# Lab Topology Reference

## Physical Topology

```
Internet
    |
ISP Router (192.168.1.0/24)
    |
    |— Port 3 (ISP uplink, untagged)
    |
HP ProCurve 2524
    |— Port 4  (MGMT, VLAN 99 untagged)
    |— Port 5  (802.1Q Trunk → pfSense USB Ethernet)
    |— Port 10 (VLAN 10 access)
    |— Port 11 (VLAN 20 access)
         |
    pfSense CE 2.8.1 (VMware Fusion VM)
         |— em1  → WAN  (DHCP from ISP)
         |— em0  → VLAN 99 / LAN  (192.168.99.1/24)
         |— em0.10 → VLAN 10 (192.168.10.1/24)
         |— em0.20 → VLAN 20 (192.168.20.1/24)
```

---

## VLAN Design

| VLAN | Name | Subnet | Gateway | Purpose |
|---|---|---|---|---|
| VLAN 10 | USERS | 192.168.10.0/24 | 192.168.10.1 | Primary user network |
| VLAN 20 | TEST | 192.168.20.0/24 | 192.168.20.1 | Isolated lab/test network |
| VLAN 99 | MGMT | 192.168.99.0/24 | 192.168.99.1 | Management + pfSense LAN |

---

## Switch Port Reference

| Port | VLAN | Mode | Connected Device |
|---|---|---|---|
| Port 3 | VLAN 1 | Access (untagged) | ISP router uplink |
| Port 4 | VLAN 99 | Access (untagged) | iMac — management access |
| Port 5 | VLAN 10, 20, 99 | 802.1Q Trunk | pfSense USB Ethernet adapter |
| Port 10 | VLAN 10 | Access (untagged) | Test device |
| Port 11 | VLAN 20 | Access (untagged) | Test device (alternate) |

---

## pfSense Interface Assignments

| Interface | Role | Address | Notes |
|---|---|---|---|
| em1 | WAN | DHCP from ISP | Bridged to built-in Ethernet (en0) |
| em0 | LAN / VLAN 99 | 192.168.99.1/24 | Bridged to USB Ethernet adapter |
| em0.10 | VLAN10_USERS | 192.168.10.1/24 | DHCP server enabled |
| em0.20 | VLAN20_TEST | 192.168.20.1/24 | DHCP server enabled |

---

## Management Access — Dual-Homed Setup

| Interface | Connected To | Address | Purpose |
|---|---|---|---|
| Built-in Ethernet (en0) | ISP router | 192.168.1.x (DHCP) | Internet access |
| USB Ethernet Adapter | Switch Port 4 | 192.168.99.x (DHCP) | pfSense WebGUI access |
| pfSense WebGUI | — | https://192.168.99.1 | Firewall management |

---

## STP State

| Setting | Value |
|---|---|
| STP Mode | 802.1D (enabled) |
| Switch Priority | 4096 |
| Root Bridge | This switch |
| Hello Time | 2 seconds |
| Max Age | 20 seconds |
| Forward Delay | 15 seconds |
