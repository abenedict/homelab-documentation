# prxmx02 — Proxmox VE host (home)

_Info from command output you pasted on 2026-10-08, plus API checks by Claude on 2026-10-08. Claude has no SSH access.
Claude's only access is the scoped API token below._

## Access

| | |
|---|---|
| IP | 192.168.100.251/24 on `vmbr0` (gateway 192.168.100.1) |
| Web UI / API | `https://192.168.100.251:8006` (reachable from vibe-lab) |
| Version | pve-manager 9.1.5, kernel 6.17.9-1-pve |
| Claude's access | API token `tofu@pve!iac`, secret in `~/.config/homelab-iac/proxmox.env` on vibe-lab (mode 600). Rights listed below |

## Network

| Bridge | Ports | Address | Use |
|---|---|---|---|
| `vmbr0` | `nic0` | 192.168.100.251/24 | Home LAN |
| `vmbr1` | none | none | `homelab-iac` lab network (`10.42.0.0/24`, addressed by the router VM). Comment `lab internal`. Applied 2026-10-08 |

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

## homelab-iac guests (created 2026-10-08, pool `lab`)

| ID | Name | Addresses | Size |
|---|---|---|---|
| 200 | `lab-router` | WAN 192.168.100.42/24 (`vmbr0`), LAN 10.42.0.1/24 (`vmbr1`) | 1 vCPU, 1 GB, 8 GB disk |
| 201 | `lab-app1` | 10.42.0.10/24 (`vmbr1`) | 2 vCPU, 2 GB, 20 GB disk |

lab-router is the lab's gateway, firewall, DNS (`*.lab`) and DHCP server (Phase 2). Guest agent on for both.

SSH: `ssh -F ~/claude/ssh/config lab-router` / `lab-app1`. Console password: `tofu output -raw console_password` in `~/homelab-iac/tofu`.

## homelab-iac access (set up 2026-10-08)

- Resource pool `lab` (empty so far). Repo: `git@github-iac:ploopyfloofee/homelab-iac.git` (private).
- User `tofu@pve`, token `tofu@pve!iac` (`--privsep 0`, so it has the user's rights). Custom role `TofuLab` on
  `/pool/lab`, `/storage/local`, `/storage/local-lvm` and `/sdn/zones/localnetwork`. Built-in `PVEAuditor` on `/nodes/prxmx02`.
  Exact commands: [plans/homelab-iac-phase0.md](../plans/homelab-iac-phase0.md), step 1.
- Checked with the token: it sees only the `lab` pool, lists no VMs, gets "Permission check failed" on VM 101 and CT 102,
  and sees only `local` and `local-lvm` storage (not `storage` or `pbs`).
- Lab VM IDs in the 200–299 range. Managed only by OpenTofu from `~/homelab-iac`; don't edit them in the UI.
- Debian 13 cloud image `local:import/debian-13-genericcloud-amd64-20261001-2618.qcow2` (downloaded by you in the UI).
  Not owned by OpenTofu.
- Internal subnet `10.42.0.0/24` (doesn't overlap 192.168.1.0/24 or 192.168.100.0/24).
