# 2026-01-09 — VLAN Creation & Access Port Assignment

## Objective
Implement Layer 2 segmentation using VLANs. Intentionally no inter-VLAN routing at this stage — the goal was to observe isolation behavior clearly before introducing a router.

## Starting State
- All ports in DEFAULT_VLAN (VLAN 1) — single broadcast domain
- ISP router on Port 3, iMac on Port 4, laptop on Port 10

---

## Configuration Applied

```
vlan 10
  name USER_LAN
  untagged 3,4
  exit

vlan 20
  name LAB_TEST
  untagged 10
  exit

vlan 1
  no untagged 3,4,10
  exit

write memory
```

### Verification
```
show vlan 10
show vlan 20
show mac-address
```

---

## Observations

| Device | VLAN | Result |
|---|---|---|
| iMac (Port 4) | VLAN 10 | Full internet connectivity maintained |
| Laptop (Port 10) | VLAN 20 | No internet — Wi-Fi disabled, Ethernet only |
| Port 3 (router) | VLAN 10 | Many MACs — all devices behind ISP router |

- No MAC flapping observed — no loops or instability
- MAC address table showed correct per-VLAN learning

> Loss of connectivity on VLAN 20 was intentional. This confirms isolation is working, not that something is broken.

---

## Key Takeaway

VLANs segment broadcast domains at Layer 2. Two devices on the same physical switch can be completely isolated from each other with zero routing involved. This is the foundation of VLAN design — separate first, route intentionally second.
