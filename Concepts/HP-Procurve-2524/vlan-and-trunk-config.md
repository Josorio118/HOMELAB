# HP ProCurve 2524 — VLAN & Trunk Configuration

## Firmware
```
show version
# Confirmed: F.01.08
```

---

## VLAN Creation

```
vlan 10
  name USER_LAN
  exit

vlan 20
  name LAB_TEST
  exit

vlan 99
  name MGMT
  exit
```

---

## Access Port Assignments

```
vlan 10
  untagged 10
  exit

vlan 20
  untagged 11
  exit

vlan 99
  untagged 4
  exit
```

---

## Trunk Port Configuration (Port 5 → pfSense)

```
vlan 10
  tagged 5
  exit

vlan 20
  tagged 5
  exit

vlan 99
  untagged 5
  exit
```

> VLAN 99 is untagged on the trunk — it serves as the native/management VLAN on this link. pfSense LAN (em0) receives VLAN 99 traffic untagged.

---

## Remove Ports from Default VLAN 1

```
vlan 1
  no untagged 10,11
  exit
```

---

## Save Configuration

```
write memory
```

---

## Verification Commands

```
show vlan
show vlan 10
show vlan 20
show vlan 99
show interfaces
show mac-address
show spanning-tree
```
