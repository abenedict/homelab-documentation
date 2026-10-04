# oracle1 — Oracle Cloud free tier VM

_Info gathered 2026-10-04 (fresh install, fully updated)._

## Access

| | |
|---|---|
| SSH alias | `oracle1` → `ssh -F ~/claude/ssh/config oracle1` |
| Public IP | 129.80.178.38 |
| Private IP | 10.0.0.206/24 (gateway 10.0.0.1), interface `enp0s6` |
| User | `ubuntu` (passwordless sudo) |
| Host key | ED25519 `SHA256:HWAWRVAW3+rtygXUstKZ0C/bgKVswCpW0qh2nyzpMhI` |
| Tailscale | `oracle1` / 100.76.43.37 / `oracle1.tail80394.ts.net` |
| Authorized keys | `alansshkey2022` (my personal key), `claude-code@vibe-lab` |

## Oracle Cloud details

| | |
|---|---|
| Instance name | `instance-20261004-1131` (also the hostname) |
| Shape | VM.Standard.A1.Flex (Ampere ARM): **1 OCPU, 6 GB RAM**, 1 Gbps |
| Region / AD | us-ashburn-1 (iad), AD-1, FAULT-DOMAIN-1 |
| Boot volume | 50 GB |

Always Free allows a total of 4 OCPU / 24 GB RAM / 200 GB block storage across A1 instances, so this
VM can be resized up without leaving the free tier.

## System

- **OS:** Ubuntu 26.04.1 LTS, kernel `7.0.0-1013-oracle`, **aarch64 (ARM64)**. Container images and
  binaries must be built for arm64.
- **Disk:** `/` ext4 48 GB (4% used), `/boot` 891 MB, `/boot/efi` 98 MB.
- **Swap:** none.
- **Time:** UTC, chrony synced.
- **Updates:** `unattended-upgrades` enabled (security updates), automatic reboot **off**.
- **Docker:** Docker CE 29.8.2 + Compose v5.6.0 from Docker's official apt repo (installed 2026-10-04). `ubuntu` is in the `docker` group. Log rotation set in `/etc/docker/daemon.json` (10 MB × 3 files per container).

## Users

- `ubuntu` (uid 1001): main login, sudo.
- `opc` (uid 1000): Oracle's default account, password locked, no login keys. Used by the Oracle Cloud
  Agent. Leave it alone.

## Network / firewall

Two firewall layers. **A new service needs BOTH opened:**
1. **OCI Security List / NSG** (Oracle web console → VCN → Security Lists).
2. **Host iptables.** Oracle's Ubuntu images ship iptables rules, not ufw. Saved in
   `/etc/iptables/rules.v4` and `rules.v6` and reloaded at boot by `netfilter-persistent`.
   Current INPUT chain: allow established, ICMP, loopback, TCP 22; reject everything else.
   **Don't enable ufw on top of this** without replacing these rules.

**Docker caveat:** a container port published as `-p 8080:8080` skips the host iptables INPUT
rules (Docker adds its own rules), so only Oracle's firewall would protect it. Publish ports bound
to localhost (`127.0.0.1:8080:8080`) unless a service really is meant to be public.

Listening ports: `22/tcp` sshd (public), `8080` code-server (127.0.0.1 only), Tailscale (tailnet only), `111` rpcbind (blocked by iptables), `53` systemd-resolved (localhost only).

## SSH server

`PasswordAuthentication no`, `KbdInteractiveAuthentication no`, `PermitRootLogin prohibit-password`, port 22.

## Running services (stock)

chrony, iscsid, rpcbind, ssh, snapd, unattended-upgrades, ModemManager, udisks2,
tailscaled, docker, containerd, Oracle Cloud Agent (snap `oracle-cloud-agent`, held). Other snaps: `core18`, `snapd`.

## Docker stacks

| Stack | Location on server | Source in this repo | Access |
|---|---|---|---|
| code-server | `~/stacks/code-server/` | `configs/oracle1/code-server/compose.yaml` | https://oracle1.tail80394.ts.net (tailnet only) |

Docs working copy: `~/homelab-docs` (git). Remote `github` (the main copy) using deploy key `~/stacks/code-server/data/config/ssh/github_ed25519` (inside the container: `/home/coder/.config/ssh/`). vibe-lab also pushes here directly. See decision 0003.

## Role

- Hosts the browser-accessible homelab docs (code-server). See decision 0002.
