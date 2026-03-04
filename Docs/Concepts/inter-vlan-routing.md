# Inter-VLAN Routing

## Why a Switch Alone Isn't Enough

Switches operate at Layer 2 and forward frames based on MAC addresses. VLANs create separate broadcast domains. A switch has no concept of IP routing — traffic destined for a different VLAN has no path to get there without a Layer 3 device.

> VLANs separate traffic. Routers reconnect traffic intentionally.
> If routing works, it's because you designed it. If routing doesn't work, it's because you haven't enabled it yet.

---

## Router-on-a-Stick

One physical router interface carries traffic for multiple VLANs using logical subinterfaces. Each subinterface is associated with one VLAN and acts as the default gateway for devices on that VLAN.

**Requirements:**
- The switch-to-router link must be configured as a trunk
- Each subinterface must be configured with `encapsulation dot1q <vlan-id>`
- Each subinterface must have an IP address — this becomes the default gateway for that VLAN
- The physical router interface itself has **no IP address**

**Naming convention (best practice):** subinterface number matches VLAN number
- `interface g0/0.10` → VLAN 10
- `interface g0/0.20` → VLAN 20

---

## End-to-End Traffic Flow — VLAN 10 to VLAN 20

1. PC on VLAN 10 sees destination IP is outside its subnet
2. PC sends frame to its default gateway (192.168.10.1)
3. Switch forwards frame — still tagged VLAN 10
4. Frame travels over trunk to router — tagged VLAN 10
5. Router receives frame on subinterface .10
6. Router makes routing decision — destination is VLAN 20
7. Router sends frame out subinterface .20 — tagged VLAN 20
8. Switch receives tagged frame — forwards to VLAN 20 access port
9. Switch strips VLAN tag — destination PC receives untagged frame

---

## Inter-VLAN Routing Fails When

- Trunk link is missing or misconfigured
- VLAN tag on subinterface doesn't match the VLAN on the switch
- Host default gateway is set incorrectly
- Subinterface is missing `encapsulation dot1q` command

---

## pfSense Implementation (This Lab)

In this lab pfSense CE acts as the router. Instead of Cisco subinterfaces, pfSense uses VLAN interfaces configured on the trunk-connected USB Ethernet adapter.

| pfSense Interface | VLAN | IP Address | Role |
|---|---|---|---|
| em0 (VLAN 99) | 99 | 192.168.99.1/24 | Management LAN |
| em0.10 | 10 | 192.168.10.1/24 | Users default gateway |
| em0.20 | 20 | 192.168.20.1/24 | Test default gateway |

Traffic flow is identical to router-on-a-stick — pfSense is simply doing the routing in software on a VM rather than on dedicated Cisco hardware.

---

## Key Rules

- Switches cannot route between VLANs — a Layer 3 device is required
- Router-on-a-stick uses one physical interface with multiple logical subinterfaces
- Each subinterface needs: `encapsulation dot1q <vlan-id>` + an IP address
- The physical interface has no IP address in this design
- The subinterface IP is the default gateway for devices in that VLAN
- The switch-to-router link must be a trunk
