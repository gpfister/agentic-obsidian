---
title: "Worktree-Based Development & Exclusive Code-Change Authority"
date: "2026-11-18"
status: "Merged / Completed"
phase: "Phase 7: Merge & Release Complete"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/5e833364-2100-4388-bb5e-9dc4468830ae/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/5e833364-2100-4388-bb5e-9dc4468830ae/plan.md"
review_path: "/Users/gpfister/.gemini/antigravity/brain/5e833364-2100-4388-bb5e-9dc4468830ae/review-worktree-exclusive-code-authority.md"
adrs:
  - "[[architecture/adr-011-worktree-based-development-lifecycle|ADR-011: Worktree-Based Development Lifecycle]]"
  - "[[architecture/adr-012-exclusive-code-change-authority|ADR-012: Exclusive Code-Change Authority]]"
tags:
  - feature
  - git-worktree
  - coding-develop
  - anti-direct-coding
  - specification
  - approved-spec
  - implementation-plan
  - approved-plan
  - phase-1-completed
  - phase-2-completed
  - phase-3-completed
  - phase-4-completed
  - phase-6-completed
  - phase-7-completed
  - merged
  - completed
affected_plugins:
  - agentic-coding-plugin
  - agentic-project-grooming-plugin
  - agentic-project-documenting-plugin
  - agentic-issue-investigation-plugin
  - agentic-task-monitoring-plugin
  - agentic-topic-researching-plugin
  - agentic
aliases:
  - worktree-based-development-and-exclusive-code-change-authority
---

# Feature: Worktree-Based Development & Exclusive Code-Change Authority

