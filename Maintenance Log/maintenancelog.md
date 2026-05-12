## 2026-05-12
**Type:** Troubleshooting — WAN connectivity loss
**Checked:**
- pfSense WAN DHCP logs — repeated lease failures, DUID recreating from scratch
- WAN interface status — no handshake completing

**Finding:** Fault traced upstream of pfSense. Lab config ruled out —
all VLANs and interfaces healthy throughout.

**Outcome:** Upstream ISP issue confirmed. No lab changes needed.
