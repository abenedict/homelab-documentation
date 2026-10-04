# 0003: GitHub is the main copy of the docs

**Date:** 2026-10-04
**Status:** Accepted (supersedes the Sync and Workflow sections of [0002](0002-docs-browser-access.md)).
Revised 2026-10-04: vibe-lab no longer pushes to oracle1; every copy pulls from GitHub.

## Decision

- The private repo `git@github.com:abenedict/homelab-documentation.git` is the main copy. Every
  copy pulls from and pushes to it.
- **vibe-lab** (`~/claude`): `main` tracks `github/main`. Auth: Claude's key
  `ssh/claude_homelab_ed25519` as a write-enabled deploy key.
- **oracle1** (`~/homelab-docs`, edited in code-server): has its own `github` remote and its own
  deploy key, `github_ed25519`, stored in `~/stacks/code-server/data/config/ssh/` (mounted in the
  container at `/home/coder/.config/ssh/`). The repo's `core.sshCommand` points at that key and at a
  `known_hosts` holding GitHub's host key.
- Nothing pushes into oracle1. After Claude pushes to GitHub, it pulls on oracle1 by running
  `git pull --ff-only` inside the code-server container (the deploy key only exists there). The
  container runs as `ubuntu` (uid 1001), the repo's owner, so file ownership stays correct.
- vibe-lab keeps its `oracle1` remote for fetching only, as a fallback if GitHub is unreachable.

## Why

- **Off-site and one source of truth:** before this, the docs only existed on vibe-lab and oracle1.
  GitHub also gives a web view and history.
- **oracle1 pushes to GitHub itself:** otherwise browser edits would reach GitHub only when vibe-lab
  next relayed them.
- **Separate key per machine:** vibe-lab's private key never leaves vibe-lab (decision 0001). Each
  deploy key is limited to this one repo and can be revoked on its own.
- **Key in the code-server config volume:** browser commits run inside the container, which can't
  see the host's `~/.ssh`. The volume is already mounted, so no compose change or restart was needed.
- **Pull, not push, on oracle1:** at first vibe-lab also pushed straight to oracle1 so the browser
  copy updated instantly. That updated oracle1's files but not its record of `github/main`, so
  code-server showed Claude's commits as changes waiting to sync. With every copy pulling from
  GitHub, each one's view of GitHub stays accurate and there is one path for every change.

## Tradeoffs

- If GitHub is down or a key is revoked, syncing through it stops. vibe-lab can still fetch browser
  commits from oracle1 directly (`git fetch oracle1`), but can't push into it.
- Anyone with access to the code-server container can push to the GitHub repo (not to anything else).

## Workflow

- **Claude (vibe-lab):** `git pull github main` before editing. After: commit, `git push github main`,
  then update the browser copy:
  `ssh -F ~/claude/ssh/config oracle1 docker exec -w /home/coder/homelab-docs code-server git pull --ff-only`.
  If oracle1 has uncommitted or unpushed browser edits that conflict, the pull is refused instead of
  merging or overwriting them.
- **You (browser):** before editing, Source Control → Pull (or Sync). After editing, commit, then Sync
  (or Push) to send it to GitHub.

## Undo

Delete both deploy keys in the GitHub repo's Settings → Deploy keys. On oracle1:
`cd ~/homelab-docs && git remote remove github && git config --unset core.sshCommand && rm -r ~/stacks/code-server/data/config/ssh`.
On vibe-lab: `git branch -u oracle1/main`, then restore the README Sync section from git history.
To go back to pushing into oracle1 directly: on oracle1, `cd ~/homelab-docs && git config receive.denyCurrentBranch updateInstead`.
