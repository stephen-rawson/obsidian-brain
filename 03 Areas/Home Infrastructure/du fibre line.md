---
type: note
updated: 2026-10-03
aliases:
  - du
---

# du fibre line

- **Account:** in own name since 22 Sept 2026 (previously in Raya's name; that
  account cancelled). Account and job numbers → Bitwarden: "du".
- **Plan:** ~250 Mbps. Measured 286 down / 105 up.
- **ONT:** GPON home gateway inside the hallway wall enclosure, 192.168.70.1.
  Wi-Fi disabled. Admin password on the label does not work; du holds it.

## Current state
- **Double NAT**: ONT routes 192.168.70.0/24, Synology WAN gets 192.168.70.2.
- No bridge mode or DMZ yet. Costs: relayed connections for NAT traversal,
  no inbound ports. Tailscale makes this mostly irrelevant.
- TV wall drops now run from the UniFi switch (Sept 2026); see [[Switch & UniFi controller]] for the remaining drops.

## To do
- [ ] Ask du for **bridge mode** (or DMZ to the Synology WAN IP + DHCP reservation)
- [ ] Ask for a goodwill credit / no contract reset after the Sept outage
- [ ] Confirm old account closed with no early-termination fee

## History
Two-week outage Sept 2026 caused by an ID-verification loop on the old account;
resolved by cancelling and reinstalling the service in own name.
