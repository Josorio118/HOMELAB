## Port Security on ProCurve (Learn Mode + Address Limit)
 
Wanted to test port security behavior on the ProCurve — specifically what happens when an unauthorized device tries to connect to a locked-down port.
 
First tried `show lldp config` for a different exercise and hit a wall — "Invalid input: lldp." Checked `show version` and found this switch is running firmware from 2000 (F.01.08), well before 802.1AB LLDP existed as a standard. That exercise is on hold until I get a newer second switch.
 
### Baseline
 
```
show port-security
```
All ports showed learn-mode: continuous, action: none — factory default.
<img width="3158" height="1813" alt="Port Security Table" src="https://github.com/user-attachments/assets/f0f0a7ad-9f9f-43dd-bae9-6ceaf5f2db8c" />

 
### Locking down Port 10
 
```
configure
port-security 10 learn-mode static address-limit 1
```
 
Verified:
```
show port-security 10
```
Port 10 — Learn Mode: Static, Address Limit: 1, Action: None, Authorized Addresses: xx:xxxx-xxxxxx (my laptop, connected via the dock on Port 10).

 
### First attempt at a violation test — dead end
 
Tried spoofing the laptop's MAC via `ifconfig` on macOS to simulate an unauthorized device. Kept hitting errors (`Network is down`, `Can't assign requested address`).
 
Tried swapping devices on the same dock instead (laptop → phone). The switch still only ever saw the same authorized MAC the whole time. Swapping hosts on the same dock never actually changes what the switch sees.
 
### Actual violation test
 
Swapped in a completely different physical adapter (Belkin USB-C LAN) connected to my phone, plugged into Port 10.
 
Result: phone got a self-assigned `169.254.x.x` address — no DHCP lease. Checked the switch:
 
```
show mac-address 10
```
Still only showed the original authorized MAC. The new adapter's MAC never got learned at all.
 
```
show interfaces
```
Port 10 showed **Intrusion Alert: Yes** — the only port flagged out of all of them.<img width="3756" height="1973" alt="Instrusion" src="https://github.com/user-attachments/assets/b7320fbc-1271-4fc6-891c-93a5177033b5" />



 
Ran a live Wireshark capture on the trunk (iMac's USB adapter) during this test, filtered to `bootp`. Zero packets from the unauthorized device showed up — not even a DHCP Discover. That confirmed the block happens at ingress on Port 10 itself, before the frame is ever forwarded across the trunk.
 
### Result
 
Port security worked as real Layer 2 access control: unauthorized MAC → no connectivity, no MAC table entry, flagged as an intrusion — all despite Action being set to None. Action only affects the notification/response layer (port disable, trap, etc.), not whether the block itself happens.
 
---
---
