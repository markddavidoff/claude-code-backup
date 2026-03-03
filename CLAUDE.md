# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Backup/restore utility for Claude Code configurations. Uses a two-repo architecture: this public repo contains scripts, while a private GitHub repo (linked via `backup/` subdir) stores encrypted user data. The `backup/` directory is gitignored to prevent accidental public exposure.

## Architecture

```
~/.claude/ configs ──> backup.sh/ps1 ──> backup/ (local git) ──> Private GitHub Repo
Private GitHub Repo ──> restore.sh/ps1 ──> ~/.claude/ configs
```

**Scripts come in pairs** — every feature must be implemented in both `bash (.sh)` and `PowerShell (.ps1)` with full feature parity:
- `setup.sh` / `setup.ps1` — One-time init linking `backup/` to private repo
- `backup.sh` / `backup.ps1` — Copy configs to `backup/` and push
- `restore.sh` / `restore.ps1` — Restore configs from `backup/`

## What Gets Backed Up

`settings.json`, `installed_plugins.json`, `known_marketplaces.json`, `mcp.json` (~/.mcp.json), `keybindings.json`, `CLAUDE.md` (global memory), `projects/` (per-project settings), `commands/` (custom slash commands), `skills/`, `todos/`

## Merge Strategies

These are intentional design decisions — do not change without understanding multi-machine implications:

- **Backup pull**: `git pull` before backup to merge latest remote changes
- **Projects/commands restore**: `rsync --delete` (full replace from backup)
- **Skills/todos restore**: `rsync` without `--delete` (non-destructive merge)
- **CLAUDE.md**: Union merge on restore (appends new lines, never overwrites)
- **Setup multi-machine**: Local files win on conflicts, remote-only files preserved

## Setup Handles 3 Scenarios

1. **Empty remote** — init new repo, push placeholder
2. **Remote has data, no local** — simple clone
3. **Both have data** — 3-way merge (clone remote → overlay local → merge commit)

## Development Guidelines

- Maintain feature parity between `.sh` and `.ps1` scripts
- Always check file/directory existence before copy operations
- Git operations should warn on failure but not abort the script
- Bash scripts use `set -euo pipefail`; PowerShell checks `$LASTEXITCODE`
- Windows scripts need `core.longpaths = true` for git
