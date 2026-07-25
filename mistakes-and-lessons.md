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

### 7/14/26

## 5. Don't trust the first ping after a pfSense reboot
 
Right after boot I pinged 8.8.8.8 and the latency was all over the place — spiked up to 8.8 seconds at one point. My first instinct was that something was wrong with the WAN link.
 
Let it keep running and it settled into ~18ms after about a minute. Ran it again clean (`ping -c 20`) and got 0% loss, stddev under 1ms. So it wasn't the connection — it was just the WAN/DHCP still settling right after boot.
 
Lesson: give it a minute after reboot before judging latency, and always re-test with a bounded ping (`-c N`) instead of eyeballing a live scroll. The first burst after a fresh boot isn't a reliable read on link quality.

## 6. pfSense's packet capture GUI isn't reliable for real-time verification
 
I tried using Diagnostics → Packet Capture in pfSense to verify the block/permit rules — start a capture on the VLAN10_USERS interface, ping from VLAN 20, stop, view. Kept coming back empty even when I knew traffic should be hitting that interface.
 
Turns out the Start/Stop/View workflow in the GUI doesn't make it obvious whether the capture is actually running, or whether "View" is showing a fresh result vs. a stale one from before. Redid it several times with a bounded packet count and it still came back empty.
 
Switched to running Wireshark directly on the iMac, capturing on the USB adapter — the same physical NIC that's bridged into VMware Fusion as pfSense's LAN trunk (Port 5). This worked instantly and showed all VLAN traffic crossing that link in real time, since it's capturing right at the trunk itself rather than going through pfSense's own tooling.
 
Lesson: for quick verification, capture directly on the physical interface carrying the traffic in Wireshark rather than relying on pfSense's built-in packet capture page. It's fine for a controlled scheduled capture, but not great for "watch this happen live" testing.
 
## 7. A rule named "allow to internet" isn't the same as a rule scoped to the internet
 
Found this by accident during the baseline test — the existing "Allow VLAN 20 to internet" rule had destination set to `*` (any), which doesn't distinguish LAN subnets from actual internet-bound traffic. So it was quietly allowing VLAN 20 → VLAN 10 the whole time, even though nobody intended that.
 
Lesson: a rule's description/name doesn't enforce its actual scope. If you want a rule to only apply to internet-bound traffic, the destination needs to explicitly exclude local subnets (or use an alias/negation), not just rely on the label matching intent.

### 7/15/26
## 8. This switch's firmware is too old for LLDP
 
Tried `show lldp config` on the ProCurve and got "Invalid input: lldp" even in config mode, not just a syntax issue, the command doesn't exist at all. Checked `show version` and the firmware is dated Oct 2000 (F.01.08). LLDP (802.1AB) wasn't ratified until 2005, so there's no version of this switch's software that would ever support it.
 
Lesson: check `show version` before assuming a missing command is a syntax problem. Sometimes it's a hard hardware/firmware ceiling, not something you can work around with the right syntax. LLDP goes on the list of things that need the second (newer) switch.
 
## 9. USB dock Ethernet adapters have their own fixed MAC — swapping hosts doesn't change what the switch sees
 
Tried to trigger a port security violation by connecting a different device (phone instead of laptop) to the same dock. The switch kept showing the same authorized MAC no matter what was plugged into the dock's USB-C port.
 
Turns out the dock's Ethernet adapter has its own MAC address baked into its own hardware and it doesn't pass through or relay the connected device's MAC. From the switch's perspective, it's always talking to the dock's adapter, not whatever's plugged into it.
 
Lesson: to genuinely test device-level network identity (MAC spoofing, port security, etc.), you need to either use a device with native Ethernet, or swap the actual adapter itself; not just what's connected to the adapter. Also relatedly: `ifconfig ether` on macOS doesn't work on every adapter; mine (a generic USB dock chipset) rejected every attempt to change its MAC address, with different errors depending on interface state.
 
## 10. "Action: None" on ProCurve port security doesn't mean no enforcement
 
Assumed that with Action set to None, port security wouldn't actually block anything — figured it'd just log or do nothing at all. Tested it by connecting an unauthorized device (different adapter, different MAC) to a locked port.
 
The device got no DHCP lease (fell back to a self-assigned 169.254 address), the switch's MAC table for that port never changed, and `show interfaces` showed Intrusion Alert: Yes for that port specifically. A live Wireshark capture on the trunk showed zero packets from the device — confirming the block happens right at ingress on the port, before the frame is even forwarded anywhere.
 
