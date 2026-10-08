# homelab-iac — Phase 1: Provision

_Started and completed 2026-10-08. Design choices: [decisions/0004](../decisions/0004-homelab-iac-phase1-design.md).
Code: `~/homelab-iac` on vibe-lab, `git@github.com:ploopyfloofee/homelab-iac.git`._

## Done by Claude

- Wrote the OpenTofu config in `tofu/`: pinned Debian 13 cloud image, `lab-router` (VM 200), `lab-app1` (VM 201).
- `tofu init`, `tofu validate` and `tofu plan` pass. Plan: **4 to add, 0 to change, 0 to destroy**.
- First commit pushed to GitHub (confirms the deploy key has write access).
- Added `Host lab-router` / `Host lab-app1` to `ssh/config`.

## [x] Your step: put the cloud image on prxmx02

You chose to download it in the Proxmox UI instead of giving the token `Sys.AccessNetwork`. It's at
`local:import/debian-13-genericcloud-amd64-20261001-2618.qcow2`. Its size matches Debian's file exactly (341,508,096 bytes).
The code now looks the image up instead of downloading it.

## [x] Applied and checked (Claude)

- `tofu apply`: 3 added (the two VMs and the console password). Both VMs are running in pool `lab`.
- `lab-router`: SSH works. `eth0` 192.168.100.42/24 (default route via .1), `eth1` 10.42.0.1/24. It reaches the
  internet and pings lab-app1. cloud-init finished.
- `lab-app1`: SSH works through lab-router. 10.42.0.10/24, gateway 10.42.0.1, passwordless sudo, Debian 13.7.
  cloud-init still `running` (its package upgrade has no internet until Phase 2, as expected).
- A second `tofu plan` reports **No changes**, so the code matches what's running.

## Things to try yourself (optional, safe)

```bash
cd ~/homelab-iac/tofu && source ~/.config/homelab-iac/proxmox.env
tofu plan                 # "No changes"
tofu state list           # what OpenTofu tracks
tofu state show proxmox_virtual_environment_vm.router
```

Change `memory { dedicated = 1024 }` to `1536` in `router.tf` and run `tofu plan` to see an in-place change (`~`).
Don't apply it; revert with `git checkout router.tf`.

## Next: Phase 2 (Ansible on lab-router)
