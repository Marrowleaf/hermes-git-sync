---
name: git-sync
description: "Auto-commit and push Obsidian vault changes with meaningful commit messages and scheduled sync."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [git, sync, obsidian, backup, cron, vault, push, commit]
    related_skills: [obsidian]
commands:
  - name: status
    description: "Show git status of the Obsidian vault — modified, added, deleted files."
    usage: git-sync status
  - name: sync
    description: "Stage all changes, commit with an auto-generated message, and push to remote."
    usage: git-sync sync [--message <msg>] [--dry-run]
    examples:
      - "git-sync sync"
      - "git-sync sync --dry-run"
      - "git-sync sync --message 'added habit tracker config'"
  - name: diff
    description: "Show a summary of changes since last commit (file names + change types)."
    usage: git-sync diff [--stat | --full]
  - name: log
    description: "Show recent git log for the vault."
    usage: git-sync log [-n <count>]
  - name: init
    description: "Initialize git repo in the Obsidian vault and set up remote."
    usage: git-sync init [--remote <url>]
  - name: conflicts
    description: "Check for and help resolve merge conflicts."
    usage: git-sync conflicts
  - name: schedule
    description: "Set up or show the cron schedule for auto-sync."
    usage: git-sync schedule [show|set <cron-expression>]
    examples:
      - "git-sync schedule show"
      - "git-sync schedule set '0 */6 * * *'"
---

# Git Sync

Auto-commit and push Obsidian vault changes with meaningful commit messages. Designed for scheduled cron runs and manual sync triggers.

## When to Use This Skill

- User wants to back up their Obsidian vault
- User mentions "sync my notes" or "push my vault"
- A cron job needs to auto-sync
- User wants to see what changed since last sync
- User asks about merge conflicts in their vault

## Prerequisites

- Git must be initialized in `~/obsidian-vault/`
- A remote must be configured (GitHub, GitLab, etc.)
- SSH key or token must be set up for push access

## Step 1: Check Status

```bash
cd ~/obsidian-vault
git status --porcelain
```

If no changes, report "Vault is clean — no changes to sync." and stop.

## Step 2: Generate Commit Message

Auto-generate from the diff:

```bash
# Get list of changed files with status
CHANGES=$(git diff --name-status HEAD 2>/dev/null || git diff --name-status --cached)
STAGED=$(git diff --cached --name-only)
UNTRACKED=$(git ls-files --others --exclude-standard)
```

**Message format:**
```
sync: <count> files changed

- 3 modified: habits/2026-05.md, budget/2026-05.md, fitness/log.md
- 1 added: habits/config.md
- 2 deleted: archive/old-note.md, temp/draft.md
```

Truncate at 50 files. If only one file changed, use:
```
sync: update habits/2026-05.md
```

## Step 3: Stage and Commit

```bash
cd ~/obsidian-vault
git add -A
git commit -m "$COMMIT_MSG"
```

**For dry-run:** show what would be committed without actually committing:
```bash
git add -A
git diff --cached --stat
# Then git reset HEAD to unstage
```

## Step 4: Pull then Push

```bash
# Pull with rebase to keep history clean
git pull --rebase origin main
# Handle conflicts if they arise (see Step 5)
git push origin main
```

**Push failure handling:**
- If push is rejected, pull --rebase and try again (max 3 retries)
- If remote doesn't exist, suggest `git-sync init`
- If auth fails, suggest checking SSH key or token

## Step 5: Conflict Resolution

If `git pull --rebase` produces conflicts:

1. List conflicting files: `git diff --name-only --diff-filter=U`
2. For each conflict, read the file and identify the conflict markers
3. For Obsidian notes (our vault), prefer the LOCAL version (our changes) since ours are more recent
4. For auto-generated files (like habit logs), merge both sides
5. Stage resolved files: `git add <resolved-file>`
6. Continue rebase: `git rebase --continue`
7. If unable to resolve, offer `git rebase --abort` and manual resolution

**Auto-resolution strategy:**
- `.md` files in `3-Resources/`: Take ours (local) — our data is more recent
- `.md` files in `Inbox/`: Merge both (newlines between sections)
- `.md` files in `1-Projects/`: Take ours (active work)
- `config.md` files: Keep both additions, flag for review
- Everything else: Abort and ask user

## Step 6: Verify

After sync:

```bash
cd ~/obsidian-vault
echo "=== Last 3 commits ==="
git log --oneline -3
echo "=== Remote status ==="
git status --short
echo "=== File count ==="
find . -name '*.md' | wc -l
```

Report: "✅ Vault synced. N files pushed. Last commit: <message>"

## Step 7: Scheduled Sync (Cron)

To set up auto-sync via Hermes cron:

```bash
# Every 6 hours
hermes cron create --name "Vault Sync" --schedule "0 */6 * * *" --prompt "Run git-sync sync on the Obsidian vault"

# Every hour during waking hours (7am-11pm BST)
hermes cron create --name "Vault Sync" --schedule "0 6-22 * * *" --prompt "Run git-sync sync on the Obsidian vault"
```

## Initialization

If the vault has no git repo:

```bash
cd ~/obsidian-vault
git init
git add -A
git commit -m "init: vault initial commit"

# Add remote (user must provide URL)
git remote add origin <REMOTE_URL>
git push -u origin main
```

Create `.gitignore`:
```
.obsidian/
.trash/
.DS_Store
*.tmp
*.swp
```

## Common Pitfalls

1. **Large binary files** — Obsidian can accumulate PDFs and images. Consider `git-lfs` or excluding `Attachments/` from tracking if the vault is large.
2. **Merge conflicts on auto-sync** — If two devices edit the same note, conflicts will arise. Configure `pull --rebase` as default.
3. **Empty commits** — Don't commit if there are no changes. Check `git diff --exit-code` first.
4. **Detached HEAD** — Always verify we're on `main` before operating: `git checkout main`
5. **SSH key in cron** — Cron jobs may not have access to SSH agent. Ensure the key is loaded or use a token-based remote URL.
6. **File permissions** — Git may track permission changes on Linux. Run `git config core.fileMode false` in the vault.

## Verification

After `git-sync sync`:
1. `git log --oneline -1` — verify the commit was made
2. `git status --short` — should be empty (clean working tree)
3. `git remote -v` — verify remote is configured