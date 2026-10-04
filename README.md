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
| `configs/<host>/` | Config files deployed to servers (e.g. Docker compose files), kept here as the source of truth |

## Conventions

- Every change made to a system gets a `docs/changelog.md` entry (date, host, what, why, how to undo).
- Non-obvious choices get a decision record in `docs/decisions/NNNN-short-title.md`.
- New servers get a page in `docs/hosts/` and a `Host` block in `ssh/config`.
- Secrets (passwords, API tokens) are **not** written into docs — docs say where the secret lives instead.
- Before editing docs: `git pull oracle1 main` (picks up edits made in the browser). After: commit, then `git push oracle1 main && git push github main`.

## Quick reference

Connect to a host the way Claude does:

```bash
ssh -F ~/claude/ssh/config <host-alias>
```

## Viewing docs in a browser

`https://oracle1.tail80394.ts.net` (needs Tailscale on the device). The code-server password is in
`/home/ubuntu/stacks/code-server/data/config/code-server/config.yaml` on oracle1.
To view the password from vibe-lab: `ssh -F ~/claude/ssh/config oracle1 grep password: stacks/code-server/data/config/code-server/config.yaml`
See [docs/decisions/0002-docs-browser-access.md](docs/decisions/0002-docs-browser-access.md).

## Sync

This folder is a git repo with two remotes:

| Remote | Where | Role |
|---|---|---|
| `oracle1` | `/home/ubuntu/homelab-docs` on oracle1 | Primary; browser edits in code-server land here |
| `github` | `git@github.com:abenedict/homelab-documentation.git` (private) | Off-site backup |

`git pull oracle1 main`, then after committing `git push oracle1 main && git push github main`.
GitHub access uses Claude's key (`ssh/claude_homelab_ed25519`) as a write-enabled deploy key on that repo only.
The private SSH key is gitignored.
