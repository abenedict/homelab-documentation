# Changelog

Newest first. Format: date — host(s) — what — why — how to undo.

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