Lesson: on ProCurve, "Action" controls the response to a violation (port disable, trap, alarm), not whether the violation is actually blocked. The blocking is automatic and happens regardless of Action, as long as learn-mode and address-limit are configured. Don't assume "Action: None" means "no security," it means "no extra response."
### 2026-07-19
## 11
 
### MariaDB won't start in Docker on macOS if you bind-mount the data directory
 
Kept getting `librenms_db` stuck in a crash loop. Logs said:
```
ERROR: The server option 'lower_case_table_names' is configured to use case sensitive
table names but the data directory resides on a case-insensitive file system.
```
 
Root cause: `compose.yml` bind-mounts `./db:/var/lib/mysql`, which puts the DB files on the actual macOS disk through Colima's virtiofs passthrough. APFS (macOS's filesystem) is case-insensitive, but MariaDB's config forces `--lower-case-table-names=0`, which needs case-sensitive storage. Straight up incompatible, no amount of retrying fixed it.
 
Fix: swap the bind mount for a Docker named volume instead, since named volumes live inside Colima's Linux VM (real case-sensitive ext4), not on the host disk.
 
```yaml
    volumes:
      - "librenms_db_data:/var/lib/mysql"
```
and at the bottom of the file:
```yaml
volumes:
  librenms_db_data:
```
 
`docker compose down && docker compose up -d` after that and it came up clean on the first try.
 
## 12
 
### Docker Desktop won't install on macOS 13, Colima is the real fix not a workaround
 
Tried `brew install --cask docker-desktop` on the iMac and got a flat error -> Docker Desktop requires Sonoma (14) or newer, no way around it on macOS 13.7. Went with Colima instead (`brew install docker docker-compose colima` then `colima start`), which spins up a small Linux VM and gives you a real `docker`/`docker compose` CLI. Wasn't sure at first if this was some sketchy workaround, but it's a legit, actively maintained, Homebrew-official tool -> the standard recommendation for exactly this situation.
 
## 13
 
### Switch had no IP on any VLAN this whole time
 
Went to add the ProCurve to LibreNMS and needed its management IP. Ran `show management` on the switch and every single VLAN showed "Disabled" under IP Config -> the switch never actually had a network-reachable address, only console/serial access this entire project. Assigned one on VLAN XX:
```
vlan XXX
ip address 192.168.XXX.XXX 255.255.255.0
```
Lesson: don't assume SNMP/reachability problems are a firewall or community-string issue before confirming the device actually has an IP assigned on the interface you're trying to reach it on.

### 2026-07-25
## 14

Killing a `screen` session by just closing the terminal window instead of a proper `Ctrl-A` `k` / exit can leave the serial port locked at the OS level. On macOS, `sudo screen` sessions are owned by root and won't show up in a plain `screen -ls` -> use `sudo screen -ls` to actually see them. Six of these had stacked up unnoticed across earlier sessions, and one was the actual cause of a `Resource busy` / `ioctl TIOCEXCL failed` error when trying to reconnect to the switch console.

## 15

Ports 25-26 on the ProCurve 2524 are the SFP transceiver slots, not standard copper ports -> they reject the standard `interface disable` syntax used on the other 24 ports. Exclude them from bulk port-range commands (e.g. `interface 1-4,6-9,12-24 disable`, not `1-26`).

## 16

Disabling a port on the switch (`Admin: down`) does not stop LibreNMS from polling and alerting on it. LibreNMS has a separate `Ignore alert tag` toggle per port (Device Settings -> Port Settings) that has to be explicitly set to ON for ports you've intentionally shut down -> otherwise a stale "port down" alert can sit open indefinitely and eventually deliver a delayed Discord notification once the queue reprocesses.

## 17

The ProCurve's `time` command sets the *local* clock, but the switch stores time internally and applies the configured timezone offset on top of whatever you enter. Setting the clock before setting the timezone means the displayed time will shift once the offset is applied afterward -> set `time timezone` first, then set the clock, not the other way around.

## 18

There is no `write memory` equivalent for the ProCurve's clock. `reload` or `boot` resets the time back to the January 1990 default regardless of any other saved config -> the clock has to be manually reset after every reboot/power cycle unless a working Timep source is configured. `ip timep dhcp` alone won't work unless the DHCP server is actually configured to hand out a Timep server address, and pfSense's NTP service isn't the same protocol as the ProCurve's Timep client.
