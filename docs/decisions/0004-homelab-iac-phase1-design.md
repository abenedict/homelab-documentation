# 0004 — homelab-iac Phase 1 design choices

**Date:** 2026-10-08
**Status:** Accepted (Claude's choices; open to change)

Repo: `git@github.com:ploopyfloofee/homelab-iac.git` (private), checked out on vibe-lab at `~/homelab-iac`.

## Decisions

| Choice | Why | Alternative not taken |
|---|---|---|
| **Separate repo** from this docs repo | The lab's code should rebuild the lab without needing anything else. This repo records why and what changed | Nest it in `~/claude`: mixes two histories |
| **Router WAN static, 192.168.100.42/24** (was "DHCP" in the Phase 0 plan) | The cloud image has no guest agent, so with DHCP nothing (Proxmox, OpenTofu, Ansible) can find the router's address. A fixed address also suits a router. `.42` is outside OPNsense's pool (.200–.250), unreserved, and nothing answered on it on 2026-10-08 | DHCP plus a Kea reservation on OPNsense: ties the lab to a router that's due to be replaced |
| **Debian 13 genericcloud image, pinned** to release `20261001-2618`, SHA-512 checked | A rebuild gets the same image. Changing it is a one-line, reviewable edit | `latest`: rebuilds drift silently |
| **Image downloaded by hand** in the Proxmox UI (Import → Download from URL, SHA-512 checked); OpenTofu looks it up with `data "proxmox_file"` | Your choice on 2026-10-08. The token keeps no extra privilege, and `tofu destroy` never deletes the image | Let OpenTofu download it (`proxmox_download_file`): fully automatic rebuilds, but the token needs `Sys.AccessNetwork` on `/nodes/prxmx02` |
| **Guest agent off** for now | The image doesn't include it, and with the agent on but not running, Proxmox operations hang. Phase 2 installs it | Custom cloud-init snippet: needs SSH to prxmx02, which breaks the API-only fence |
| **Static lab addresses** (router .1, app1 .10) | No DHCP on `vmbr1` until Phase 2 | — |
| **Key-only SSH, user `debian`, plus a generated console password** | The console is the way back in after a firewall mistake | No password: a lockout means rebuilding |
| **Local state file**, gitignored | One operator, one machine. State holds the console password | Remote/encrypted state: planned for Phase 4 |
| **Two plain resources, no module** | Easier to read while learning. Refactor into a module once a third VM appears | Module from the start |

## Consequences

- Rebuilding on a fresh Proxmox needs the image downloaded by hand first (instructions in `tofu/image.tf`).
  To automate it later: grant `Sys.AccessNetwork` on `/nodes/prxmx02` and switch back to `proxmox_download_file`.
- If OPNsense ever hands out `.42` (pool change), the router's WAN address conflicts. Keep `.42` out of any pool.
- On Proxmox 9.1 the token can't turn off cloud-init's first-boot package upgrade (root-only setting), so
  `lab-app1`'s first boot tries to upgrade and fails until the router routes in Phase 2. Harmless.
