# 2026-01-06 — Unknown Unicast Flooding Study

## Objective
Observe and understand unknown unicast flooding behavior — what triggers it, what it looks like, and how to diagnose it.

## What Was Observed
- All switch port LEDs flashing simultaneously — visible sign of flooding
- Flooding caused network instability — packet loss and ping timeouts to local devices and internet
- Wireshark capture showed repeated Ethernet frames from a specific source MAC
- Correlating that source MAC to the switch MAC address table identified the flooding port

> Unknown unicast flooding is normal switch behavior when the destination MAC is not in the table. Excessive flooding from a single source is a problem.

---

## Troubleshooting Method

1. Observe port LED behavior — all ports lighting simultaneously is abnormal
2. Capture with Wireshark — identify a repeating source MAC
3. Run `show mac-address` on switch — find which port that MAC is learned on
4. Isolate that port — flooding stops

---

## Key Takeaway

Flooding is not always a sign of a broken network — it is how switches discover where devices live. The problem is when flooding is excessive or caused by a single misbehaving source. Wireshark plus the MAC address table is the complete diagnostic toolkit for this scenario.
