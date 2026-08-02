# SNMP Trap Forwarding, Port Hardening, and Alert Tuning
 
Picked this up to finish the last open item from the LibreNMS monitoring stack build -> SNMP trap forwarding from the ProCurve to LibreNMS, which needed the iMac's VLAN 99 IP to finalize.
 
**What Was Done**
 
- Connected to the ProCurve via console -> hit `Resource busy` from six stacked, detached `sudo screen` sessions left over from earlier troubleshooting. Killed them all with `sudo screen -X -S <name> quit`, confirmed clean with `sudo screen -ls`, reconnected fine.
- Grabbed the iMac's VLAN 99 IP, added the trap receiver:
```
  snmp-server host public <iMac-VLAN99-IP> critical
```
- Verified with `show snmp-server`
- Live-tested by flapping port 11 (VLAN 20 / LAB_TEST) -> confirmed in `show log` and confirmed in LibreNMS's Eventlog for the switch (`ifOperStatus: down -> up`, duplex change, STP topology recalc)
Trap forwarding is now fully live and verified end to end.
 
**Port Hardening**
 
Disabled every unused port to cut down VLAN-hopping surface area:
```
interface 1-4,6-9,12-26 disable
```
First pass errored on port 25 -> ports 25-26 are the SFP transceiver slots, not standard copper, and don't take the same `disable` syntax. Corrected to `1-4,6-9,12-24`, ran `write memory`, confirmed with `show interfaces`. Ports 5 (trunk), 10 (VLAN 10), and 11 (VLAN 20) left enabled on purpose.
 
**LibreNMS Alert Cleanup**
 
The "Port status up/down" rule had been firing since 07/19 across every unused port -> a stale, un-cleared fault list. Switch-side disable had registered correctly in LibreNMS (`Admin: down`), but LibreNMS was still polling/alerting on those ports regardless, since `Ignore alert tag` (Device Settings -> Port Settings) was OFF everywhere. Batch-set `Ignore alert tag: ON` for the disabled ports, left it OFF for ports 10 and 11 since those are active access ports that should still alert if something plugged in drops unexpectedly. The stuck 07/19 alert finally cleared through Discord as a backlog notification once saved.
 
**Cleared an old intrusion alarm**
 
`show interfaces` showed `Intrusion Alert: Yes` on port 10, left over from an earlier intentional port-security lab. Cleared with `clear intrusion`.
 
**Switch Clock Fix**
 
`show log` timestamps were stuck at January 1990 -> `ip timep dhcp` was configured but never resolving (no Timep server via DHCP, and pfSense speaks NTP, not the older Timep protocol the ProCurve expects). Set manually:
```
time 14:44:20 07/25/26
time timezone 240 daylight-time-rule continental-us-and-canada
```
(first tried timezone 300 for standard Eastern, undershot since Eastern's in DST right now -> corrected to 240)
 
**Open items for next session**
 
- Confirm switch clock holds through the DST changeover in November, or reset manually
- Look into whether pfSense can serve as a real Timep source, or accept manual reset as the permanent workaround
- Static routing lab on pfSense (VLAN 20, next-hop, verify/break/restore) -> next lab up
