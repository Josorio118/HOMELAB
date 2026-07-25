## 2026-05-12
**Type:** Troubleshooting — WAN connectivity loss
**Checked:**
- pfSense WAN DHCP logs — repeated lease failures, DUID recreating from scratch
- WAN interface status — no handshake completing

**Finding:** Fault traced upstream of pfSense. Lab config ruled out —
all VLANs and interfaces healthy throughout.

**Outcome:** Upstream ISP issue confirmed. No lab changes needed.

## Maintenance Log — 2026-07-14
 
### Lab restoration
 
Moved the lab to new location and had to bring everything back up from scratch. ISP is Xfinity now instead of Verizon FiOS.
 
Recabled:
- iMac built-in NIC → Xfinity router (WAN)
- USB Ethernet adapter → ProCurve Port 5 (trunk)
Checked VMware Fusion bridging was still pointing em1 to the built-in NIC and em0 to the USB adapter, then booted pfSense.
 
Console came up clean on first boot:
- WAN (em1): DHCP from ISP, 10.0.0.4/24
- LAN (em0): 192.168.99.1/24
- VLAN10_USERS: 192.168.10.1/24
- VLAN20_TEST: 192.168.20.1/24
Port 5 link light came up, so the trunk was tagging correctly. Confirmed with two separate DHCP leases:
- iMac on Port 4 → 192.168.99.100 (mgmt)
- MacBook on Port 10 → 192.168.10.100 (users)
Pinged 8.8.8.8 from the VLAN 10 client to confirm WAN was actually routing, not just interfaces being up. Full restoration confirmed end to end.
 

## Maintenance Log — 2026-07-14 (session 2)
 
### pfSense firewall rules as ACL analogs (VLAN 20 → VLAN 10)
 
Wanted to see how pfSense handles inter-VLAN traffic control, similar to ACLs on a physical switch/router.
 
Moved the laptop to Port 11 (VLAN 20, access mode), confirmed it pulled 192.168.20.100 / gateway 192.168.20.1.
 
**Baseline:** ping'd 192.168.10.1 (VLAN10_USERS gateway) from the VLAN 20 client. 10/10 packets, 0% loss. Turns out the existing "Allow VLAN 20 to internet" rule had destination set to any, which doesn't exclude LAN subnets — so it was letting inter-VLAN traffic through even though the rule was only meant to allow internet access.
 
**Added a block rule** on the VLAN20_TEST interface: block, any protocol, source VLAN20_TEST subnets, destination VLAN10_USERS subnets. Placed it above the existing allow rule. Re-tested: 100% packet loss. Rule order confirmed working — first match wins.
 
**Added a permit rule above the block rule**: pass, ICMP, any subtype, same source/destination. Re-tested: 0% loss again, but only for ICMP — everything else stays blocked by the rule below it.
 
**Verified in Wireshark**, captured live on the laptop's own interface, filtered to `icmp && ip.addr==192.168.10.1` — clean echo request/reply pairs the whole way through, confirming the permit rule takes effect exactly where expected.

## 2026-07-25 
 
- SNMP trap receiver added on ProCurve (`snmp-server host public <iMac-VLAN99-IP> critical`) -> verified live via port 11 flap test
- Ports 1-4, 6-9, 12-24 administratively disabled on ProCurve; ports 5, 10, 11 left active; `write memory` saved
- LibreNMS: `Ignore alert tag` set ON for all disabled ports under the ProCurve device; left OFF for ports 10 and 11
- Cleared intrusion alarm on port 10 (`clear intrusion`)
- Switch clock manually set (`time 14:44:20 07/25/26`), timezone set to 240 min / continental-us-and-canada DST rule -> note: will reset to default on next reboot, needs manual reset until a real time source is configured
