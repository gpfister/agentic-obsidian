---
title: "Obsidian Vault Git Worktree & PR Lifecycle (@coding-document)"
date: "2026-11-26"
status: "Merged / Completed"
phase: "Phase 7: Merge & Release Complete"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/80702574-ccc2-4666-8292-74b14ad1fcb5/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/7c7af799-883f-444a-a23e-3c109736f1cf/plan.md"
review_path: "/Users/gpfister/.gemini/antigravity/brain/e6771348-445c-4880-88aa-179e69836f9d/review-obsidian-worktree-pr-lifecycle.md"
adrs:
  - "[[architecture/adr-016-obsidian-vault-worktree-and-pr-lifecycle|ADR-016: Obsidian Vault Git Worktree & PR Lifecycle]]"
tags:
  - feature
  - git-worktree
  - coding-document
  - obsidian-vault
  - pull-request
  - question-relay
  - defer-pr
  - specification
  - approved-spec
  - implementation-plan
  - approved-plan
  - phase-7-completed
  - merged
  - released
affected_plugins:
  - agentic-project-documenting-plugin
  - agentic-coding-plugin
aliases:
  - obsidian-worktree-pr-lifecycle
---

# Feature: Obsidian Vault Git Worktree & PR Lifecycle (`@coding-document`)

