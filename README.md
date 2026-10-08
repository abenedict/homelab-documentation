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
| `docs/plans/` | Working checklists and plans for projects in progress |
| `ssh/` | Claude's SSH key, SSH config, and known_hosts (dir is `chmod 700`) |
| `configs/<host>/` | Config files deployed to servers (e.g. Docker compose files), kept here as the source of truth |

## Conventions

- Every change made to a system gets a `docs/changelog.md` entry (date, host, what, why, how to undo).
- Non-obvious choices get a decision record in `docs/decisions/NNNN-short-title.md`.
- New servers get a page in `docs/hosts/` and a `Host` block in `ssh/config`.
- Secrets (passwords, API tokens) are **not** written into docs — docs say where the secret lives instead.
- Before editing docs: `git pull github main`. After: commit, `git push github main`, then pull on oracle1 (see Sync).

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

GitHub is the main copy. This folder is a git repo; `main` tracks `github/main`.

| Remote | Where | Role |
|---|---|---|
| `github` | `git@github.com:abenedict/homelab-documentation.git` (private) | Main copy; everything syncs through it |
| `oracle1` | `/home/ubuntu/homelab-docs` on oracle1 | Browser (code-server) working copy; fetch-only fallback, never pushed to |

`git pull github main`, then after committing `git push github main`, then update the browser copy from GitHub:
`ssh -F ~/claude/ssh/config oracle1 docker exec -w /home/coder/homelab-docs code-server git pull --ff-only`.
The oracle1 copy also has a `github` remote, so browser edits go to GitHub directly (Source Control → Sync).
Each machine has its own write-enabled deploy key on the GitHub repo. Private keys are never synced.
See [docs/decisions/0003-github-main-copy.md](docs/decisions/0003-github-main-copy.md).
