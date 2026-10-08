# homelab-iac — Phase 0 checklist (your steps)

_Written 2026-10-08. When all four steps are done, tell Claude: "Phase 0 done". Claude will check that the token can
see the `lab` pool and can't see VM 101 or CT 102, then start Phase 1._

**Status 2026-10-08:** steps 1, 2 and 4 verified by Claude. Step 3 is half done: `vmbr1` was created but
**Apply Configuration wasn't run**, so it's still pending on prxmx02. Repo: `git@github.com:ploopyfloofee/homelab-iac.git`.

## Overall plan

| Phase | What gets built | What you learn |
|---|---|---|
| **0. Prep** | Tools on vibe-lab, a scoped Proxmox pool and token, the `vmbr1` bridge, the GitHub repo | Keeping IaC from touching anything outside its scope |
| **1. Provision** | `homelab-iac` repo. OpenTofu creates the router VM (WAN + LAN) and one internal VM from the Debian 13 cloud image | OpenTofu: providers, resources, variables, state, `plan` vs `apply` |
| **2. Router** | Ansible: nftables firewall/NAT, dnsmasq (DHCP + `*.lab` DNS), Tailscale subnet router | Ansible: inventory, roles, templates, idempotency |
| **3. Platform** | Docker, Caddy reverse proxy, a `whoami` test service at `whoami.lab` | Running Compose stacks from Ansible |
| **4. Prove it** | SOPS/age secrets, a README runbook, then destroy everything and rebuild it | That the code really can rebuild the environment |

Design: the router VM's WAN connects to `vmbr0` and gets its address by DHCP. The lab sits behind it on `vmbr1`,
`10.42.0.0/24`. You reach the lab over Tailscale, so it works on any network without port forwarding.

## Already done by Claude (on vibe-lab)

- Installed in `~/.local/bin`: OpenTofu 1.13.1, ansible-core 2.21.5, sops 3.13.3, age 1.3.2, uv.
- Created the GitHub deploy key `ssh/claude_iac_github_ed25519` and the `Host github-iac` entry in `ssh/config`.
- Created `~/.config/homelab-iac/proxmox.env` (mode 600) with a placeholder for the API token.
- Documented [prxmx02](../hosts/prxmx02.md). Changelog entry dated 2026-10-08.

**Choices made:** lab VMs go on `local-lvm`, lab VM IDs are 200–299, and the internal network is `10.42.0.0/24` on `vmbr1`.

**Unrelated, but worth a look:** the `pbs` backup storage is **95% full**, so backups of HAOS and cloudflared may start failing.

## Your steps

### [x] 1. Create the restricted Proxmox account

Run on prxmx02 as root:

```bash
pveum pool add lab --comment "Managed by homelab-iac (OpenTofu)"
pveum role add TofuLab -privs "VM.Allocate VM.Audit VM.Clone VM.Config.CDROM VM.Config.Cloudinit VM.Config.CPU VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Console VM.Migrate VM.PowerMgmt VM.Snapshot VM.GuestAgent.Audit Pool.Audit Datastore.AllocateSpace Datastore.AllocateTemplate Datastore.Audit SDN.Use"
pveum user add tofu@pve --comment "OpenTofu for homelab-iac"
pveum acl modify /pool/lab --users tofu@pve --roles TofuLab
pveum acl modify /storage/local-lvm --users tofu@pve --roles TofuLab
pveum acl modify /storage/local --users tofu@pve --roles TofuLab
pveum acl modify /sdn/zones/localnetwork --users tofu@pve --roles TofuLab
pveum acl modify /nodes/prxmx02 --users tofu@pve --roles PVEAuditor
pveum user token add tofu@pve iac --privsep 0
```

What this does:

- The **pool** is the fence: the account can only manage VMs inside `lab`, so HAOS (101) and cloudflared (102) are out of reach.
- The **storage and SDN grants** let it create disks and attach VMs to the bridges.
- The **node grant** is read-only.
- The **last command prints a secret once.** Copy it.

The privilege list was written from memory of Proxmox 9's names. If `role add` errors on a privilege name, save the
error and give it to Claude.

### [x] 2. Save the token secret

On vibe-lab, edit `~/.config/homelab-iac/proxmox.env` in your editor and replace `PASTE-SECRET-HERE` with the secret.
Don't paste the secret into a chat.

### [ ] 3. Create the internal bridge (created; **still needs Apply Configuration**)

Proxmox UI: **prxmx02 → System → Network → Create → Linux Bridge**

- Name: `vmbr1`
- IPv4, gateway and bridge ports: leave blank
- Comment: `lab internal`

Then click **Apply Configuration**. With no bridge ports, `vmbr1` is a virtual switch with no physical connection.
Only the router VM will connect it to the outside.

### [x] 4. Create the GitHub repo

1. Make a new **private** repo called `homelab-iac`, completely empty (no README or license).
2. Go to **Settings → Deploy keys → Add deploy key**, check **Allow write access**, and paste:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIYU6Qrk78fN9W1860KnuIbSLdVxc3h2D98MOaVtPIbN claude@vibe-lab homelab-iac deploy key
```
