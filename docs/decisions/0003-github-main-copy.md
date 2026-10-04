# 0003: GitHub is the main copy of the docs

**Date:** 2026-10-04
**Status:** Accepted (supersedes the Sync and Workflow sections of [0002](0002-docs-browser-access.md))

## Decision

- The private repo `git@github.com:abenedict/homelab-documentation.git` is the main copy. Every
  copy pulls from and pushes to it.
- **vibe-lab** (`~/claude`): `main` tracks `github/main`. Auth: Claude's key
  `ssh/claude_homelab_ed25519` as a write-enabled deploy key.
- **oracle1** (`~/homelab-docs`, edited in code-server): has its own `github` remote and its own
  deploy key, `github_ed25519`, stored in `~/stacks/code-server/data/config/ssh/` (mounted in the
  container at `/home/coder/.config/ssh/`). The repo's `core.sshCommand` points at that key and at a
  `known_hosts` holding GitHub's host key.
- vibe-lab still pushes to oracle1 directly (`receive.denyCurrentBranch=updateInstead`), so the
  browser copy's files update right away.

## Why

- **Off-site and one source of truth:** before this, the docs only existed on vibe-lab and oracle1.
  GitHub also gives a web view and history.
- **oracle1 pushes to GitHub itself:** otherwise browser edits would reach GitHub only when vibe-lab
  next relayed them.
- **Separate key per machine:** vibe-lab's private key never leaves vibe-lab (decision 0001). Each
  deploy key is limited to this one repo and can be revoked on its own.
- **Key in the code-server config volume:** browser commits run inside the container, which can't
  see the host's `~/.ssh`. The volume is already mounted, so no compose change or restart was needed.

## Tradeoffs

- If GitHub is down or a key is revoked, syncing through it stops. vibe-lab ↔ oracle1 direct pushes
  still work as a fallback.
- Anyone with access to the code-server container can push to the GitHub repo (not to anything else).

## Workflow

- **Claude (vibe-lab):** `git pull github main` before editing. After: commit, `git push github main`,
  then `git push oracle1 main` to refresh the browser copy. If oracle1 has unpushed browser commits,
  that push is refused instead of overwriting them.
- **You (browser):** before editing, Source Control → Pull (or Sync). After editing, commit, then Sync
  (or Push) to send it to GitHub.

## Undo

Delete both deploy keys in the GitHub repo's Settings → Deploy keys. On oracle1:
`cd ~/homelab-docs && git remote remove github && git config --unset core.sshCommand && rm -r ~/stacks/code-server/data/config/ssh`.
On vibe-lab: `git branch -u oracle1/main`, then restore the README Sync section from git history.
