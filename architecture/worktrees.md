---
title: "Parallel Agent Sessions with Git Worktrees"
date: "2026-10-25"
tags:
  - architecture
  - concurrency
  - git-worktree
---

# Parallel Agent Sessions with Git Worktrees

## Problem & Motivation
In a multi-agent or multi-session Antigravity environment, having multiple sessions write to the same working directory causes:
- Uncommitted file overwrites and race conditions.
- Git checkout conflicts when agents switch or create branches.
- Interrupted build or test states.

## Worktree Solution
Instead of cloning separate repositories, Git worktrees allow multiple working directories attached to the same repository. Each worktree:
- Checks out its own branch independently (e.g. `session/<session-id>`).
- Shares the local git object database without duplicating disk space.
- Allows simultaneous, non-blocking execution across parallel sessions.

## Directory Organization
Worktrees are provisioned in `.worktrees/` at the root of `agentic/`:

```text
agentic/
├── .worktrees/
│   ├── session-alpha/
│   │   ├── agentic-root/         (branch: session/session-alpha)
│   │   ├── gp-coding-plugin/     (branch: session/session-alpha-gp-coding-plugin)
│   │   └── obsidian-vault/       (branch: session/session-alpha-obsidian-vault)
│   └── session-beta/
│       └── gp-coding-plugin/     (branch: session/session-beta-gp-coding-plugin)
```

All `.worktrees/` folders are globally ignored across `.gitignore` files so that parent repositories remain completely clean.

## Lifecycle Automation
The Python CLI tool `scripts/worktree.py` manages lifecycle commands:
- `create <session_name> [--repo <name>] [--branch <branch>]`
- `list`
- `remove <session_name> [-D]`
- `prune`
