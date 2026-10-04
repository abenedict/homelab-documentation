# Changelog

Newest first. Format: date — host(s) — what — why — how to undo.

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
