# 2026-02-01 — Router-on-a-Stick Full Build, STP Root Control & Firewall Segmentation

## Objective
Major milestone session. Implement a full router-on-a-stick architecture with pfSense routing multiple VLANs, take control of STP root bridge election, validate with a live loop test, and enforce firewall segmentation policy.

---

## What Was Built

- 802.1Q trunk from switch Port 5 to pfSense USB Ethernet adapter
- Three routed VLANs: 99 (management), 10 (users), 20 (test)
- DHCP per VLAN — devices received correct IP, gateway, and DNS automatically
- Firewall rules per VLAN interface — controlled internet access with pfSense default-deny
- NAT on WAN — all internal traffic NATed through ISP IP

---

## Switch Trunk Configuration

```
vlan 10
  untagged 10
  tagged 5
  exit

vlan 20
  untagged 11
  tagged 5
  exit

vlan 99
  untagged 4
  untagged 5
  exit

vlan 1
  no untagged 10,11
  exit

write memory
```

---

## Validation Results

| VLAN | Device IP | Gateway | Internet |
|---|---|---|---|
| VLAN 10 | 192.168.10.x | 192.168.10.1 | ✅ |
| VLAN 20 | 192.168.20.x | 192.168.20.1 | ✅ |
| VLAN 99 | 192.168.99.x | 192.168.99.1 | ✅ |

- Wi-Fi disabled during testing to force Ethernet path only
- YouTube and ping tests confirmed internet access through pfSense NAT
- VLAN isolation confirmed — VLAN 20 could not reach VLAN 10 without explicit rules

---

## STP Root Bridge Control

### Initial State
Running `show spanning-tree` revealed STP was disabled by default on the HP ProCurve 2524. When enabled, the ISP router was elected root bridge because it was sending BPDUs with a lower bridge ID.

```
Root MAC Address : <ISP router MAC>
Root Path Cost   : 10
Root Port        : 3
Root Priority    : 32767
```

> The ISP router was winning the root election. The HP ProCurve should be root since it is the core Layer 2 device in this topology.

### Root Bridge Takeover

```
spanning-tree
spanning-tree priority 4096
write memory
```

### Verified State

```
Switch Priority  : 4096
Root Path Cost   : 0
Root Port        : This switch is root
Root Priority    : 4096
Hello Time: 2  Max Age: 20  Forward Delay: 15
```

> After taking root, STP timers normalized to 802.1D defaults. Before, the switch inherited whatever timer values the ISP router was advertising.

---

## Loop Test — STP in Action

1. Plugged a cable between two unused switch ports to create a physical Layer 2 loop
2. STP detected the loop and placed Port 25 into Blocking state
3. Removed cable — port returned to Forwarding after convergence
4. Wireshark confirmed BPDUs advertising Root Priority 4096

> Port 25 blocked because when both ends of a loop belong to the same switch, the higher port number becomes Blocking. Deterministic, rule-based behavior.

---

## Dual-Homed Management Access

Rather than swapping cables every time pfSense management is needed:

| Interface | Role |
|---|---|
| Built-in Ethernet (en0) | Connected to ISP router — internet access |
| USB Ethernet Adapter | Connected to Port 4 — VLAN 99 untagged — pfSense management |
| pfSense GUI | https://192.168.99.1 — accessible via USB adapter |

> This is how engineers manage firewalls in production — dedicated management interface, separate from data plane traffic.

---

## Firewall Segmentation Rules

pfSense processes rules top-down — first match wins.

### WAN
- No inbound rules — implicit deny all
- NAT handles all outbound traffic

### VLAN 10 (USERS)

| Priority | Action | Source | Destination | Protocol |
|---|---|---|---|---|
| 1 | BLOCK | VLAN 10 subnets | 192.168.99.0/24 | Any |
| 2 | PASS | VLAN 10 subnets | Any | IPv4 |

### VLAN 20 (TEST)

| Priority | Action | Source | Destination | Protocol |
|---|---|---|---|---|
| 1 | BLOCK | VLAN 20 subnets | LAN subnets | Any |
| 2 | PASS | VLAN 20 subnets | Any | IPv4 |

> Rule order matters. The block rule must be above the allow rule. pfSense default-deny means unlisted traffic is blocked — rules must be explicit.
