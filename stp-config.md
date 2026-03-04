# HP ProCurve 2524 — STP Configuration

## Enable STP

```
spanning-tree
```

> STP is disabled by default on the HP ProCurve 2524. Must be explicitly enabled.

---

## Set Root Bridge Priority

```
spanning-tree priority 4096
```

> Priority must be set in increments of 4096. Setting to 4096 ensures this switch wins the root election against any device using the default priority of 32768.

---

## Save Configuration

```
write memory
```

---

## Verify Root Bridge Status

```
show spanning-tree
```

**Expected output after taking root:**
```
Switch Priority  : 4096
Root Path Cost   : 0
Root Port        : This switch is root
Root Priority    : 4096
Hello Time: 2  Max Age: 20  Forward Delay: 15
```

---

## Notes

- Before setting priority, the ISP router was winning the root election because it was sending BPDUs with a lower bridge ID
- After setting priority to 4096, this switch became root and STP timers normalized to 802.1D defaults
- A loop test was performed by connecting a cable between two unused ports — STP correctly placed the higher-numbered port into Blocking state
- Wireshark confirmed BPDUs advertising Root Priority 4096 from this switch's MAC address
