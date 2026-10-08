# Changelog

Newest first. Format: date — host(s) — what — why — how to undo.

## 2026-10-08 — homelab-iac Phase 1 code written (not applied yet)

- **prxmx02** (you): applied the pending `vmbr1` bridge. Claude confirmed it's active and nothing is pending.
- **vibe-lab**: Created `~/homelab-iac` (git, remote `origin` = `git@github-iac:ploopyfloofee/homelab-iac.git`,
  `core.sshCommand` uses `ssh/config`). Wrote the OpenTofu config, ran `tofu init`/`validate`/`plan`, pushed
  the first commit. Nothing created on prxmx02 yet.
- **vibe-lab**: Added `Host lab-router` (192.168.100.42) and `Host lab-app1` (10.42.0.10 via lab-router) to
  `ssh/config`, with their own known_hosts file `ssh/known_hosts_lab`.
  Undo: delete those two blocks.
- Read-only checks: pinged 192.168.100.40/.42/.50/.60 from vibe-lab and read OPNsense's ARP table to confirm .42 is free.
- Why: Phase 1 of `homelab-iac`. Choices in [decisions/0004](decisions/0004-homelab-iac-phase1-design.md);
  next step in [plans/homelab-iac-phase1.md](plans/homelab-iac-phase1.md).

## 2026-10-08 — homelab-iac Phase 0 set up and checked

- **prxmx02** (you): created pool `lab`, role `TofuLab`, user `tofu@pve` and token `tofu@pve!iac` with the ACLs in
  [plans/homelab-iac-phase0.md](plans/homelab-iac-phase0.md) step 1. Created bridge `vmbr1` in the UI, but the change is
  still pending (not applied).
  Undo: `pveum user delete tofu@pve` (removes its token and ACLs), `pveum role delete TofuLab`, `pveum pool delete lab`;
  in the UI, select `vmbr1` → Remove, or **Revert** while it's still pending.
- **vibe-lab** (you): put the token secret in `~/.config/homelab-iac/proxmox.env`.
- **GitHub** (you): created private repo `ploopyfloofee/homelab-iac` (empty) and added Claude's deploy key with write access.
- **vibe-lab**: Claude checked the token with read-only API calls: it sees only pool `lab`, lists no VMs, is denied on
  VM 101 and CT 102, and sees only `local` and `local-lvm` storage. The deploy key authenticates to the repo.
  Fixed the repo owner in the `ssh/config` comment (`ploopyfloofee`, not `abenedict`).
- Why: Phase 0 of `homelab-iac`. Docs: [hosts/prxmx02.md](hosts/prxmx02.md).

## 2026-10-08 — IaC tooling installed; prxmx02 documented; homelab-iac deploy key created

- **vibe-lab**: Installed into `~/.local/bin` (no sudo): OpenTofu 1.13.1, sops 3.13.3, age 1.3.2, uv 0.12.23, and
  ansible-core 2.21.5 (via `uv tool install`). Checksums were verified for tofu, sops and uv. age publishes no checksum file.
  Undo: `uv tool uninstall ansible-core`, then delete `tofu sops age age-keygen uv uvx` from `~/.local/bin`.
- **vibe-lab**: Generated `ssh/claude_iac_github_ed25519` (gitignored) and added `Host github-iac` to `ssh/config`, above
  `Host *` so GitHub sees that key first. Why: GitHub requires a unique deploy key per repo.
  Undo: delete the key pair and the `Host github-iac` block.
- Docs: added [hosts/prxmx02.md](hosts/prxmx02.md) from output you pasted. Nothing changed on prxmx02.
- Why: start of the `homelab-iac` project (a self-contained, rebuildable lab on Proxmox).

## 2026-10-06 — oracle1 code-server crash loop after password change fixed

- **oracle1** (you): changed the code-server password to an argon2 hash, but set `auth: hashed-password`. That isn't
  a valid value, so code-server crash-looped and the site showed a blank white page.
- **oracle1**: In `~/stacks/code-server/data/config/code-server/config.yaml`, changed `auth: hashed-password` to
  `auth: password` and renamed the `password:` key (holding the hash) to `hashed-password:`. Restarted the stack.