> [!NOTE] Lifecycle Status: Phase 7 Complete — Merged & Released (`MERGED`)
> - **Architecture Decision Record**: [[architecture/adr-016-obsidian-vault-worktree-and-pr-lifecycle|ADR-016: Obsidian Vault Git Worktree & PR Lifecycle for @coding-document]]
> - **Architecture Plan Note**: [[architecture/obsidian-worktree-pr-lifecycle|Obsidian Vault Git Worktree & PR Lifecycle Architecture Plan]]
> - **Merged Repositories**:
>   - [`agentic-project-documenting-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin): [PR #3](https://github.com/gpfister/agentic-project-documenting-plugin/pull/3) (`MERGED` into `main` at [`bb6ce00`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin), `15/15` pytest passed; `.worktrees/obsidian-worktree-pr-lifecycle` removed & pruned, `feature/obsidian-worktree-pr-lifecycle` deleted locally & remotely)
>   - [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin): [PR #8](https://github.com/gpfister/agentic-coding-plugin/pull/8) (`MERGED` into `main` at [`b843fbe`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin), `28/28` pytest passed; `.worktrees/obsidian-worktree-pr-lifecycle` removed & pruned, `feature/obsidian-worktree-pr-lifecycle` deleted locally & remotely)
> - **Current Phase**: Phase 7: Merge & Release Complete (`Merged / Completed`)
> - **Approved Spec**: [`specification.md`](file:///Users/gpfister/.gemini/antigravity/brain/80702574-ccc2-4666-8292-74b14ad1fcb5/specification.md)
> - **Approved Plan**: [`plan.md`](file:///Users/gpfister/.gemini/antigravity/brain/7c7af799-883f-444a-a23e-3c109736f1cf/plan.md)
> - **Review & QA Audit**: [`review-obsidian-worktree-pr-lifecycle.md`](file:///Users/gpfister/.gemini/antigravity/brain/e6771348-445c-4880-88aa-179e69836f9d/review-obsidian-worktree-pr-lifecycle.md) (**PASS**, zero unplanned changes)

---

## 1. Feature Summary & Motivation

Previously, `@coding-document` wrote documentation directly to the primary `*-obsidian/` working tree without Git worktree isolation, commits, PRs, or cleanup—causing dirty `main`/`master` trees and mixing concurrent or multi-phase documentation edits. This feature makes `@coding-document` self-manage the complete `*-obsidian/` Git worktree, branch (`docs/<slug>`), incremental commit, push, PR creation, Hybrid PR Waiting Gate, base sync, and cleanup lifecycle, while `@coding` coordinates multi-phase accumulation via `defer_pr`.

---

## 2. Functional Requirements

- **FR-1 (Self-Managed Ownership)**: `@coding-document` self-manages the entire `*-obsidian/` Git lifecycle; `@coding-merge` remains scoped to code repositories exclusively.
- **FR-2 (Worktree Isolation & Branching)**: `<vault_root>` remains on `main`/`master`; worktree at `<vault_root>/.worktrees/<slug>` on branch `docs/<slug>`; `.worktrees/` ignored via `.gitignore` or `.git/info/exclude`; `write_file` targets `<vault_root>/.worktrees/<slug>/...` (except Non-Git Fallback).
- **FR-3 (Multi-Phase `@coding` Accumulation — `defer_pr`)**: `@coding` passes consistent `slug` and `defer_pr=true` on Post-Plan and Pre-Merge hooks (commit incrementally, keep worktree, no push/PR), and `defer_pr=false` on Post-Merge (commit final status/changelog and execute Release & Cleanup).
- **FR-4 (Single-Shot Callers)**: `@coding-design`, `@coding-investigate`, `@coding-research`, `@coding-monitor`, and direct user invocations default to `defer_pr=false` and execute full Release & Cleanup immediately.
- **FR-5 (Release & Cleanup Strategies)**:
  1. **Remote (`origin` configured)**: `git push -u origin docs/<slug>`, `gh pr create` (or compare URL fallback), non-blocking **Hybrid PR Waiting Gate** (`gh pr view`), and **Post-Merge Base Sync & Cleanup** (`checkout <base>`, `pull --ff-only`, `worktree remove` + `prune`, idempotent `push origin --delete`, `branch -d`, `fetch --prune`, verify current branch).
  2. **Local-Only (no remote)**: `checkout <base>`, `merge docs/<slug>`, `worktree remove` + `prune`, `branch -d docs/<slug>`.
  3. **Non-Git Fallback**: Emit warning and write directly to `<vault_root>/...`.
- **FR-6 (Dual-Mode Question Relay)**: In Subagent Mode (`caller_id` present), `@coding-document` relays all prompts via `[QUESTION_RELAY_REQUEST]` / `[QUESTION_RELAY_RESPONSE]` (forwarded by `@coding`); in Standalone Mode, `@coding-document` calls `ask_question` directly.

---

## 3. Non-Functional Requirements, Edge Cases & Acceptance Criteria

- **NFR-1 (Non-Blocking Safety)**: Never `sleep` or poll in a synchronous loop waiting for PR reviews/CI.
- **NFR-2 (Exclusive Code-Change Authority)**: `edit_file` remains prohibited on `@coding-document`; `run_command` is scoped to read-only code-repo inspection, `scripts/sync_sessions.py`, and `*-obsidian/` git/PR lifecycle commands.
- **NFR-3 (Backward Compatibility)**: All existing tests in `agentic-project-documenting-plugin` (11 tests) and `agentic-coding-plugin` (27 tests) continue to pass alongside new assertions.
- **Key Edge Cases**: No-op commits/empty branches (skip empty commit, clean up if `defer_pr=false` and 0 commits ahead); "Proceed without waiting" on OPEN PR (keep worktree/branch intact, return `pending_approval: true`); dirty `<vault_root>` before pull/merge (prompt/report without discarding); already-deleted remote branch (`ls-remote` check).
- **Acceptance Criteria**: Updated specs in `agentic-project-documenting-plugin` (6 files) and `agentic-coding-plugin` (4 files), with `uv run pytest` passing 100% in both repositories.

---

## 4. Implementation, QA Review & Release Verification

The implementation and QA review ([`review-obsidian-worktree-pr-lifecycle.md`](file:///Users/gpfister/.gemini/antigravity/brain/e6771348-445c-4880-88aa-179e69836f9d/review-obsidian-worktree-pr-lifecycle.md)) passed all checks and both repositories have been merged and cleaned up:

- **`agentic-project-documenting-plugin`**:
  - Merged into `main` via [PR #3](https://github.com/gpfister/agentic-project-documenting-plugin/pull/3) at `bb6ce00` (`feat(document): add Obsidian git worktree, defer_pr, PR lifecycle, and Question Relay (#3)`).
  - Modified planned files (`README.md`, `agents/agent.md`, `agents/coding-document/agent.md`, `rules/AGENTS.md`, `skills/coding-document/SKILL.md`, `tests/test_plugin.py`) with **0 unplanned changes**.
  - Unit tests: **15 passed, 0 failed** (`uv run pytest -v`).
  - Post-merge cleanup: `.worktrees/obsidian-worktree-pr-lifecycle` removed and pruned; `feature/obsidian-worktree-pr-lifecycle` deleted locally and remotely; primary repository clean on `main` (`bb6ce00`).
- **`agentic-coding-plugin`**:
  - Merged into `main` via [PR #8](https://github.com/gpfister/agentic-coding-plugin/pull/8) at `b843fbe` (`feat: coordinate @coding-document worktree defer_pr lifecycle and question relay (#8)`).
  - Modified planned files (`agents/coding/agent.md`, `rules/AGENTS.md`, `skills/coding-orchestrator/SKILL.md`, `tests/test_plugin.py`) with **0 unplanned changes**.
  - Unit tests: **28 passed, 0 failed** (`uv run pytest -v`).
  - Post-merge cleanup: `.worktrees/obsidian-worktree-pr-lifecycle` removed and pruned; `feature/obsidian-worktree-pr-lifecycle` deleted locally and remotely; primary repository clean on `main` (`b843fbe`).
- **Local Deployment**: Not required for either repository.
