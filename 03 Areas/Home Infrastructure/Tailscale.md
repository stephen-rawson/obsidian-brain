---
type: note
updated: 2026-09-25
---

# Tailscale (remote access)

WireGuard mesh; devices connect outbound, so it works through the double NAT
with **no port forwards**. Replaces the need for Nabu Casa remote access.

- **Tailnet owner:** GitHub identity (hardware-key 2FA). Passkey user added for
  daily use — note: passkeys cannot *create* a tailnet, only join one, and a
  passkey username is globally unique and never reusable.
- Devices: pihole, rayphen-storage, iphone, sr-lenovo.

## Configuration
- **Subnet router = the Pi**, advertising 192.168.1.0/24, route approved,
  key expiry disabled.
- **The NAS must not advertise the LAN route**: it cannot reach its own macvlan
  container (NPM at .250), so remote access to proxied names breaks if clients
  pick the NAS as router.
- **DNS**: MagicDNS on; custom nameserver 192.168.1.31 **restricted to**
  `home.stephenrawson.uk`, so home names resolve on 5G.
- **Pi-hole must listen on ALL interfaces** — queries arrive from 100.x.
- Optional **exit node** on the Pi for travel (Pi-hole everywhere + home IP);
  enable per trip, not permanently.

## Checks when remote access misbehaves
1. `tailscale status --json` on the Pi — its `AllowedIPs` must include
   192.168.1.0/24 and `PrimaryRoutes` must list it.
2. Phone: disconnect/reconnect Tailscale; the Pi entry should show the subnet.
3. `http://192.168.1.31:8080/admin` first (proves routing), then a proxied name.