- Why: code-server only accepts `auth: password` or `auth: none`. A hash goes in the `hashed-password:` key.
- Undo: restore `config.yaml.bak-20261006` in the same folder (the broken config).

## 2026-10-04 — oracle1 pulls from GitHub instead of being pushed to

- **oracle1**: Ran `git fetch github` inside the code-server container to fix a stale `github/main` ref, which made
  code-server show 2 commits waiting to sync that GitHub already had.
- **vibe-lab**: Stopped pushing to oracle1. After pushing to GitHub, Claude now runs `git pull --ff-only` on oracle1
  inside the code-server container. The `oracle1` remote stays for fetch-only fallback. Decision 0003 and the README
  Sync section updated.
- **oracle1** (you): unset `receive.denyCurrentBranch` in `~/homelab-docs`, since nothing pushes into it now.
- **vibe-lab** (you): removed the `git push oracle1 main` allow rule from `.claude/settings.local.json`.
- **vibe-lab**: Added allow rule `Bash(ssh -F /home/abenedict/claude/ssh/config oracle1 *)` to `.claude/settings.local.json`,
  so Claude can run commands on oracle1 without a prompt (at your request). Undo: delete that line.
- Why: see [decisions/0003-github-main-copy.md](decisions/0003-github-main-copy.md) ("Pull, not push, on oracle1").
- Undo: on oracle1, `git config receive.denyCurrentBranch updateInstead`; restore the old Workflow from git history.

## 2026-10-04 — OPNsense notes updated

- Docs only, nothing changed on any system. [hosts/opnsense.md](hosts/opnsense.md): marked the WireGuard setup as an
  abandoned leftover (old Oracle server, not oracle1), deferred its cleanup, and recorded the plan to replace the
  router with Ubiquiti and switch ISP from Quantum Fiber to Google Fiber.

## 2026-10-04 — OPNsense router onboarded and documented

- **opnsense** (you, web UI): created user `claude` (admins group) with Claude's public key.
  - First attempt failed: oracle1's GitHub deploy key had been pasted by mistake (with a stray `+`, so it never worked). Replaced with the correct key.
- **vibe-lab**: Added `Host opnsense` (192.168.1.1, user `claude`) to `ssh/config`. Host key recorded in `ssh/known_hosts`
  (ED25519 `SHA256:9Dr9J1aiPywaX0OKE5L2HzgLPf8H2rrCX3JVduJT6u0`).
- **opnsense**: Read `config.xml` and live state (read-only, nothing changed). Results in [hosts/opnsense.md](hosts/opnsense.md),
  secrets left out. The raw config was copied to a temporary scratch folder for parsing and deleted afterwards.
- **vibe-lab**: Ran one `curl ifconfig.me` to check which route the server VLAN takes to the internet.
- Undo: delete user `claude` in System → Access → Users; remove the `Host opnsense` block and the `192.168.1.1` line in `ssh/known_hosts`.

## 2026-10-04 — GitHub becomes the main docs copy

- **vibe-lab**: `main` now tracks `github/main` (was `oracle1/main`). README Sync section rewritten.
- **oracle1**: In the code-server container, generated deploy key `~/.config/ssh/github_ed25519`
  (host: `~/stacks/code-server/data/config/ssh/`), fingerprint `SHA256:haUTj+nD2E1azG3PzAZYtiZU2AJMKrvRM+YD4cJHlG8`.
  Added GitHub's host key (copied from vibe-lab's verified `ssh/known_hosts`) to `known_hosts` next to it.
- **oracle1**: In `~/homelab-docs`, added remote `github` and set `core.sshCommand` to use that key.
- **vibe-lab**: Gitignored `.claude/settings.local.json` (Claude Code permissions for this machine; allows only `git push` to `oracle1 main` and `github main`).
- Why: see [decisions/0003-github-main-copy.md](decisions/0003-github-main-copy.md).
- Undo: see the Undo section of decision 0003.

## 2026-10-04 — GitHub off-site backup remote

