# pfSense + VLAN Integration — Architecture Constraint Discovery

## Objective
Integrate pfSense with the HP ProCurve VLAN setup and establish stable inter-VLAN routing. This session exposed a fundamental architecture constraint that required a hardware change to resolve.

---

## Why Progress Was Stuck

- Attempted VLAN configuration and routing before stable pfSense access was established
- pfSense LAN was set to 192.168.99.1 while iMac was on 192.168.1.x — completely different subnets with no path between them
- Could not reach pfSense WebGUI to make changes — needed pfSense access to fix pfSense access
- Switch configs were being changed without confirmed connectivity

> Key mistake: trying to do Layer 3 routing before confirming Layer 2 access. Always secure management access first.

---

## Access Recovery Steps

1. Switch flattened to VLAN 1 only — ports 3 and 4 untagged in VLAN 1, all other VLANs removed from management ports
2. iMac: Wi-Fi OFF, Ethernet ON — confirmed IP from ISP router
3. VMware: pfSense VM adapter set to Bridged, bridged to Ethernet (en0)
4. pfSense console: WAN = DHCP, LAN = DHCP (temporary) — pfSense pulled an IP
5. WebGUI became reachable — first repeatable access established

---

## Architecture Constraint Discovered

Attempting to trunk VLAN 20 on Port 4 (the iMac management port) caused management drops. This led to the discovery of a fundamental constraint:

> A VM router cannot share a physical NIC between untagged management traffic (iMac) and tagged VLAN trunk traffic (pfSense). This is not a configuration mistake — it is an architecture constraint.

---

## The Fix — Second NIC

| Item | Detail |
|---|---|
| Problem | pfSense VM had only one physical NIC — already required for iMac management |
| Solution | Add USB-to-Ethernet adapter as a dedicated trunk interface |
| Result | Built-in Ethernet = iMac management (untagged / ISP side) |
| Result | USB Ethernet = pfSense VLAN trunk (carries VLAN 10, 20, 99 tagged) |
| Architecture | Clean separation of management plane vs data plane |
