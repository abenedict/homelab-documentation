# 0001 — Dedicated SSH key for Claude

**Date:** 2026-10-04
**Status:** Accepted

## Decision

Claude uses its own SSH key, `ssh/claude_homelab_ed25519`, stored in this workspace, with a
dedicated SSH config at `ssh/config` (invoked via `ssh -F`).

## Details

- **Type:** ed25519, 100 KDF rounds (`ssh-keygen -t ed25519 -a 100`)
- **Comment:** `claude-code@vibe-lab (homelab automation key, created 2026-10-04)` — makes it easy to
  find and remove in any server's `authorized_keys`.
- **Fingerprint:** `SHA256:tlmvXHny9qvzD+ezLs070hpQ207qJf4OE4pZDSozoFA`
- **Passphrase:** none.
- **Host key checking:** `accept-new` — first connection trusts and records the host key in
  `ssh/known_hosts`; a *changed* key afterwards is refused.

## Why

- **Separate key instead of reusing mine:** access Claude has can be audited and revoked on its own
  without touching my personal keys. Searching `authorized_keys` for `claude-code@vibe-lab` shows
  exactly where it's installed.
- **No passphrase:** Claude runs commands non-interactively and can't type a passphrase. Tradeoff:
  anyone who can read this file can use the key. Mitigations: file is `600`, directory is `700`,
  and it lives only on this workstation.
- **Separate SSH config/known_hosts:** keeps my own `~/.ssh` untouched.

## Possible hardening later

- Prefix the key in `authorized_keys` with `from="<vibe-lab IP>"` so it only works from this machine.
- Use a non-root user with scoped sudo on each server rather than root login.

## Revoking

1. On each server: remove the line containing `claude-code@vibe-lab` from `~/.ssh/authorized_keys`.
2. Locally: delete `ssh/claude_homelab_ed25519*`.
