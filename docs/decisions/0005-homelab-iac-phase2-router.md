# 0005 — homelab-iac Phase 2: how lab-router is configured

**Date:** 2026-10-08
**Status:** Accepted (Claude's choices; open to change)

| Choice | Why | Alternative not taken |
|---|---|---|
| **Ansible connects through `~/claude/ssh/config`** (`ssh_args = -F …`) | One place for SSH addresses, key and known_hosts; ProxyJump to lab-app1 just works | Duplicate addresses and key paths in the inventory |
| **Only `ansible.builtin` modules**, no Galaxy collections | Nothing extra to install or pin; sysctl is a file plus a handler | `ansible.posix.sysctl`, `community.general` |
| **Lab can't open connections to private ranges** (10/8, 172.16/12, 192.168/16, 100.64/10) | A lab is where you break things; it shouldn't be able to reach the home network or tailnet. Replies to connections *into* the lab still work | Let the lab reach home: easier, but a compromised lab VM could reach home devices |
| **Home network can't reach the lab directly**, only lab-router's SSH | Access goes through Tailscale (works anywhere) or SSH via lab-router | A static route on OPNsense: OPNsense is due to be replaced |
| **nftables replaces only its own tables** (`lab_filter`, `lab_nat`), not `flush ruleset` | Tailscale adds its own rules; a full flush on reload would delete them | Full flush: simpler file, but breaks Tailscale until it restarts |
| **dnsmasq on 10.42.0.1 only** (`bind-dynamic`), `no-hosts`, `filter-AAAA` | systemd-resolved keeps 127.0.0.53; the router's `/etc/hosts` maps itself to 127.0.1.1; the lab is IPv4-only | Replace systemd-resolved |
| **DHCP pool .100–.199**, fixed addresses below .100 | lab-router (.1) and lab-app1 (.10) keep their cloud-init addresses | Everything on DHCP reservations |
| **Tailscale: one-time auth key in a file on vibe-lab, used only if not logged in**; settings via `tailscale set` | The key never enters the repo or the state; reruns don't need it | Reusable/tagged key in SOPS: Phase 4 |
| **`--accept-dns=false` on lab-router** | lab-router's own DNS must stay the home resolver; it *serves* the tailnet's `.lab` names | Accept tailnet DNS |
| **Guest agent on** (`agent.enabled = true`) after Ansible installed it | Proxmox sees IPs and can shut VMs down cleanly | Leave it off |

## Consequences

- **Rebuild gap:** on a fresh `tofu apply`, the VMs are created with the agent expected, but the image has no agent
  until Ansible installs it, so OpenTofu waits up to 15 minutes. To fix in Phase 4 (e.g. a shorter agent timeout
  or installing the agent at first boot).
- `.lab` values (subnet, addresses) appear in both `tofu/terraform.tfvars` and `ansible/group_vars/all.yml`. Keep them in step.
