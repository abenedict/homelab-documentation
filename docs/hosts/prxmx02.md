# prxmx02 — Proxmox VE host (home)

_Info from command output you pasted on 2026-10-08. Claude has no SSH or API access to this host yet._

## Access

| | |
|---|---|
| IP | 192.168.100.251/24 on `vmbr0` (gateway 192.168.100.1) |
| Web UI / API | `https://192.168.100.251:8006` (reachable from vibe-lab) |
| Version | pve-manager 9.1.5, kernel 6.17.9-1-pve |

## Network

| Bridge | Ports | Address | Use |
|---|---|---|---|
| `vmbr0` | `nic0` | 192.168.100.251/24 | Home LAN |
| `vmbr1` | none | none | _Planned:_ internal network for the `homelab-iac` lab environment |

## Storage

| Name | Type | Notes |
|---|---|---|
| `local` | dir | ~54 GB free. ISOs, templates, cloud images |
| `local-lvm` | lvmthin | ~148 GB, empty. _Planned:_ lab VM disks |
| `storage` | lvm | ~214 GB free |
| `pbs` | Proxmox Backup Server | **95% full** (~0 bytes reported available) |

## Existing guests (not managed by IaC; must not be touched)

| ID | Type | Name |
|---|---|---|
| 101 | VM | `haos-17.1` (Home Assistant OS) |
| 102 | LXC | `cloudflared` |

## Planned for homelab-iac

- Resource pool `lab`, API token `tofu@pve!iac` with rights only on that pool and the storage/bridges it needs.
- Lab VM IDs in the 200–299 range.
- Internal subnet `10.42.0.0/24` (doesn't overlap 192.168.1.0/24 or 192.168.100.0/24).
