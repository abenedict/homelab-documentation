# homelab-iac — Phase 1: Provision

_Started 2026-10-08. Design choices: [decisions/0004](../decisions/0004-homelab-iac-phase1-design.md).
Code: `~/homelab-iac` on vibe-lab, `git@github.com:ploopyfloofee/homelab-iac.git`._

## Done by Claude

- Wrote the OpenTofu config in `tofu/`: pinned Debian 13 cloud image, `lab-router` (VM 200), `lab-app1` (VM 201).
- `tofu init`, `tofu validate` and `tofu plan` pass. Plan: **4 to add, 0 to change, 0 to destroy**.
- First commit pushed to GitHub (confirms the deploy key has write access).
- Added `Host lab-router` / `Host lab-app1` to `ssh/config`.

## [ ] Your step: allow the token to download the image

Proxmox's download-url API requires `Sys.AccessNetwork` on the node. The token doesn't have it, so `tofu apply`
would fail on the image download. Run on prxmx02 as root:

```bash
pveum role add TofuDownload -privs "Sys.AccessNetwork"
pveum acl modify /nodes/prxmx02 --users tofu@pve --roles TofuDownload
```

This lets the token make prxmx02 fetch a URL into storage, and nothing more. Undo:
`pveum acl delete /nodes/prxmx02 --users tofu@pve --roles TofuDownload` then `pveum role delete TofuDownload`.

Then tell Claude "download permission added".

## Then (Claude)

1. Check the token has `Sys.AccessNetwork`, then `tofu apply`.
2. Check: both VMs in pool `lab`, SSH to `lab-router` (WAN and LAN addresses up), SSH to `lab-app1` through it.
3. `tofu plan` again shows **no changes** (proves the code matches what was built).
4. Record it in the changelog and host docs.
