# pfSense CE 2.8.1 — Firewall Rules

## How pfSense Processes Rules

- Rules are processed **top-down** — first match wins
- pfSense is **default-deny** — any traffic not explicitly permitted is blocked
- Rules are applied **per interface** on inbound traffic
- NAT on WAN handles all outbound traffic automatically

---

## WAN Rules

No inbound rules configured. All unsolicited inbound traffic from the internet is blocked by default.

---

## VLAN 99 — MGMT (LAN)

No explicit block rules. Management VLAN has full access by default. pfSense WebGUI accessible at `https://192.168.99.1`.

---

## VLAN 10 — USERS

Rules applied in order (first match wins):

| Priority | Action | Source | Destination | Protocol | Purpose |
|---|---|---|---|---|---|
| 1 | **BLOCK** | VLAN 10 subnets | 192.168.99.0/24 | Any | Block access to management VLAN |
| 2 | **PASS** | VLAN 10 subnets | Any | IPv4 | Allow internet access |

---

## VLAN 20 — TEST

Rules applied in order (first match wins):

| Priority | Action | Source | Destination | Protocol | Purpose |
|---|---|---|---|---|---|
| 1 | **BLOCK** | VLAN 20 subnets | LAN subnets | Any | Block access to management VLAN |
| 2 | **PASS** | VLAN 20 subnets | Any | IPv4 | Allow internet access |

---

## Rule Design Notes

- The **block rule must always be above the allow rule**. If the allow rule were first, traffic to the management VLAN would be permitted before the block rule is ever evaluated.
- VLAN 10 and VLAN 20 are also isolated from each other by default — no explicit inter-VLAN allow rules exist between them.
- pfSense default-deny means anything not listed is blocked. Rules must be explicit.

---

## NAT

Outbound NAT is configured on the WAN interface. All internal VLAN traffic is NATed through the ISP-assigned WAN IP before leaving the network.
