# 2026-01-05 — MAC Aging & Port Behavior Verification

## Objective
Verify MAC aging behavior and switch learning in a stable state after the loop incident from the previous session.

## What Was Done
- Logged into switch and verified active interfaces with `show interfaces`
- Connected laptop to dock on Port 10
- Confirmed MAC was learned on Port 10 using `show mac-addresses`
- Unplugged laptop from dock
- Observed MAC entry on Port 10 age out — port showed as down

---

## Key Observation

> MAC entries are not permanent. Each entry has an aging timer (~300 seconds). If a device stops sending traffic, the entry is removed.

---

## Why This Matters

This directly maps to two real-world troubleshooting scenarios:

1. **Why unplugging and replugging sometimes fixes connectivity issues** — forces the switch to re-learn the MAC on the correct port
2. **Why a device moving to a new port causes temporary flooding** — the old MAC entry still exists until it ages out, so the switch briefly floods until the table updates
