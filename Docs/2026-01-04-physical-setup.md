# 2026-01-04 — Physical Setup & First Layer 2 Observations

## Objective
First physical lab session. Get all hardware connected, verify switch is operational, and begin observing Layer 2 behavior.

## What Was Done
- Connected ISP router and iMac to HP ProCurve 2524
- Verified switch firmware via `show version` — confirmed F.01.08
- Verified active ports via `show interfaces` — Ports 3 and 4 active at 100FDx (100 Mbps, Full Duplex)
- Attempted to rename Port 3 with `name ROUTER_UPLINK` — command unsupported on this firmware
- Used `show mac-address` to identify device-to-port mappings instead
- Connected laptop via USB-C dock to Port 10 and observed MAC learning on that port

---

## The First Real Mistake — Layer 2 Loop

The original blue Ethernet cable connected the ISP router directly to the iMac. When setting up the switch, a second cable was run from the router to the switch — but the original cable was accidentally left plugged in, connecting the same router to the switch twice.

**Result:** Network broke immediately — no internet, no communication.

**Diagnosis:** `show mac-address` showed the same MAC appearing on multiple ports — classic loop indicator.

**Fix:** Removed the duplicate cable. One connection router-to-switch only.

**Recovery:** Connectivity restored immediately after correcting cabling.

> Connecting the same device to a switch twice at Layer 2 creates a loop. Broadcast traffic circulates indefinitely because switches have no TTL at Layer 2.

---

## What Was Learned

- Switches forward broadcasts out all ports — without STP, loops are fatal
- MAC address tables reveal topology without physically tracing cables
- Port 3 (router uplink) showed many MACs — all personal devices behind the ISP router
- Port 4 (iMac) showed one MAC — single device access port behavior
- Port 10 (dock) showed MAC learned dynamically when laptop was plugged in
- Unplugging the dock eventually caused Port 10 MAC to age out — default aging ~300 seconds
