## pfSense Firewall Rules as ACL Analogs (VLAN 20 → VLAN 10)
 
Wanted to see how pfSense handles inter-VLAN traffic control, similar to ACLs on a physical switch/router.
 
Moved the laptop to Port 11 (VLAN 20, access mode), confirmed it pulled 192.168.20.100 / gateway 192.168.20.1.
 
### Baseline
 
Ping 192.168.10.1 (VLAN10_USERS gateway) from the VLAN 20 client. 10/10 packets, 0% loss. Turns out the existing "Allow VLAN 20 to internet" rule had destination set to any, which doesn't exclude LAN subnets — so it was letting inter-VLAN traffic through even though the rule was only meant to allow internet access.
 
### Block rule
 
Added a block rule on the VLAN20_TEST interface: block, any protocol, source VLAN20_TEST subnets, destination VLAN10_USERS subnets. Placed it above the existing allow rule.
 
Re-tested: 100% packet loss.
 
![Block rule - 100% packet loss]<img width="1940" height="1481" alt="Terminal- block rule in effect" src="https://github.com/user-attachments/assets/03fcdcec-6bb4-4e56-9e47-26d0dccf97b4" />

 
Rule order confirmed working — first match wins.
 
### Permit rule
 
Added a permit rule above the block rule: pass, ICMP, any subtype, same source/destination.
 
Re-tested: 0% loss again, but only for ICMP — everything else stays blocked by the rule below it.
 
![Permit rule - 0% packet loss]<img width="2948" height="2071" alt="Terminal Permit Rule in Effect" src="https://github.com/user-attachments/assets/f2fd8a93-6185-4e38-ace5-746476c6e332" />

 
### Verification in Wireshark
 
Captured live on the laptop's own interface, filtered to `icmp && ip.addr==192.168.10.1` — clean echo request/reply pairs the whole way through, confirming the permit rule takes effect exactly where expected.
 
![Wireshark - ICMP verified]<img width="626" height="329" alt="Wireshark Capture ICMP" src="https://github.com/user-attachments/assets/4c5daff6-5cfd-41f5-8e1a-095246b7ac7d" />






 
---
---
