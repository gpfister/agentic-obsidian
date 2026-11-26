---
adr_id: "016"
title: "Obsidian Vault Git Worktree & PR Lifecycle for @coding-document"
status: "accepted"
date: "2026-11-26"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/80702574-ccc2-4666-8292-74b14ad1fcb5/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/7c7af799-883f-444a-a23e-3c109736f1cf/plan.md"
tags:
  - architecture-decision
  - git-worktree
  - coding-document
  - obsidian-vault
  - pull-request
  - question-relay
  - defer-pr
  - plan-approved
aliases:
  - ADR-016
  - adr-016-obsidian-vault-worktree-and-pr-lifecycle
---

# ADR-016: Obsidian Vault Git Worktree & PR Lifecycle for `@coding-document`

See canonical record: [[architecture/adr-016-obsidian-vault-worktree-and-pr-lifecycle|ADR-016]].

## Summary

- **Self-Managed `*-obsidian/` Lifecycle**: `@coding-document` owns worktree isolation (`<vault_root>/.worktrees/<slug>` on `docs/<slug>`), incremental commits, push, PR creation, Hybrid PR Waiting Gate, base sync, and cleanup. `@coding-merge` remains scoped to code repositories.
- **Multi-Phase `defer_pr` Accumulation**: `@coding` passes consistent `slug` and `defer_pr=true` on Post-Plan and Pre-Merge hooks, and `defer_pr=false` on Post-Merge to publish a single consolidated PR; single-shot callers default to `defer_pr=false`.
- **Release Strategies & Non-Blocking Gates**: Supports Remote (`gh pr create` / compare URL fallback + Hybrid PR Waiting Gate via `[QUESTION_RELAY_REQUEST]`), Local-Only merge, and Non-Git Fallback, while preserving `edit_file` prohibition on `@coding-document`.
