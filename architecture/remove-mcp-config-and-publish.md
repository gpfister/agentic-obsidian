---
title: "Architecture & Implementation Plan: agentic-design-plugin Cleanup & GitHub Publication"
date: "2026-11-26"
status: "Merged & Released"
slug: "remove-mcp-config-and-publish"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/f9451afa-521e-429c-9a1e-11830bd1ce85/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/33bab1a3-fa8f-4437-85a7-49f372d0dffc/plan.md"
review_path: "/Users/gpfister/.gemini/antigravity/brain/4703cbed-c182-4bbd-a87f-f468d5b6cfde/review-remove-mcp-config-and-publish.md"
tags:
  - architecture
  - implementation-plan
  - agentic-design-plugin
  - worktree-isolation
  - github-publication
  - merged
---

# Architecture & Implementation Plan: `agentic-design-plugin` Cleanup & GitHub Publication

> [!NOTE]
> - **Feature Note**: [[features/remove-mcp-config-and-publish|agentic-design-plugin GitHub Remote Setup & Working Tree Finalization]]
> - **Approved Specification**: [specification.md](file:///Users/gpfister/.gemini/antigravity/brain/f9451afa-521e-429c-9a1e-11830bd1ce85/specification.md)
> - **Approved Plan**: [plan.md](file:///Users/gpfister/.gemini/antigravity/brain/33bab1a3-fa8f-4437-85a7-49f372d0dffc/plan.md)
> - **Code Review Report**: [review-remove-mcp-config-and-publish.md](file:///Users/gpfister/.gemini/antigravity/brain/4703cbed-c182-4bbd-a87f-f468d5b6cfde/review-remove-mcp-config-and-publish.md)
> - **Related Decision Record**: [[architecture/adr-013-coding-design-plugin-and-penpot-mcp|ADR-013]]

---

## 1. Architectural Impact

### Externalized PenPot MCP Configuration
Originally, [[architecture/adr-013-coding-design-plugin-and-penpot-mcp|ADR-013]] bundled a portable `mcp_config.json` inside `agentic-design-plugin/`. Removing `mcp_config.json` shifts PenPot MCP server registration to the user's host Antigravity MCP configuration while retaining `PENPOT_MCP_URL` (defaulting to `http://localhost:4401/mcp`) as the documented endpoint contract in [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-design-plugin/README.md).

### Worktree Isolation & Release Scope
- **Target Repository**: [`agentic-design-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-design-plugin) (`gpfister/agentic-design-plugin`)
- **Worktree & Merge**: `.worktrees/remove-mcp-config-and-publish` on `task/remove-mcp-config-and-publish` (`3fa6b0f`) fast-forward merged into `main`, worktree removed & pruned, and local branch deleted.
- **GitHub Publication**: Created [`https://github.com/gpfister/agentic-design-plugin`](https://github.com/gpfister/agentic-design-plugin) (`PUBLIC`) mirroring `gpfister/agentic-coding-plugin` (`allow_squash_merge=true`, `allow_merge_commit=false`, `allow_rebase_merge=false`, `allow_auto_merge=true`, `delete_branch_on_merge=true`, `allow_update_branch=true`, `has_issues/projects/wiki/discussions=false`), configured `origin` (`git@github.com:gpfister/agentic-design-plugin.git`), and pushed `main` (`3fa6b0f`).
- **Local Deployment**: **Not Required** (no `Dockerfile`, `compose.yaml`, local deployment scripts, Terraform/K8s manifests, or `Makefile` in `agentic-design-plugin`). `coding-deploy` skipped.

---

## 2. Execution Workflow

```mermaid
flowchart LR
    D["Stage 1: @coding-develop (Completed: 3fa6b0f)\nWorktree task/remove-mcp-config-and-publish\n- git rm -f mcp_config.json\n- Update README.md & tests/test_plugin.py\n- 7/7 uv run pytest -v passed"] --> R["Stage 3: @coding-review (Approved)\n- Plan vs. actual audit (0 unplanned changes)\n- 7/7 uv run pytest -v passed\n- Parent Makefile untouched"]
    R --> M["Stage 4: @coding-merge (Completed)\n- Fast-forward merged 3fa6b0f into main\n- Removed worktree & deleted branch\n- Created PUBLIC gpfister/agentic-design-plugin\n- Added origin & pushed main (3fa6b0f)"]
```

### Stage Breakdown
1. **Stage 1 (`@coding-develop`) — Completed (`3fa6b0f`)**:
   - Created `.worktrees/remove-mcp-config-and-publish` on `task/remove-mcp-config-and-publish`.
   - Removed `mcp_config.json` (`git rm -f mcp_config.json`).
   - Updated `README.md` (`## PenPot MCP Configuration`) to document external Antigravity MCP configuration, `PENPOT_MCP_URL` (`http://localhost:4401/mcp`), and stdio (`npx`) alternative.
   - Replaced `test_mcp_config_valid_and_portable` with `test_mcp_config_not_bundled` in `tests/test_plugin.py`.
   - Verified `uv run pytest -v` (`7/7` passed) and committed `3fa6b0f`.
2. **Stage 2 (`@coding-deploy`) — Skipped**:
   - `Local Deployment: Not Required`.
3. **Stage 3 (`@coding-review`) — Approved**:
   - Audited `git status` and `git diff main..task/remove-mcp-config-and-publish`: zero unplanned changes, primary repo on `main`, parent `Makefile` and sibling repositories clean.
   - Verified `uv run pytest -v`: **7 passed in 0.02s** (`0` failed, `0` skipped).
4. **Stage 4 (`@coding-merge`) — Completed**:
   - Fast-forward merged `task/remove-mcp-config-and-publish` (`3fa6b0f`) into `main`, removed & pruned `.worktrees/remove-mcp-config-and-publish`, and deleted `task/remove-mcp-config-and-publish`.
   - Inspected `gpfister/agentic-coding-plugin` via `gh` and created `https://github.com/gpfister/agentic-design-plugin` (`PUBLIC`) with mirrored settings.
   - Configured `origin` (`git@github.com:gpfister/agentic-design-plugin.git`), pushed `main` (`git push -u origin main`), and verified clean working tree tracking `origin/main`.

---

## 3. Verification Matrix

| Phase | Command / Check | Status / Outcome |
| :--- | :--- | :--- |
| Development / Review | `uv run pytest -v` | **Passed** (`7/7` tests in `tests/test_plugin.py` passed in `0.02s`) |
| Merge / Publish | `gh repo view gpfister/agentic-design-plugin` | **Verified** (`PUBLIC`, mirrored settings from `gpfister/agentic-coding-plugin`) |
| Merge / Publish | `git remote -v && git branch -vv && git status` | **Verified** (`main...origin/main` at `3fa6b0f`, working tree clean) |
