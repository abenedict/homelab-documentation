# Changelog

Newest first. Format: date — host(s) — what — why — how to undo.

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
