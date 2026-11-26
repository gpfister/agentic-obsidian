---
title: "agentic-design-plugin GitHub Remote Setup & Working Tree Finalization"
date: "2026-11-26"
status: "Ready to Merge (Pre-Merge)"
phase: "Phase 6: Review & Pre-Merge"
slug: "remove-mcp-config-and-publish"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/f9451afa-521e-429c-9a1e-11830bd1ce85/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/33bab1a3-fa8f-4437-85a7-49f372d0dffc/plan.md"
review_path: "/Users/gpfister/.gemini/antigravity/brain/4703cbed-c182-4bbd-a87f-f468d5b6cfde/review-remove-mcp-config-and-publish.md"
tags:
  - feature
  - agentic-design-plugin
  - coding-design
  - penpot
  - mcp
  - github-publication
  - review-approved
  - ready-to-merge
affected_plugins:
  - agentic-design-plugin
aliases:
  - remove-mcp-config-and-publish
---

# Feature: `agentic-design-plugin` GitHub Remote Setup & Working Tree Finalization

> [!NOTE] Lifecycle Status: Phase 6 — Review Approved & Ready to Merge (Pre-Merge Documentation)
> - **Approved Specification**: [specification.md](file:///Users/gpfister/.gemini/antigravity/brain/f9451afa-521e-429c-9a1e-11830bd1ce85/specification.md)
> - **Approved Plan**: [plan.md](file:///Users/gpfister/.gemini/antigravity/brain/33bab1a3-fa8f-4437-85a7-49f372d0dffc/plan.md)
> - **Code Review Report**: [review-remove-mcp-config-and-publish.md](file:///Users/gpfister/.gemini/antigravity/brain/4703cbed-c182-4bbd-a87f-f468d5b6cfde/review-remove-mcp-config-and-publish.md)
> - **Develop Commit**: `3fa6b0f` on `task/remove-mcp-config-and-publish` (`7/7` `uv run pytest -v` passed)
> - **Architecture Plan**: [[architecture/remove-mcp-config-and-publish|Architecture & Implementation Plan]]
> - **Plugin Spec**: [[plugins/agentic-design-plugin|agentic-design-plugin]]
> - **Related ADR**: [[architecture/adr-013-coding-design-plugin-and-penpot-mcp|ADR-013: Coding Design Plugin (@coding-design) with Official PenPot MCP Integration]]

---

## 1. Summary & Motivation

The [`agentic-design-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-design-plugin) repository is currently initialized locally on `main` with an unstaged deletion of `mcp_config.json`, stale references to `mcp_config.json` in [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-design-plugin/README.md#L19-L36), and no configured `origin` remote.

This task finalizes the working tree and publishes the repository to GitHub:
1. Commit the removal of `mcp_config.json`, update [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-design-plugin/README.md) to document external PenPot MCP configuration (`PENPOT_MCP_URL`), and align [`tests/test_plugin.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-design-plugin/tests/test_plugin.py).
2. Inspect `gpfister/agentic-coding-plugin` via `gh` to mirror its visibility and merge/repository settings.
3. Create `gpfister/agentic-design-plugin` on GitHub using the description from [`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-design-plugin/plugin.json#L3), configure `origin` (`git@github.com:gpfister/agentic-design-plugin.git`), and push `main` with upstream tracking.

---

## 2. Functional Requirements

| ID | Requirement | Target |
| :--- | :--- | :--- |
| **FR-1** | Stage/commit `mcp_config.json` deletion, update `README.md` (`## PenPot MCP Configuration`), update `tests/test_plugin.py`, and verify `uv run pytest`. | `agentic-design-plugin` |
| **FR-2** | Dynamically inspect `gpfister/agentic-coding-plugin` via `gh repo view` / `gh api` for visibility, merge settings, and repo features. | `gpfister/agentic-coding-plugin` (read-only) |
| **FR-3** | Create `gpfister/agentic-design-plugin` on GitHub with `plugin.json` description and mirrored visibility/merge settings. | GitHub (`gpfister/agentic-design-plugin`) |
| **FR-4** | Configure `origin` (`git@github.com:gpfister/agentic-design-plugin.git`), push `main` (`git push -u origin main`), and verify tracking & clean status. | `agentic-design-plugin` |
| **FR-5** | Keep parent workspace [`Makefile`](file:///Users/gpfister/Development/Projects/gpfister/agentic/Makefile) and sibling repositories untouched; do not create `.gitmodules`. | `agentic` (no changes) |

---

## 3. Affected Repositories

| Repository | Path | Impact |
| :--- | :--- | :--- |
| `agentic-design-plugin` | `/Users/gpfister/Development/Projects/gpfister/agentic/agentic-design-plugin` | Remove `mcp_config.json`, update `README.md` & `tests/test_plugin.py`, configure `origin`, create GitHub repo, push `main`. |
| Parent workspace (`agentic`) | `/Users/gpfister/Development/Projects/gpfister/agentic` | **None** (`Makefile` already lists `agentic-design-plugin`). |
| `agentic-coding-plugin` | `gpfister/agentic-coding-plugin` (GitHub) | **Read-only** inspection via `gh` to mirror settings. |

---

## 4. Acceptance Criteria

- [ ] **AC-1**: `mcp_config.json` deletion is committed on `main` in `agentic-design-plugin` (`git status` clean) — *committed in `3fa6b0f` on `task/remove-mcp-config-and-publish`; pending Stage 4 merge to `main`*.
- [x] **AC-2**: `README.md` no longer links to or implies a bundled `mcp_config.json` and accurately documents external PenPot MCP configuration.
- [x] **AC-3**: `uv run pytest` passes in `agentic-design-plugin` with 0 failures (`7/7` passed in `0.02s`).
- [ ] **AC-4**: `gpfister/agentic-design-plugin` exists on GitHub with `plugin.json` description and visibility/merge settings matching `gpfister/agentic-coding-plugin` — *scheduled for Stage 4 (`@coding-merge`)*.
- [ ] **AC-5**: `git remote -v` shows `origin` set to `git@github.com:gpfister/agentic-design-plugin.git` — *scheduled for Stage 4 (`@coding-merge`)*.
- [ ] **AC-6**: `git branch -vv` confirms `main` tracks `origin/main` and is up to date — *scheduled for Stage 4 (`@coding-merge`)*.
- [x] **AC-7**: Parent workspace `Makefile` and repository layout remain untouched (verified clean in review).

---

## 5. Pre-Merge Review & Verification

- **Worktree & Branch**: `.worktrees/remove-mcp-config-and-publish` on `task/remove-mcp-config-and-publish` (commit `3fa6b0f` on top of `main` at `9b434b4`), with primary repository remaining on `main`.
- **Plan vs. Actual Audit**: All three file changes (`mcp_config.json` deleted, `README.md` updated for external Antigravity MCP & `PENPOT_MCP_URL`, `tests/test_plugin.py` updated to `test_mcp_config_not_bundled`) strictly match `plan.md` with **zero unplanned changes**.
- **Test Results**: `uv run pytest -v` passed **7/7** in `0.02s` (`0` failed, `0` skipped).
- **Local Deployment**: Not Required (`coding-deploy` skipped).
- **Next Step**: Stage 4 (`@coding-merge`) to restore unstaged `D mcp_config.json` on `main`, merge `task/remove-mcp-config-and-publish`, clean up worktree/branch, create `gpfister/agentic-design-plugin` on GitHub, and push `main` with upstream tracking.
