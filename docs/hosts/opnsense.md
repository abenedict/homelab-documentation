# opnsense (router / firewall)

Documented 2026-10-04 from `/conf/config.xml` and live state, read-only. Nothing on the router was changed.
Raw config is **not** stored in this repo (it contains password hashes, VPN private keys, and backup credentials).

> **Status:** working, but expected to be replaced. Plan (2026-10-04): move to a Ubiquiti router + Wi-Fi, and switch
> ISP from Quantum Fiber to Google Fiber. No timeline yet. This page doubles as the inventory for that migration.

## Summary

| | |
|---|---|
| Role | Home router, firewall, DHCP, DNS resolver, WireGuard endpoint |
| Version | OPNsense 26.1.6 (amd64), FreeBSD 14.3-RELEASE-p10 |
| Hostname | `OPNsense.localdomain` (default name) |
| Hardware | 4× Intel igc NICs (`igc0`–`igc3`), MACs `60:be:b4:07:ed:80`–`83` |
| Timezone | America/Chicago |
| Uptime at doc time | 52 days |
| Web UI | `https://192.168.1.1` |
| SSH | `ssh -F ~/claude/ssh/config opnsense` (user `claude`, LAN only) |

## Access

- User `claude` (uid 2000), member of `admins` (privilege `page-all`), shell `/bin/sh`, key login with
  `ssh/claude_homelab_ed25519`. Password is scrambled (unused). **Effectively full admin**: can read
  `config.xml` (group `wheel`). `sudo` requires a password, so no root shell.
- Created by you in the web UI on 2026-10-04. Claude only runs read-only commands unless you approve a change.
- sshd: listens on LAN only, `AllowGroups wheel`, root login **permitted**, password login **enabled**
  (you log in as root with a password from 192.168.1.20).
- Host key: ED25519 `SHA256:9Dr9J1aiPywaX0OKE5L2HzgLPf8H2rrCX3JVduJT6u0` (recorded on first connect; not yet
  verified against the console).

## Interfaces

| OPNsense name | Description | Device | Address | Notes |
|---|---|---|---|---|
| `wan` | WAN | `vlan02` = VLAN 201 on `igc0` | DHCP from ISP | VLAN 201 labelled "Quantum" (ISP tagging) |
| `lan` | LAN_ETH1 | `igc1` | 192.168.1.1/24 | 2.5 GbE link. IPv6: track WAN (prefix id 0) |
| `opt1` | VLAN100ETH2 | `igc2` (untagged) | 192.168.100.1/24 | Server network; vibe-lab is here (192.168.100.252) |
| `opt2` | VLAN200ETH3 | `igc3` | 192.168.200.1**/32** | No cable connected. /32 leaves no usable subnet |
| `opt3` | WGLAN | `wg0` | 10.0.30.2/30 | WireGuard tunnel (see below) |

VLAN 100 (`vlan01`, tag 100 on `igc2`) is defined but **not assigned** to any interface. `opt1` uses `igc2` untagged
despite its name.

## Routing and gateways

- Default route: WAN DHCP gateway.
- `WG_GW`: 10.0.30.1 via `opt3` (WireGuard), not default, monitoring disabled.
- No static routes.

## WireGuard (leftover — ignore for now)

Left over from an abandoned attempt to route the whole network through a VPN on an old Oracle server. Not in use.
Possible cleanup later.

- Instance `wg0`: listen port 51820, tunnel address 10.0.30.2/30, MTU 1420.
- One peer, `oracle`: endpoint `129.159.100.12:51820`, allowed IPs `10.0.30.0/30, 192.168.100.0/24`,
  keepalive 25 s. Keys are in `config.xml` (not copied here).
- The endpoint is the old Oracle server, **not** oracle1 (129.80.178.38).
- No WAN firewall rule allows inbound UDP 51820, so the tunnel only works as an outbound connection from the router.

## DHCP (Kea — active)

ISC DHCP (`os-isc-dhcp` plugin) is installed but disabled on all interfaces. Kea DHCPv4 serves `lan` and `opt1`.

| Subnet | Pool | Router | DNS handed out |
|---|---|---|---|
| 192.168.1.0/24 (LAN-DHCP) | .21 – .139 | 192.168.1.1 | 192.168.1.1 |
| 192.168.100.0/24 (VLAN100-Server-DHCP) | .200 – .250 | 192.168.100.1 | 192.168.1.1 |

Reservations:

