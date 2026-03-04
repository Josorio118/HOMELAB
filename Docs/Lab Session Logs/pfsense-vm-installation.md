# pfSense CE 2.8.1 — VM Installation (VMware Fusion)

## Objective
Install pfSense CE inside a VMware Fusion VM and establish repeatable WebGUI access. What appeared to be a simple installation turned into a deep lesson in virtualization networking.

---

## The Installation Loop Problem

The pfSense installer repeatedly cycled through the same screens — Connectivity Check, Interface Assignment, LAN/WAN Setup — and pressing Continue looped back every time.

**Root cause:** VMware networking instability, not pfSense configuration errors. The interface mapping was actually correct the entire time.

---

## VMware Network Adapter Experiments

Changing adapter modes while pfSense was running made behavior unpredictable and extended the confusion significantly.

| VMware Mode | What Happened |
|---|---|
| NAT | pfSense shares host IP — WAN gets IP but causes subnet conflicts |
| Bridged (Autodetect) | pfSense bridges to physical NIC — same subnet as ISP router — conflicts |
| Bridged (Wi-Fi) | Puts pfSense on same subnet as home network — wrong |
| Host-only | Isolates pfSense from upstream — no WAN connectivity |
| Private to Mac | Fully isolated virtual network — useful for LAN side only |
| vmnet2 | Created by accident — caused additional confusion |

> Changing VMware network modes mid-installation makes behavior unpredictable. Always confirm network mode before starting the installer.

---

## How the Loop Was Broken

- Stabilized VMware adapter configuration and left it alone
- Installer reached the pfSense Plus subscription screen — key breakthrough
- Selected **Install CE**
- Proceeded through: ZFS filesystem → GPT partition scheme → da0 disk (20GB) → pfSense CE 2.8.1

---

## First Successful Boot Output

```
WAN (wan) -> em0 -> IPv4 DHCP: 192.168.1.x/24
LAN (lan) -> em1 -> IPv4: 192.168.1.1/24

pfSense 2.8.1-RELEASE amd64 — Bootup complete
```

---

## GUI Access Failure — Important Lesson

After install, attempting to access `https://192.168.1.1` from the Mac failed with "Can't connect to the server." This was not a pfSense problem.

**Why it failed:**
- The Mac was on the ISP router's network (192.168.1.x) — on the WAN side of pfSense
- pfSense LAN only exists inside the virtual network
- No network path existed from the Mac to pfSense LAN yet

> Virtual firewalls must be treated like real network appliances. WAN is not LAN. GUI access requires the client to be on the pfSense LAN side.

---

## Key Takeaways

- pfSense installer loops are caused by network instability, not configuration errors
- VMware adapter modes are not interchangeable — pick one and commit before starting the installer
- Understanding traffic flow direction is more important than finding the right setting
- GUI access requires being on the correct side of the firewall
