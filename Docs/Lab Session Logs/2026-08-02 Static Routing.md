# Static Routing Lab -> Phantom Subnet via VLAN 20

Single-router topology (pfSense as the only L3 hop) means there's no real second router to point a static route at yet -> that's hardware-blocked until the second switch/hypervisor situation gets sorted. Worked around it by turning a VLAN 20 host into a fake next-hop so I could still run a real verify/break/restore cycle.

## Baseline

Pulled the route table before touching anything (`Diagnostics -> Routes` on pfSense):

```
0.0.0.0          10.0.0.1     UGS   em1
10.0.0.0/24      link#2       U     em1
10.0.0.1         link#2       UHS   em1
10.0.0.4         link#4       UHS   lo0
75.75.75.75      link#2       UHS   em1
75.75.76.76      link#2       UHS   em1
127.0.0.1        link#4       UH    lo0
192.168.10.0/24  link#7       U     em0.10
192.168.10.1     link#4       UHS   lo0
192.168.20.0/24  link#8       U     em0.20
192.168.20.1     link#4       UHS   lo0
192.168.99.0/24  link#1       U     em0
192.168.99.1     link#4       UHS   lo0
```

No static routes present -> clean baseline, all three VLANs directly connected, single default route out WAN.

Noticed VLAN 99 shows up on `em0` (untagged) while VLAN 10/20 show up on `em0.10`/`em0.20` (tagged sub-interfaces). Went and double checked this wasn't a lost config from a `write memory` that didn't save -> it's not, it's intentional. My original trunk config on Port 5 has VLAN 99 explicitly set `untagged` (native VLAN on the trunk) while 10 and 20 are `tagged`. That's by design, not a mistake -> worth being able to explain in an interview: management VLAN riding native/untagged on a single-switch trunk, no untrusted device positioned to double-tag onto it.

Also confirmed while I was in there: Port 4 (used to be a dedicated VLAN 99 mgmt access port) isn't in use anymore -> checked and it's administratively down, consistent with the "disable unused ports" convention I already applied elsewhere on the switch. Only Port 5 (the trunk) is active now.

## Build

- Host: laptop, plugged into Port 11 (VLAN 20 access port per my original trunk config)
- Confirmed IP via dock's Ethernet interface (`en10` this time -> changes depending on what else is plugged in) -> landed on `192.168.20.100`, correct VLAN 20 range
- Enabled IP forwarding on the laptop (macOS uses `net.inet.ip.forwarding`, not the Linux `ip_forward` sysctl):
  ```
  sudo sysctl -w net.inet.ip.forwarding=1
  ```
- Added a phantom subnet as a loopback alias so pfSense would have a real destination to route to:
  ```
  sudo ifconfig lo0 alias 192.168.50.1/24
  ```
- On pfSense: created a new gateway (`VLAN20_GHOSTGW`) on the VLAN20_TEST interface pointing to `192.168.20.100`, then added a static route for `192.168.50.0/24` via that gateway

## Verify

From the iMac (VLAN 99):

```
traceroute 192.168.50.1
1  pfsense.home.arpa (192.168.99.1)  0.921 ms
2  192.168.50.1 (192.168.50.1)  1.175 ms
```

Two clean hops -> pfSense, then straight to the phantom subnet via the laptop as next-hop. Ping came back clean too once I fixed a typo on my end (typed `10.168.50.1` instead of `192.168.50.1` the first time -> got 100% loss on that, not a real result, just me fat-fingering the IP).

## Break

Disabled the static route on pfSense, applied changes, re-tested:

```
ping -c 5 192.168.50.1
100% packet loss
```

Got a bonus lesson out of this one -> the ping didn't just silently fail locally. Since the static route was gone but the default route (`0.0.0.0 -> 10.0.0.1`) still existed, pfSense fell back to sending the traffic out WAN instead of dropping it. Got an actual ICMP response back from `96.120.64.161` (Comcast/Xfinity edge equipment) saying "Communication prohibited by filter" -> the packet genuinely left my network and got rejected upstream since `192.168.50.1` obviously isn't a real routable address.

Good concrete example of route precedence (specific route beats default route) and what happens to traffic that should've stayed internal when its specific route disappears but a default route is still sitting there ready to catch it.

## Restore

Re-enabled the static route, applied changes, re-tested:

```
ping -c 5 192.168.50.1
0% packet loss
```

Recovered clean.

## Cleanup

Reverted the laptop back to normal since it's my daily driver, not a dedicated lab box:

```
sudo sysctl -w net.inet.ip.forwarding=0
sudo ifconfig lo0 -alias 192.168.50.1
```

Confirmed `lo0` back to just `127.0.0.1`/`::1`.

Left the `VLAN20_GHOSTGW` gateway and the static route on pfSense in place but fully disabled -> keeping it as a reference/reusable artifact for next time instead of tearing the whole thing down, since it's not actively monitoring or doing anything while disabled.

## Next up
- OSPF via pfSense FRR package (once static routing concept is solid, which -> today confirmed it is)
