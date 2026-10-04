# Home / Homelab Workspace

Personal workspace where Claude helps with the homelab and other home projects.
Everything Claude generates or changes is recorded here so it can be rebuilt or audited later.

## Layout

| Path | What's in it |
|---|---|
| `README.md` | This index |
| `claude.md` | Project instructions for Claude (written by me) |
| `docs/changelog.md` | Chronological log of everything done, newest first |
| `docs/decisions/` | Why something was configured a certain way (one file per decision) |
| `docs/hosts/` | One file per server/device: IP, OS, role, services, how to access |
| `ssh/` | Claude's SSH key, SSH config, and known_hosts (dir is `chmod 700`) |

## Conventions

- Every change made to a system gets a `docs/changelog.md` entry (date, host, what, why, how to undo).
- Non-obvious choices get a decision record in `docs/decisions/NNNN-short-title.md`.
- New servers get a page in `docs/hosts/` and a `Host` block in `ssh/config`.
- Secrets (passwords, API tokens) are **not** written into docs — docs say where the secret lives instead.

## Quick reference

Connect to a host the way Claude does:

```bash
ssh -F ~/claude/ssh/config <host-alias>
```