- **vibe-lab**: Added git remote `github` = `git@github.com:abenedict/homelab-documentation.git` (private repo, already existed).
  - Why: off-site backup; before this the docs existed only on vibe-lab and oracle1.
  - Auth: Claude's existing key `ssh/claude_homelab_ed25519` added as a deploy key (write access) on that repo only. Reused rather than a new key because whoever holds it already has root on oracle1.
  - GitHub's host key added to `ssh/known_hosts`; fingerprint matched GitHub's published one (`SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU`).
  - The repo had two earlier commits (README added, then removed; empty tree). Merged them in with `--allow-unrelated-histories` instead of force-pushing, so nothing on GitHub was overwritten.
  - Undo: `git remote remove github`, delete the deploy key in the repo's Settings → Deploy keys, remove the `github.com` line from `ssh/known_hosts`.

## 2026-10-04 — Browser docs: Tailscale, code-server, git sync

Decision: [decisions/0002-docs-browser-access.md](decisions/0002-docs-browser-access.md)

- **vibe-lab**: `~/claude` is now a git repo. Added `.gitignore` (private key excluded). Repo-local identity `abenedict <abenedict@vibe-lab>` (no work email). Remote `oracle1` added; `core.sshCommand` uses `ssh/config`.
- **oracle1**: Installed Tailscale 1.102.4 (apt repo `pkgs.tailscale.com`), joined the tailnet as `oracle1` (100.76.43.37). Tailscale SSH not enabled.
  - Undo: `sudo tailscale down && sudo apt purge tailscale`, then remove the machine from the Tailscale admin console.
- **oracle1**: Created `~/homelab-docs` (git working copy, `receive.denyCurrentBranch=updateInstead`).
- **oracle1**: Deployed code-server from `configs/oracle1/code-server/compose.yaml` to `~/stacks/code-server/`. Bound to 127.0.0.1:8080, runs as uid 1001.
  - Undo: `cd ~/stacks/code-server && docker compose down && rm -rf ~/stacks/code-server`
- **oracle1**: `tailscale serve --bg --https=443 http://127.0.0.1:8080` (requires Serve/HTTPS enabled in the tailnet admin).
  - Undo: `sudo tailscale serve reset`
  - Note: Serve was enabled in the tailnet admin console first, then the command was re-run. Verified: valid HTTPS certificate, redirects to the code-server login page.

## 2026-10-04 — oracle1: Docker installed

- **oracle1**: Installed Docker CE 29.8.2, buildx, and the Compose plugin (v5.6.0) from Docker's official apt repo (`/etc/apt/sources.list.d/docker.sources`, key `/etc/apt/keyrings/docker.asc`).
  - Why the official repo instead of Ubuntu's `docker.io`: newer releases and the Compose v2+ plugin. Updates arrive through normal `apt upgrade`.
  - Added `ubuntu` to the `docker` group (that group effectively has root access).
  - `/etc/docker/daemon.json`: json-file log rotation, max 10 MB × 3 per container, so logs can't fill the 50 GB disk.
  - Verified with `hello-world`.
  - Undo: `sudo apt purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin && sudo rm -rf /var/lib/docker /etc/docker /etc/apt/sources.list.d/docker.sources /etc/apt/keyrings/docker.asc`

## 2026-10-04 — oracle1 onboarded

- **vibe-lab**: Added `Host oracle1` (129.80.178.38, user `ubuntu`) to `ssh/config`. Server's host key recorded in `ssh/known_hosts`.
- **oracle1**: Checked the server's setup (read-only, nothing changed). Results are in [hosts/oracle1.md](hosts/oracle1.md).
  - Note: my first login attempt was rejected because the key wasn't installed yet; it worked after it was added.

## 2026-10-04 — workspace setup

- **vibe-lab (local workstation)** — Created workspace structure (`README.md`, `docs/`, `ssh/`).
- **vibe-lab** — Generated Claude's SSH key `ssh/claude_homelab_ed25519` (ed25519, no passphrase).
  - Fingerprint: `SHA256:tlmvXHny9qvzD+ezLs070hpQ207qJf4OE4pZDSozoFA`
  - Why: see [decisions/0001-claude-ssh-key.md](decisions/0001-claude-ssh-key.md)
  - Undo: delete the key files and remove the public key line from `~/.ssh/authorized_keys` on every server it was added to.
- **vibe-lab** — Created `ssh/config` (dedicated SSH config; does not touch `~/.ssh/config`) and empty `ssh/known_hosts`.