| IP | Hostname | MAC |
|---|---|---|
| 192.168.1.5 | Office_deco-XE75Pro | 48:22:54:5a:73:4b |
| 192.168.1.7 | LivingRoom_deco-XE75Pro | 48:22:54:5a:7a:df |
| 192.168.1.9 | Bedroom_deco-XE75Pro | 48:22:54:5a:6d:0b |
| 192.168.1.13 | console-stereo | b8:27:eb:8a:1e:5e |
| 192.168.1.19 | nintendo-switch | bc:74:4b:b6:46:c5 |
| 192.168.1.20 | alan-mbp | 76:22:91:e3:3d:b9 |
| 192.168.1.127 | esphome-upstairs | c8:f0:9e:f1:b4:dc |
| 192.168.100.2 | plex | bc:24:11:da:b6:e5 |
| 192.168.100.3 | apps | bc:24:11:f9:af:49 |
| 192.168.100.4 | i2p-router | bc:24:11:2f:0e:f7 |
| 192.168.100.249 | homeassistant | 02:aa:15:30:14:58 |

The old (inactive) ISC config still has static mappings that never moved to Kea: truenas .1.3, windows10 .1.4,
proxmox .1.6, garage-door .1.8, bastion .1.10, ipmi .1.11, apps .1.12, talos .1.254, plus PXE boot (netboot.xyz from
192.168.1.28). These are **not in effect**. Those devices are either statically addressed or getting pool addresses.

## DNS

- Unbound resolver enabled on port 53, forwarding mode, to the system DNS servers **1.1.1.2 / 1.0.0.2** (Cloudflare
  malware-blocking).
- Registers DHCP lease hostnames; no host overrides, no DNS blocklist, no DNS-over-TLS.
- Dnsmasq disabled.

## Firewall

Aliases:

| Alias | Value | Note |
|---|---|---|
| `plex` | 192.168.100.3 | ⚠ Kea reserves .100.3 for `apps`; `plex` is .100.2 |
| `apps` | 192.168.100.3 | |
| `truenas` | 192.168.100.2 | ⚠ Kea reserves .100.2 for `plex` |
| `test_site` | 192.168.1.68 | Unused in rules |

Port forwards (WAN, IPv4, TCP):

| Port | To | Status |
|---|---|---|
| 80 | `apps`:80 | Active ("apps http") |
| 443 | `apps`:443 | Active ("apps https", pure NAT reflection) |
| 32400 | `plex`:32400 | Active ("Plex"). Resolves to 192.168.100.3, the same host as `apps` |
| 80, 443 | `apps` | Disabled duplicates ("Traefik HTTP/HTTPS") |

Outbound NAT: hybrid. NAT reflection disabled globally (enabled per-rule on 443).

Filter rules, in evaluation order (first match wins):

| # | Interface | Rule |
|---|---|---|
| 1–3 | WAN | Allow TCP to `plex`:32400, `apps`:80, `apps`:443 (linked to port forwards) |
| 4–5 | WAN | Disabled Traefik duplicates |
| 6–7 | LAN | Allow LAN net → any (IPv4 and IPv6) |
| 8–9 | opt1 | Allow opt1 net → any (IPv4 and IPv6) |
| 10 | opt1 | Allow any → any **via `WG_GW`** ("Route server VLAN through WireGuard") |
| 11 | WireGuard group | Allow all inbound |

Rule 10 is part of the WireGuard leftover. It never matches, because rule 8 above it already allows all IPv4 from opt1,
so the server VLAN goes out the normal WAN (checked 2026-10-04: vibe-lab's public IP is the router's WAN address).
Harmless as is.

No inter-VLAN isolation: LAN and opt1 can reach each other freely.

## Other services

| Service | State |
|---|---|
| Netdata | Enabled, 127.0.0.1:19999 (local only) |
| Hostwatch | Enabled |
| Intrusion detection (Suricata) | Disabled |
| Web proxy, captive portal, traffic shaper, IPsec, OpenVPN | Not configured / disabled |
| SNMP | Not configured |
| Remote syslog | None |

Plugins installed: `os-cpu-microcode-intel`, `os-dmidecode`, `os-gdrive-backup`, `os-isc-dhcp`.

## Backups

- Local: 30 config revisions kept on the router.
- Google Drive (`os-gdrive-backup`): enabled, keeps 60 copies, encrypted with a password stored in `config.xml`
  (System → Configuration → Backups). The Google account and service key are also there.

## Open questions

Deferred: WireGuard leftovers (`wg0`, peer `oracle`, `WG_GW`, filter rules 10–11), to clean up later or drop with
the router replacement.

1. `plex`/`truenas` aliases vs the Kea reservations. Which IP is Plex on now?
2. Should the inactive ISC static mappings be moved to Kea or removed?
3. Is `opt2` (VLAN200, /32, unplugged) still wanted?
