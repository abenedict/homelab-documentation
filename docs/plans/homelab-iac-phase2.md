# homelab-iac — Phase 2: Router

_Started 2026-10-08. Design choices: [decisions/0005](../decisions/0005-homelab-iac-phase2-router.md).
Code: `~/homelab-iac/ansible` (commit `8bd8c04`)._

## Done by Claude (2026-10-08)

- Ansible roles `base`, `router`, `dnsmasq`, `tailscale`. All applied **except `tailscale`**, which needs your auth key.
- OpenTofu: guest agent turned on for both VMs (they rebooted). Proxmox now shows their IP addresses.
- Checked:
  - lab-app1 reaches the internet and resolves `*.lab` names
  - lab-app1 can't reach the home network (192.168.100.1 blocked)
  - DHCP offers 10.42.0.1xx with the right gateway, DNS and search domain (tested with `dhcpcd -T`, nothing changed)
  - Firewall, NAT, dnsmasq and forwarding all came back after the reboot
  - A second playbook run reports `changed=0`, and `tofu plan` reports no changes

## [ ] Your steps: Tailscale

### 1. Create an auth key

Tailscale admin console → **Settings → Keys → Generate auth key**:

- Reusable: **off**. Ephemeral: **off** (a router should stay listed when it reboots).
- Pre-approved: **on**, if your tailnet requires device approval.
- Expiration: 1 day is plenty. It's only used for the first login.
- Tags: none for now.

### 2. Save it on vibe-lab

In your own terminal on vibe-lab (not in this chat), run this and paste the key at the prompt (it isn't echoed):

```bash
( umask 077; read -rsp 'Tailscale auth key: ' k && printf '%s' "$k" > ~/.config/homelab-iac/tailscale-authkey; echo )
```

Then tell Claude: "Tailscale key saved". Claude runs the `tailscale` role, which logs lab-router in.

### 3. In the admin console, after lab-router appears

- **Machines → lab-router → ⋯ → Edit route settings:** approve `10.42.0.0/24`.
- **Machines → lab-router → ⋯ → Disable key expiry** (otherwise the router drops off the tailnet after ~180 days).
- **DNS → Nameservers → Add nameserver → Custom:** `10.42.0.1`, turn on **Restrict to domain**, domain `lab`.
  Then `*.lab` names work from every tailnet device.

On Linux clients only, also run `sudo tailscale set --accept-routes` (other OSes accept routes by default).

## Then (Claude)

1. Check from vibe-lab's view of the tailnet that lab-router advertises the route, and that `lab-router.lab` resolves via split DNS.
2. Delete the auth key file (it's single-use, and logged in is logged in).
3. Mark Phase 2 done, then Phase 3 (Docker, Caddy, `whoami.lab` on lab-app1).
