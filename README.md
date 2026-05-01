# Git Sync

> Automatically sync and version-control your Obsidian vault and notes with Git.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://github.com/Marrowleaf/hermes-git-sync)

## Features

- Automatic Git commit and push for Obsidian vault changes
- Scheduled sync via cron (configurable intervals)
- Conflict detection and resolution assistance
- Branch management for note experiments
- Commit messages generated from changed filenames
- Selective sync — ignore temp/cache files
- Rollback support to restore previous note versions
- Multi-device sync coordination

## Installation

```bash
hermes skills install note-taking/git-sync
```

Or manually clone into `~/.hermes/skills/note-taking/git-sync/`.

## Usage

```
git-sync push
git-sync pull
git-sync status
git-sync rollback --file "Daily Notes/2025-01-15.md"
git-sync init --remote https://github.com/user/vault.git
git-sync log --last 10
```

## Configuration

- `GIT_SYNC_REMOTE`: Git remote URL for your vault repository
- `GIT_SYNC_BRANCH`: Branch to sync (default: `main`)
- `GIT_SYNC_INTERVAL`: Cron interval in minutes (default: 15)
- `GIT_SYNC_COMMIT_NAME`: Commit author name
- `GIT_SYNC_COMMIT_EMAIL`: Commit author email

Configure in `config.md` or via environment variables.

## Requirements

- Hermes Agent v0.12+
- Git installed and configured
- SSH key or token for remote repository access

## License

MIT