> [!NOTE] Lifecycle Status: Merged / Completed — `PHASE 7 COMPLETED`
> - **Architecture Decision Records**:
>   - [[architecture/adr-011-worktree-based-development-lifecycle|ADR-011: Worktree-Based Development Lifecycle (.worktrees/<slug> Isolation)]]
>   - [[architecture/adr-012-exclusive-code-change-authority|ADR-012: Exclusive Code-Change Authority for @coding-develop]]
> - **Architecture Plan Note**: [[architecture/worktree-based-development-and-exclusive-code-change-authority|Worktree-Based Development & Exclusive Code-Change Authority Architecture Plan]]
> - **Affected Repositories**: `agentic-coding-plugin`, `agentic-project-grooming-plugin`, `agentic-project-documenting-plugin`, `agentic-issue-investigation-plugin`, `agentic-task-monitoring-plugin`, `agentic-topic-researching-plugin`, `agentic` (root)
> - **Current Phase**: Phase 7: Merge & Release Complete (`Merged / Completed`)
> - **Approved Spec**: [`specification.md`](file:///Users/gpfister/.gemini/antigravity/brain/5e833364-2100-4388-bb5e-9dc4468830ae/specification.md)
> - **Approved Plan**: [`plan.md`](file:///Users/gpfister/.gemini/antigravity/brain/5e833364-2100-4388-bb5e-9dc4468830ae/plan.md)
> - **Quality Review**: [`review-worktree-exclusive-code-authority.md`](file:///Users/gpfister/.gemini/antigravity/brain/5e833364-2100-4388-bb5e-9dc4468830ae/review-worktree-exclusive-code-authority.md)

---

## 1. Feature Summary & Motivation

This feature establishes two structural invariants across the `agentic` plugin ecosystem:
1. **Worktree-Based Development**: All code, test, configuration, and repository-local documentation changes executed by `@coding-develop` occur inside an isolated Git worktree (`.worktrees/<slug>`) rather than the primary repository working tree, which remains strictly on `main` or `master`. Downstream lifecycle stages (`@coding-deploy`, `@coding-review`, `@coding-merge`) target that worktree and clean it up upon merge.
2. **Exclusive Code-Change Authority**: `@coding-develop` is the sole agent across the entire workspace authorized to modify repository source code, configuration files, and test suites. All other `agentic-coding-plugin` subagents and companion plugin agents are restricted from code-editing tools and instructions.

---

## 2. Functional Requirements

### FR-1: Worktree-Based Development (`@coding-develop` & Lifecycle)
1. **Regular Repository Invariant**: Primary repository working trees MUST remain on `main` or `master` at all times during development, deployment, and review.
2. **Worktree Creation & Execution (`@coding-develop`)**:
   - Check `git worktree list`; run `git worktree prune` if stale; create or reuse `<repo_root>/.worktrees/<slug>` via `git worktree add` on `feature/<slug>`, `task/<slug>`, or `bugfix/<slug>`.
   - Ensure `.worktrees/` is ignored via `.gitignore` or `.git/info/exclude`.
   - Execute all edits, tests, linters, and repo-local doc edits exclusively inside `<worktree_path>`.
   - Return `worktree_path` (absolute path) and `branch` in `DevelopReport`.
3. **Downstream Worktree Targeting**:
   - `@coding-deploy`: Runs local deployments, builds, and health checks against `worktree_path`.
   - `@coding-review`: Runs `git status` / `git diff`, tests, and linters inside `worktree_path`.
   - `@coding-merge`: Merges/creates PR in primary repo, removes worktree (`git worktree remove <worktree_path>` / `--force` post-merge if needed), runs `git worktree prune`, and cleans up branches.
4. **Orchestrator Propagation (`@coding`)**: Captures `worktree_path` from `DevelopReport` and passes it explicitly to `@coding-deploy`, `@coding-review`, and `@coding-merge`.

### FR-2: Exclusive Code-Change Authority (`@coding-develop` Only)
1. **Scope**: Applies to all `agentic-coding-plugin` agents (`@coding`, `@coding-clarify`, `@coding-specify`, `@coding-plan`, `@coding-deploy`, `@coding-review`, `@coding-merge`) and all companion plugin agents (`@coding-investigate`, `@coding-document`, `@coding-monitor`, `@coding-research`, `/project-grooming`).
2. **Tool Declaration Restrictions**: `edit_file` MUST be declared ONLY on `@coding-develop` (removed from `@coding-deploy`, `@coding-document`, etc.).
3. **`write_file` Restrictions**: Restricted strictly to session artifacts under `<appDataDir>/brain/<conversation-id>/*.md` (or `*-obsidian/` for `@coding-document` only).
4. **`run_command` Restrictions**: Strictly read-only or operational on all agents other than `@coding-develop` (and `@coding-merge` for git merge/branch/worktree lifecycle commands).
5. **`/project-grooming` Fallback**: Outputs manual scaffolding guidance (commands and file templates) instead of writing repository files directly when `agentic-coding-plugin` is absent.

---

## 3. Non-Functional Requirements, Edge Cases & Acceptance Criteria

- **Isolation & Concurrency**: Multiple tasks on the same repository use distinct `.worktrees/<slug>` directories without interfering with each other or `main`/`master`.
- **Edge Cases**:
  - Reuse existing `.worktrees/<slug>` or `git worktree prune` stale entries before `git worktree add`.
  - Ignore `.worktrees/` via `.gitignore` or `.git/info/exclude`.
  - Use `git worktree remove --force` only after merge confirmation if untracked build/test artifacts remain.
  - Multi-repo tasks create `.worktrees/<slug>` in each modified repository and report all `worktree_path` mappings.
- **Verifiable Acceptance Criteria**:
  1. Grepping `edit_file` across all `agents/` files in all plugin repositories returns matches **only** in `agentic-coding-plugin/agents/coding-develop/agent.md`.
  2. `agentic-coding-plugin` enforces `.worktrees/<slug>` lifecycle and `worktree_path` propagation.
  3. All 5 companion plugins enforce exclusive `@coding-develop` code-change authority and `/project-grooming` manual fallback.
  4. Root `agentic/AGENTS.md` documents the worktree-based development flow and exclusive `@coding-develop` authority.

---

## 4. Implementation & Review Verification (Pre-Merge)

All 7 repositories were implemented on branch `feature/worktree-exclusive-code-authority` inside isolated worktrees (`.worktrees/worktree-exclusive-code-authority`) while primary repositories remained on `main`:

| Repository | Commit | Verification |
| :--- | :--- | :--- |
| `agentic-coding-plugin` | `8cb89dd` | `uv run pytest`: **26 passed** (added `tests/test_plugin.py` assertions for exclusive `edit_file` and worktree lifecycle) |
| `agentic-project-grooming-plugin` | `c135bb1` | `uv run pytest`: **18 passed** (added `tests/test_plugin.py` assertions for manual fallback and `edit_file` absence) |
| `agentic-project-documenting-plugin` | `b7564ef` | `git status`, `git diff main...HEAD`, `grep -rn "edit_file"` |
| `agentic-issue-investigation-plugin` | `cf54db7` | `git status`, `git diff main...HEAD`, `grep -rn "edit_file"` |
| `agentic-task-monitoring-plugin` | `cec5751` | `uv run pytest`: **12 passed** (added `tests/test_plugin.py` assertions for `edit_file` absence and read-only boundaries) |
| `agentic-topic-researching-plugin` | `a45cdfa` | `git status`, `git diff main...HEAD`, `grep -rn "edit_file"` |
| `agentic` (root) | `68484f0` | `git status`, `git diff main...HEAD`, `grep -rn "edit_file" */.worktrees/worktree-exclusive-code-authority/agents/` |

---

## 5. Post-Merge Release & Cleanup

All 7 Pull Requests were merged into `main`, all 7 isolated worktrees (`.worktrees/worktree-exclusive-code-authority`) were removed and pruned, and all `feature/worktree-exclusive-code-authority` branches were deleted:

1. `agentic-coding-plugin`: [PR #5](https://github.com/gpfister/agentic-coding-plugin/pull/5) (`MERGED`)
2. `agentic-project-grooming-plugin`: [PR #1](https://github.com/gpfister/agentic-project-grooming-plugin/pull/1) (`MERGED`)
3. `agentic-project-documenting-plugin`: [PR #1](https://github.com/gpfister/agentic-project-documenting-plugin/pull/1) (`MERGED`)
4. `agentic-issue-investigation-plugin`: [PR #2](https://github.com/gpfister/agentic-issue-investigation-plugin/pull/2) (`MERGED`)
5. `agentic-task-monitoring-plugin`: [PR #1](https://github.com/gpfister/agentic-task-monitoring-plugin/pull/1) (`MERGED`)
6. `agentic-topic-researching-plugin`: [PR #1](https://github.com/gpfister/agentic-topic-researching-plugin/pull/1) (`MERGED`)
7. `agentic` (root): [PR #3](https://github.com/gpfister/agentic/pull/3) (`MERGED`)
