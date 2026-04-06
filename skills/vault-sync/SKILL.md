---
name: vault-sync
description: "Sync an OpenClaw workspace to a private Git repository as a backup. Use when user asks to back up their workspace, sync vault to GitHub, set up workspace git sync, or needs a recurring backup of their agent's memory and files. Also useful as a reference for setting up cron-based git push patterns."
---

# vault-sync — Git-Based Workspace Backup

Automatically commit and push your OpenClaw workspace to a private Git repository. Keeps a versioned history of all workspace changes — memory files, notes, configs, skills.

## Architecture

```
~/.openclaw/workspace/ ──git add/commit/push──▶ Private GitHub repo
                              ▲
                        Cron (every N hours)
```

## Prerequisites

- Git initialized in workspace directory
- Remote repository created (GitHub, GitLab, etc.)
- Authentication configured (PAT, SSH key, or credential helper)
- `.gitignore` for any files you don't want synced

## Setup

### 1. Initialize git in workspace
```bash
cd ~/.openclaw/workspace
git init
git remote add origin https://github.com/<user>/<repo>.git
```

### 2. Create .gitignore
```gitignore
# Secrets — never commit
*.key
*.pem
*.env
.env*

# Large/binary files
*.sqlite
*.db
node_modules/

# Temp files
*.tmp
*.swp
.DS_Store
```

### 3. Authentication
Use a Personal Access Token (PAT) stored in your OS keychain:

```bash
# macOS Keychain
security add-generic-password -a $(whoami) -s "github-pat" -w "<your-pat>"

# Use in push command
PAT=$(security find-generic-password -s "github-pat" -w)
git push https://<user>:$PAT@github.com/<user>/<repo>.git main
```

Or use SSH keys (no token management needed):
```bash
git remote set-url origin git@github.com:<user>/<repo>.git
git push origin main
```

### 4. First commit
```bash
cd ~/.openclaw/workspace
git add -A
git commit -m "Initial vault sync"
git push origin main
```

## Cron Job Setup

Set up an OpenClaw cron to run every 4 hours:

```
Schedule: 0 */4 * * * (every 4 hours)
Session: isolated
Model: sonnet (lightweight)
```

**Cron payload:**
```
Run a git sync for the vault:
1. cd <workspace-path> && git add -A
2. Check if there are changes with git status --porcelain
3. If changes exist, commit with a descriptive message based on what changed
4. Push to remote
Do this silently unless there's an error.
```

The agent generates a meaningful commit message based on which files changed (e.g., "Update daily notes, add new recipe" instead of "auto-sync").

## Conflict Resolution

Since only one agent writes to the workspace, conflicts are rare. If they occur:

```bash
git pull --rebase origin main
# Fix any conflicts
git push origin main
```

## What Gets Synced

Typical workspace contents:
- `MEMORY.md` — long-term curated memory
- `memory/` — daily notes, facts, active work
- `SOUL.md`, `USER.md` — identity and user context
- `TOOLS.md`, `AGENTS.md` — configuration notes
- Skills, scripts, reference files

## Security Notes

- **Always use a private repository** — workspace contains personal context
- **Never commit secrets** — use `.gitignore` and keychain for sensitive data
- **PATs expire** — set a reminder to rotate before expiry
- **Review commits** — periodically check what's being synced
- The agent should never log PATs or credentials in commit messages or output

## Tips

- Use `git status --porcelain` to check for changes before committing (avoids empty commits)
- Keep commit messages descriptive — they become your workspace changelog
- Add `staggerMs` to cron schedule to avoid exact-hour collisions with other jobs
- If push fails (network), the next sync will pick up accumulated changes
- Consider `git gc` occasionally to keep repo size manageable
