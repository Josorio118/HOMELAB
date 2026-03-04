# Mistakes & Lessons Learned

Real failures encountered during this lab build, how each one was diagnosed, and what was learned. These are documented because the mistakes taught more than the parts that worked.

---

## 1. Layer 2 Loop on Day 1

### What Happened
During initial physical setup, a cable was run from the ISP router to the switch. The original direct cable between the router and the iMac was left plugged in at the same time. This created two active Layer 2 paths between the same devices — a loop.

The network broke immediately. No internet, no local communication.

### Diagnosis
Running `show mac-address` on the switch revealed the same MAC address appearing on multiple ports simultaneously. This is the classic indicator of a Layer 2 loop — a switch seeing the same source MAC on more than one port means a frame is circulating and arriving from multiple directions.

### Fix
Removed the duplicate cable. One connection between router and switch only.

### What Was Learned
- Switches flood broadcasts out all ports and have no TTL at Layer 2 — without STP, a loop causes frames to circulate indefinitely
- The MAC address table is a diagnostic tool, not just an operational one — it can reveal topology problems without physically tracing cables
- STP exists specifically because redundant physical links are desirable for fault tolerance, but dangerous without loop prevention

---

## 2. pfSense Installer Loop (VMware Fusion)

### What Happened
The pfSense CE installer repeatedly cycled through the same screens — Connectivity Check, Interface Assignment, LAN/WAN Setup — every time Continue was pressed it looped back to the beginning. This went on for an extended period and appeared to be a pfSense configuration error.

Making it worse: VMware network adapter modes were changed multiple times while the installer was running, trying to find a combination that would work.

### Diagnosis
The interface mapping inside pfSense was actually correct the entire time. The loop was caused by VMware networking instability — the installer could not complete its connectivity check because the underlying virtual network kept changing.

Changing adapter modes mid-installation made the behavior increasingly unpredictable and extended the confusion significantly.

### Fix
Stopped changing adapter settings. Stabilized the VMware adapter configuration and left it alone long enough for the installer to complete its connectivity check and progress past the loop.

### What Was Learned
- Virtual machine network adapter modes are not interchangeable — pick one and commit before starting an installer
- Changing network config mid-installation creates unpredictable state that is harder to diagnose than the original problem
- When an installer appears to loop, the cause is usually environmental instability, not a configuration error inside the installer itself
- pfSense and any virtual firewall must be treated like real network appliances — the installer has real networking dependencies

---

## 3. Trying to Route Before Securing Management Access

### What Happened
After pfSense was installed, VLAN configuration and inter-VLAN routing were attempted before stable WebGUI access was established. pfSense LAN was set to 192.168.99.1/24 while the iMac was on 192.168.1.x from the ISP router — completely different subnets with no path between them.

The result was a catch-22: pfSense WebGUI was unreachable, but pfSense WebGUI access was needed to fix the configuration that was blocking pfSense WebGUI access.

Switch configurations were also being changed without confirmed end-to-end connectivity, which added more variables to an already unclear state.

### Diagnosis
The iMac had no route to 192.168.99.0/24. The pfSense LAN interface only existed inside the virtual network, and the Mac was sitting on the WAN side with no bridge to cross.

### Fix
1. Flattened the switch back to VLAN 1 only — removed all VLAN assignments from management ports
2. Turned Wi-Fi off, confirmed iMac was on Ethernet getting an IP from the ISP router
3. Set pfSense VM adapter to Bridged on the correct physical NIC
4. Temporarily set pfSense LAN to DHCP to let it pull a reachable IP
5. WebGUI became accessible — established a stable baseline before making any further changes

### What Was Learned
- Always secure management access before attempting to build anything else — you cannot configure a device you cannot reach
- Layer 2 connectivity must be confirmed before attempting Layer 3 configuration
- When stuck in a configuration loop, flatten to a known working baseline and rebuild incrementally
- Every change should be verified before making the next one

---

## 4. Single NIC Architecture Constraint

### What Happened
After pfSense was accessible, the next attempt was to trunk VLANs on Port 4 — the same port the iMac used for management access. The goal was to have pfSense receive tagged VLAN traffic on the same physical NIC that the iMac used for untagged management.

Every time the trunk was configured on Port 4, management access dropped. Reconfiguring the trunk in different ways produced the same result.

### Diagnosis
This was not a misconfiguration — it was a fundamental architecture constraint. A single physical NIC cannot simultaneously carry:
- Untagged traffic for the iMac (which expects untagged frames and has no VLAN awareness)
- Tagged 802.1Q trunk traffic for pfSense (which needs to receive and process VLAN tags)

Both devices were sharing the same physical wire. There was no way to separate their traffic at Layer 2.

### Fix
Added a USB-to-Ethernet adapter as a second physical NIC dedicated entirely to the pfSense VLAN trunk. This gave the lab two completely separate physical paths:

- **Built-in Ethernet (en0)** → iMac management, untagged, ISP side
- **USB Ethernet Adapter** → pfSense trunk, 802.1Q tagged, VLAN 10/20/99

### What Was Learned
- A VM router cannot share a physical NIC between untagged host traffic and a tagged VLAN trunk — this is an architecture constraint, not a configuration problem
- Recognizing the difference between a misconfiguration and an architecture constraint saves significant troubleshooting time
- Dedicated management interfaces are standard practice in real networks for exactly this reason — separation of management plane from data plane
- Sometimes the right fix is adding hardware, not changing configuration
