# 0002: Browser access to docs: code-server + Tailscale + git

**Date:** 2026-10-04
**Status:** Accepted; the Sync and Workflow sections are superseded by [0003](0003-github-main-copy.md) (GitHub is now the main copy)

## Decision

- **Editor:** code-server (`codercom/code-server`, VS Code in the browser), running in Docker on oracle1.
- **Access:** only over Tailscale. The container listens on `127.0.0.1:8080`. `tailscale serve`
  publishes it on the tailnet at `https://oracle1.tail80394.ts.net` with an automatic HTTPS
  certificate. No ports are opened in Oracle's firewall or the host iptables.
- **Sync:** `~/claude` on vibe-lab is a git repo. oracle1 has a working-copy repo at
  `/home/ubuntu/homelab-docs` with `receive.denyCurrentBranch=updateInstead`, so a push from
  vibe-lab updates the files immediately. Edits made in the browser are committed there and pulled
  back to vibe-lab.

## Why

- **Tailscale instead of a public site:** code-server includes a terminal, so anyone who gets in
  controls the server. Tailscale keeps it off the internet entirely. No domain, open ports, or
  extra login layer needed. You were already using a tailnet (macbook, `apps`).
- **`tailscale serve` HTTPS:** code-server needs HTTPS for some features (clipboard, webviews,
  service worker). Tailscale provides a trusted certificate for free.
- **code-server password kept on:** an extra layer in case another device on the tailnet is compromised.
- **Git, two-way:** gives a full history of every doc change and lets edits flow both directions.
  If the server has uncommitted edits, the push is refused instead of overwriting them.
- **Container runs as uid 1001 (`ubuntu`):** on this image `ubuntu` is 1001 (Oracle's `opc` took
  1000). This keeps file ownership consistent with the repo.

## What is NOT synced

`ssh/claude_homelab_ed25519` (private key) is in `.gitignore` and never leaves vibe-lab. The public
key, `ssh/config`, and `known_hosts` are synced (not secret).

## Workflow

- Claude: before editing docs, `git pull oracle1 main` to pick up any browser edits. After changes,
  commit and `git push oracle1 main`.
- You (browser): edit, then commit using the Source Control panel. No push is needed: Claude pulls from it.
  Uncommitted browser edits block Claude's push, which is intentional.
