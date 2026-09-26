---
type: note
updated: 2026-09-25
---

# Router — Synology RT6600ax (SRM)

- **Address:** https://192.168.1.1:8001 · also `router.home.stephenrawson.uk`
- Login: `stephen-router: shack-region-overbite`
- SSID: `Rayphen: boobie101`
- **Admin user:** see → Bitwarden: "Synology SRM"
- **Config backup:** `.dss` export in `backups/router/` on the NAS (re-export after changes)

## Roles
- Gateway and firewall for 192.168.1.0/24; WAN is the du box (double NAT).
- DHCP server: pool .20–.199, reservations for NAS/Pi/HA.
- **DNS handed to clients: 192.168.1.31 (Pi-hole).**
- Wi-Fi: see [[Network & IPs]].
- Device tracking feeds Home Assistant (legacy `synology_srm` platform,
  deprecated in HA 2027.5 — presence already runs on Companion app trackers).

## Settings that matter
- UPnP **off** (torrent port comes from the VPN, not UPnP).
- External access / QuickConnect **off**; admin from LAN or Tailscale only.
- 2FA on the admin account.
- Router's own upstream DNS: public resolvers, **not** Pi-hole (avoids a loop).

## Where things live in SRM
Network Center → Local Network → *Network* tab → edit "Network 1" for DHCP
settings; *DHCP Client* lists leases; *DHCP Reservation* adds fixed addresses.
