---
title: "Worktree-Based Development & Exclusive Code-Change Authority Architecture Plan"
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
  - architecture
  - implementation-plan
  - git-worktree
  - coding-develop
  - anti-direct-coding
  - approved-plan
  - phase-3-completed
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

# Worktree-Based Development & Exclusive Code-Change Authority Architecture Plan

> [!NOTE] Lifecycle Status: Merged / Completed — `PHASE 7 COMPLETED`
> - **Architecture Decision Records**:
>   - [[architecture/adr-011-worktree-based-development-lifecycle|ADR-011: Worktree-Based Development Lifecycle (.worktrees/<slug> Isolation)]]
>   - [[architecture/adr-012-exclusive-code-change-authority|ADR-012: Exclusive Code-Change Authority for @coding-develop]]
> - **Feature Specification Note**: [[features/worktree-based-development-and-exclusive-code-change-authority|Feature: Worktree-Based Development & Exclusive Code-Change Authority]]
> - **Related Worktree Architecture**: [[architecture/worktrees|Parallel Agent Sessions with Git Worktrees]]
> - **Target Repositories (7)**: `agentic-coding-plugin`, `agentic-project-grooming-plugin`, `agentic-project-documenting-plugin`, `agentic-issue-investigation-plugin`, `agentic-task-monitoring-plugin`, `agentic-topic-researching-plugin`, `agentic` (root)
> - **Isolation Worktree / Branch**: `<repo_root>/.worktrees/worktree-exclusive-code-authority` on `feature/worktree-exclusive-code-authority`
> - **Local Deployment (`coding-deploy`)**: Not Required across all 7 repositories
> - **Approved Spec**: [`specification.md`](file:///Users/gpfister/.gemini/antigravity/brain/5e833364-2100-4388-bb5e-9dc4468830ae/specification.md)
> - **Approved Plan**: [`plan.md`](file:///Users/gpfister/.gemini/antigravity/brain/5e833364-2100-4388-bb5e-9dc4468830ae/plan.md)
> - **Quality Review**: [`review-worktree-exclusive-code-authority.md`](file:///Users/gpfister/.gemini/antigravity/brain/5e833364-2100-4388-bb5e-9dc4468830ae/review-worktree-exclusive-code-authority.md)

---

## 1. Overview & Execution Strategy

This architecture plan implements two structural invariants across all **7 affected repositories** in the `agentic` workspace:
1. **Worktree-Based Development (`FR-1`, [[architecture/adr-011-worktree-based-development-lifecycle|ADR-011]])**: Primary repository working trees remain strictly on `main`/`master`. `@coding-develop` creates/reuses `<repo_root>/.worktrees/<slug>` on `feature/<slug>`, `task/<slug>`, or `bugfix/<slug>`, performs all edits inside `worktree_path`, and reports `worktree_path` to `@coding`, which propagates it to `@coding-deploy`, `@coding-review`, and `@coding-merge` (for post-merge `git worktree remove` & `git worktree prune`).
2. **Exclusive Code-Change Authority (`FR-2`, [[architecture/adr-012-exclusive-code-change-authority|ADR-012]])**: `@coding-develop` is the sole agent authorized to declare `edit_file` and modify repository source code, configuration files, test suites, or repo-local docs.

> [!IMPORTANT] Worktree Isolation for This Implementation
> During `@coding-develop` execution for this plan, each of the 7 affected repositories MUST remain checked out on `main` in its primary working tree. All edits MUST be performed inside `<repo_root>/.worktrees/worktree-exclusive-code-authority` on branch `feature/worktree-exclusive-code-authority`.

---

## 2. Per-Repository Work Breakdown

### Repository 1: [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin)
- **Branch**: `feature/worktree-exclusive-code-authority`
- **Worktree**: `agentic-coding-plugin/.worktrees/worktree-exclusive-code-authority`
- **Files to Modify**:
  1. [`agents/coding-develop/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-develop/agent.md): Add Worktree Isolation protocol (`main`/`master` invariant, `git worktree list` / `prune` / `add`, `.worktrees/` ignore check, edits strictly in `worktree_path`, return `worktree_path` + `branch` in `DevelopReport`).
  2. [`agents/coding-deploy/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-deploy/agent.md): Remove `edit_file` from `tools:`; restrict `write_file` to `deployment-issue*.md` and `run_command` to operational deployment/health checks; target `worktree_path`.
  3. [`agents/coding-review/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-review/agent.md): Target `worktree_path` for `git status` / `git diff`, tests, and linters.
  4. [`agents/coding-merge/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-merge/agent.md): Accept `worktree_path`; merge `<branch>` into `<base>` in primary repo; clean up via `git worktree remove <worktree_path>` (or `--force` post-merge if untracked artifacts remain) + `git worktree prune`.
  5. [`agents/coding/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding/agent.md): Update Stage 4–7 instructions and Mermaid diagram to capture and propagate `worktree_path`; reinforce anti-direct-coding guardrails.
  6. [`agents/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/agent.md): Remove `edit_file` from `@coding-deploy` in the agent registry table; document worktree targeting and exclusive `@coding-develop` authority.
  7. [`skills/coding-orchestrator/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/coding-orchestrator/SKILL.md): Document `.worktrees/<slug>` lifecycle, `worktree_path` propagation, and exclusive `@coding-develop` code-change authority.
  8. [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/rules/AGENTS.md): Codify primary-repo-on-main invariant, `.worktrees/<slug>` isolation, and exclusive `@coding-develop` code-change authority.

### Repository 2: [`agentic-project-grooming-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin)
- **Branch**: `feature/worktree-exclusive-code-authority`
- **Worktree**: `agentic-project-grooming-plugin/.worktrees/worktree-exclusive-code-authority`
- **Files to Modify**:
  1. [`skills/project-grooming/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/skills/project-grooming/SKILL.md): Update Step 4 fallback (when `agentic-coding-plugin` or `agentic-project-documenting-plugin` is absent) to output manual scaffolding guidance/templates instead of writing repository files directly.
  2. [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/rules/AGENTS.md): Require manual scaffolding guidance in fallback; restrict `write_file` to `Project_Request_Grooming.md` and `run_command` to read-only inspection.
  3. [`agents/coding-groom/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/agents/coding-groom/agent.md): Verify `edit_file` is absent; document artifact-only `write_file` and read-only `run_command` guardrails.

### Repository 3: [`agentic-project-documenting-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin)
- **Branch**: `feature/worktree-exclusive-code-authority`
- **Worktree**: `agentic-project-documenting-plugin/.worktrees/worktree-exclusive-code-authority`
- **Files to Modify**:
  1. [`agents/coding-document/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/agents/coding-document/agent.md): Remove `edit_file` from `tools:` and prose; restrict `write_file` strictly to `*-obsidian/` vault documentation and session artifacts (delegating repo-local docs like `README.md`/`CONTRIBUTING.md` to `@coding-develop`); scope `run_command` to read-only git inspection and `scripts/sync_sessions.py`.
  2. [`agents/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/agents/agent.md): Remove `edit_file` from `@coding-document`'s tools column.
  3. [`skills/coding-document/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/skills/coding-document/SKILL.md): Align skill instructions with `write_file` restricted to `*-obsidian/` and session artifacts.
  4. [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/rules/AGENTS.md): Update Section 2 to remove `edit_file` and restrict `write_file` strictly to `*-obsidian/` and session artifacts.

### Repository 4: [`agentic-issue-investigation-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin)
- **Branch**: `feature/worktree-exclusive-code-authority`
- **Worktree**: `agentic-issue-investigation-plugin/.worktrees/worktree-exclusive-code-authority`
- **Files to Modify**:
  1. [`agents/coding-investigate/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/agents/coding-investigate/agent.md): Restrict `write_file` to `issue-assessment.md` and `run_command` to strictly read-only.
  2. [`skills/coding-investigate/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/skills/coding-investigate/SKILL.md): Reinforce exclusive `@coding-develop` code-change authority via `@coding` handoff.
  3. [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/rules/AGENTS.md): Explicitly prohibit `edit_file`, restrict `write_file` to `issue-assessment.md`, and restrict `run_command` to read-only.

### Repository 5: [`agentic-task-monitoring-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-task-monitoring-plugin)
- **Branch**: `feature/worktree-exclusive-code-authority`
- **Worktree**: `agentic-task-monitoring-plugin/.worktrees/worktree-exclusive-code-authority`
- **Files to Modify**:
  1. [`agents/coding-monitor/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-task-monitoring-plugin/agents/coding-monitor/agent.md): Enforce non-destructive read/tail `run_command` scope and prohibit modifying repository source, configs, or tests.
  2. [`skills/coding-monitor/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-task-monitoring-plugin/skills/coding-monitor/SKILL.md): Reinforce read-only surveillance.
  3. [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-task-monitoring-plugin/rules/AGENTS.md): Prohibit `edit_file` and `write_file` for repository code/config/tests; restrict `run_command` to non-destructive observation.

### Repository 6: [`agentic-topic-researching-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-topic-researching-plugin)
- **Branch**: `feature/worktree-exclusive-code-authority`
- **Worktree**: `agentic-topic-researching-plugin/.worktrees/worktree-exclusive-code-authority`
- **Files to Modify**:
  1. [`agents/coding-research/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-topic-researching-plugin/agents/coding-research/agent.md): Restrict `write_file` to `research-report.md` and `run_command` to strictly read-only.
  2. [`skills/coding-research/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-topic-researching-plugin/skills/coding-research/SKILL.md): Reinforce artifact-only `write_file` (`research-report.md`) and prohibition on repository edits.
  3. [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-topic-researching-plugin/rules/AGENTS.md): Restrict `write_file` to `research-report.md` (with Obsidian vault notes delegated to `@coding-document`), and prohibit `edit_file`.

### Repository 7: [`agentic`](file:///Users/gpfister/Development/Projects/gpfister/agentic) (Root Workspace Repository)
- **Branch**: `feature/worktree-exclusive-code-authority`
- **Worktree**: `agentic/.worktrees/worktree-exclusive-code-authority`
- **Files to Modify**:
  1. [`AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/AGENTS.md): Update Section 2.3 and Section 3.1 (`/project-grooming` fallback) to document the `.worktrees/<slug>` lifecycle and exclusive `@coding-develop` code-change authority.
  2. [`.gitignore`](file:///Users/gpfister/Development/Projects/gpfister/agentic/.gitignore): Ensure `.worktrees/` is ignored in the root repository.

---

## 3. Global Verification Commands

```bash
grep -rn "edit_file" */agents/
```
Expected result: Matches **only** in `agentic-coding-plugin/agents/coding-develop/agent.md` (or `.worktrees/worktree-exclusive-code-authority/agents/coding-develop/agent.md`).
