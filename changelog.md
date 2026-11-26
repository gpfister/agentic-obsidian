---
title: "Workspace Release Changelog"
date: "2026-11-26"
status: "active"
tags:
  - changelog
  - releases
  - multi-repo
  - coding-document
  - obsidian-vault
  - defer-pr
  - pull-request
  - agentic-prompt-triage-plugin
  - laya-router
  - system-1
  - semantic-classification
  - minigma-ai-web-app
  - spartan-ui
  - tailwind-v4
  - angular-19
  - agentic-coding-plugin
  - agentic-design-plugin
  - coding-design
  - change-request-loop
  - penpot
  - mcp
  - git-worktree
  - anti-direct-coding
  - coding-develop
  - dual-mode
  - multi-hop-relay
  - guardrails
  - coding-orchestrator
  - coding-review
  - plan-audit
  - unplanned-changes
  - question-relay
  - artifact-approval-relay
  - coding-merge
  - branch-cleanup
  - git-prune
  - base-branch-sync
---

# Workspace Release Changelog

Chronological log of cross-repository releases, feature merges, and architectural upgrades across the `agentic` workspace.

---

## [2026-11-26] — Obsidian Vault Git Worktree & PR Lifecycle (`@coding-document`)

> [!NOTE] Release Summary: `obsidian-worktree-pr-lifecycle` Merged & Released across 2 Repositories
> - **Architecture Decision**: [[architecture/adr-016-obsidian-vault-worktree-and-pr-lifecycle|ADR-016: Obsidian Vault Git Worktree & PR Lifecycle]]
> - **Feature Note**: [[features/obsidian-worktree-pr-lifecycle|Obsidian Vault Git Worktree & PR Lifecycle (@coding-document)]]
> - **Architecture Note**: [[architecture/obsidian-worktree-pr-lifecycle|Obsidian Vault Git Worktree & PR Lifecycle Architecture Plan]]
> - **Verification**: `15/15` automated tests passing in `agentic-project-documenting-plugin` (`uv run pytest -v`), `28/28` automated tests passing in `agentic-coding-plugin` (`uv run pytest -v`), QA review **PASS** with zero unplanned changes.
> - **Branch & Worktree Lifecycle**: Merged via GitHub [PR #3](https://github.com/gpfister/agentic-project-documenting-plugin/pull/3) (`bb6ce00`) and [PR #8](https://github.com/gpfister/agentic-coding-plugin/pull/8) (`b843fbe`) into `main`; `.worktrees/obsidian-worktree-pr-lifecycle` removed & pruned in both repositories; `feature/obsidian-worktree-pr-lifecycle` deleted locally & remotely; primary repositories clean on `main`.

### Overview & Key Capabilities Delivered
Enabled `@coding-document` to self-manage the complete `*-obsidian/` Git worktree, branch (`docs/<slug>`), incremental commit, push, PR creation, Hybrid PR Waiting Gate, base sync, and cleanup lifecycle, coordinated by `@coding` via `defer_pr`:
- **Self-Managed `*-obsidian/` Worktree Isolation**:
  - Primary `<vault_root>` remains on `<base>` (`main`/`master`); all documentation edits target `<vault_root>/.worktrees/<slug>` on `docs/<slug>`.
  - `@coding-merge` remains scoped exclusively to code repositories.
- **Multi-Phase `defer_pr` Accumulation (`@coding`)**:
  - `@coding` passes consistent `slug=<slug>` and `defer_pr=true` on Post-Plan and Pre-Merge hooks (incremental commits without push/PR/cleanup) and `defer_pr=false` on Post-Merge (final status/changelog commit + Release & Cleanup).
  - Single-shot callers (`@coding-design`, `@coding-investigate`, `@coding-research`, `@coding-monitor`, direct invocations) default to `defer_pr=false`.
- **Release & Cleanup Strategies & Dual-Mode Question Relay**:
  - Supports Remote Release (`git push`, `gh pr create` / compare URL fallback, non-blocking Hybrid PR Waiting Gate, Post-Merge Base Sync & Cleanup), Local-Only Release, and Non-Git Fallback.
  - `@coding-document` uses `[QUESTION_RELAY_REQUEST]` / `[QUESTION_RELAY_RESPONSE]` in Subagent Mode (`caller_id` present) and `ask_question` directly in Standalone Mode.

### Merged Repositories & Release Commits
- [`agentic-project-documenting-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin): [PR #3](https://github.com/gpfister/agentic-project-documenting-plugin/pull/3) merged into `main` at `bb6ce00` — `feat(document): add Obsidian git worktree, defer_pr, PR lifecycle, and Question Relay (#3)` (**15/15 tests passed**).
- [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin): [PR #8](https://github.com/gpfister/agentic-coding-plugin/pull/8) merged into `main` at `b843fbe` — `feat: coordinate @coding-document worktree defer_pr lifecycle and question relay (#8)` (**28/28 tests passed**).

---

## [2026-11-26] — `agentic-prompt-triage-plugin`: Semantic Laya Triage & Parallel `choice` + `noul` Decomposition (PR #2)

> [!NOTE] Release Summary: `agentic-prompt-triage-plugin` Semantic Laya Triage Merged & Released
> - **Architecture Decision**: [[architecture/adr-017-semantic-laya-triage-and-parallel-choice-noul-decomposition|ADR-017: Semantic Laya Triage & Parallel choice + noul Decomposition]]
> - **Feature Note**: [[features/semantic-laya-triage|Feature: Semantic Laya Triage & Parallel choice + noul Decomposition]]
> - **Architecture Note**: [[architecture/semantic-laya-triage|Semantic Laya Triage & Parallel choice + noul Decomposition Architecture Plan]]
> - **Plugin Spec**: [[plugins/agentic-prompt-triage-plugin|agentic-prompt-triage-plugin]]
> - **Verification**: `101/101` automated tests passing (`uv run pytest -v` in `3.96s`: `12` plugin tests, `89` prompt triage tests), CLI smoke test exited `0` (`{"injectSteps": []}`), empirical Laya evaluation (`scripts/eval_laya_triage.py`) selected `default` (`Router(preload=False)`) over `typed-decisions`, QA review **PASS** with zero unplanned changes.
> - **Branch & Worktree Lifecycle**: Merged via GitHub [PR #2](https://github.com/gpfister/agentic-prompt-triage-plugin/pull/2) (`feature/semantic-laya-triage` -> `main` at [`ff5a3ae`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin)); worktree `.worktrees/semantic-laya-triage` removed and pruned; `feature/semantic-laya-triage` deleted locally and remotely; primary repo clean on `main` (`ff5a3ae`).

### Overview & Key Capabilities Delivered
Refactored [`hooks/prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/hooks/prompt_triage.py) in [`agentic-prompt-triage-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin) to improve Laya Router System 1 zero-shot classification accuracy, calibration, and observability in a single forward pass:
- **Semantic `criteria` Keys & Action-Oriented Descriptions**:
  - Replaced opaque `response 1`..`response 9` keys in `build_dynamic_triage_spec` with semantic keys (`inquiry`, `coding`, `monitoring`, `investigation`, `research`, `documentation`, `grooming`, `design`) and removed `mixed` from `choice` criteria while preserving `response_to_intent["mixed"]` and `response_to_directive["mixed"]`.
- **Single-Pass Parallel `choice` + `noul` Decomposition**:
  - Evaluates `"about"` (`choice`), `"requires_changes"` (`noul`), and `"is_mixed"` (`noul`) in one `self._router.predict({"prompt": text}, questions)` call.
  - Gates mutating agents (`MUTATING_AGENTS = frozenset({"coding", "documentation", "grooming", "design"})`) against `requires_changes == "no"` and `inquiry` against `requires_changes == "yes"`, while exempting `NON_MUTATING_AGENTS` (`monitoring`, `investigation`, `research`).
- **Anchored Instructions, `usage` Diagnostics & Calibrated Confidence**:
  - Set `TRIAGE_QUESTION = "Which single agent or action should handle `prompt`?"` and anchored `noul` questions to `` `prompt` ``.
  - Added `_inspect_usage` to warn on `truncated` / `truncated_questions` and `options` collisions, and preferred calibrated `ans.get("confidence")`.
  - Benchmarked `default` (`Router(preload=False)`) vs `typed-decisions` via [`scripts/eval_laya_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/scripts/eval_laya_triage.py), confirming `default` as the superior router configuration.

### Merged Repositories & Release Commits
- [`agentic-prompt-triage-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin): [PR #2](https://github.com/gpfister/agentic-prompt-triage-plugin/pull/2) merged into `main` at `ff5a3ae` — `feat: semantic Laya triage with parallel choice + noul decomposition (#2)`.

---

## [2026-11-25] — `minigma-ai-web-app`: UI Alignment (`spartan/ui` + Tailwind CSS 4) & Screen Cleanup (PR #4)

> [!NOTE] Release Summary: `minigma-ai-web-app` UI Alignment & Screen Cleanup Merged & Released
> - **Architecture Decision**: [[architecture/adr-015-minigma-ai-web-app-spartan-ui-and-screen-cleanup|ADR-015: Minigma AI Web App: spartan/ui + Tailwind CSS 4 Migration, Dual-Theme Tokens, and Route Surface Reduction]]
> - **Feature Note**: [[features/minigma-ai-web-app-ui-alignment|Minigma AI Web App — UI Alignment & Screen Cleanup]]
> - **Architecture Note**: [[architecture/minigma-ai-web-app-ui-alignment|Architecture Plan — Minigma AI Web App UI Alignment & Screen Cleanup]]
> - **Design Spec**: [[designs/minigma-ai-web-app-ui-alignment|Design Spec — Minigma AI Web App UI Alignment & Screen Cleanup]]
> - **Project Note**: [[projects/minigma-ai-web-app|Minigma AI Web App (`minigma-ai-web-app`)]]
> - **Verification**: `npx tsc --noEmit` (0 errors), `npm run build` (`481.49 kB` initial), and active local Kubernetes deployment at `http://app.localhost:8080/` (`minigma-ai-web-app:94f1f67-1795625897` in namespace `minigma-ai-web-app`).
> - **Branch & Worktree Lifecycle**: Squash-merged via GitHub [PR #4](https://github.com/gpfister/minigma-ai-web-app/pull/4) (`feature/penpot-ui-alignment` -> `main` at [`49360d2`](file:///Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app)); worktree `.worktrees/penpot-ui-alignment` removed and pruned; local and remote `feature/penpot-ui-alignment` deleted; primary repo clean on `main` (`49360d2`).

### Overview & Key Capabilities Delivered
Aligned [`minigma-ai-web-app`](file:///Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app) (Angular `^19.0.0`) with the approved **spartan/ui + Tailwind CSS 4** PenPot design system (`42288968-6e5e-808d-8008-fab5dcc739f5`):
- **Tailwind CSS 4 & `spartan/ui` Migration**:
  - Phased out `@angular/material`, `--mat-sys-*`, and Google Material Icons across `AppComponent`, `LoginComponent`, `CallbackComponent`, `OnboardingComponent`, `ChatComponent`, `src/styles.scss`, and `src/index.html`.
  - Adopted `@spartan-ng/brain`, local Hlm primitives (`src/app/shared/ui/`), and Tailwind CSS 4 (`.postcssrc.json`).
- **Dual-Theme Token System & Brand Assets**:
  - Configured **Late Sun Engineering (Light)** and **Twilight Engineering (Dark, `html[data-theme="dark"]`)** CSS custom properties (`--color-*`, `--font-mono`, `--font-sans`, `--space-*`, `--radius-*`) in `src/styles.scss`.
  - Added `logo-light.svg`, `logo-dark.svg`, and `prefers-color-scheme` favicons with `"output": "assets"` in `angular.json`.
- **Route Surface Reduction & Screen Alignment**:
  - Restricted `src/app/app.routes.ts` strictly to **Auth** (`auth/login`, `auth/callback`), **Onboarding** (`onboarding`), and **Chat** (`chat`), deleting `spec`, `code`, `ops`, `sandbox`, `user/account`, `admin/users`, and `adminGuard`.
  - Added `OrgService.selectOrganisation(id: string)` for the `spartan-navbar` Organisation Switcher on `/chat`, and aligned `CallbackComponent` to `HlmCardDirective` / `HlmButtonDirective` redirecting to `/chat`.

### Merged Repositories & Release Commits
- [`minigma-ai-web-app`](file:///Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app): [PR #4](https://github.com/gpfister/minigma-ai-web-app/pull/4) squash-merged into `main` at `49360d2` — `feat(ui): align web app with PenPot spartan/ui + Tailwind CSS 4 design (#4)`.

---

## [2026-11-25] — `agentic-coding-plugin`: UI Design Stage & Change-Request Loop Between Specification and Planning

> [!NOTE] Release Summary: UI Design Stage & Change-Request Loop Merged & Released in `agentic-coding-plugin`
> - **Architecture Decision**: [[architecture/adr-014-ui-design-stage-and-change-request-loop|ADR-014: UI Design Stage & Change-Request Loop Between Specification and Planning]]
> - **Feature Note**: [[features/ui-design-stage-and-change-request-loop|Feature: UI Design Stage & Change-Request Loop (@coding <-> @coding-design)]]
> - **Architecture Note**: [[architecture/ui-design-stage-and-change-request-loop|UI Design Stage & Change-Request Loop Architecture & Implementation Plan]]
> - **Plugin Spec**: [[plugins/agentic-coding-plugin|agentic-coding-plugin]]
> - **Verification**: 27/27 automated unit tests passing in `agentic-coding-plugin` (100% pass rate, `uv run pytest -v` in `0.03s`).
> - **Branch & Worktree Lifecycle**: Merged via GitHub [PR #7](https://github.com/gpfister/agentic-coding-plugin/pull/7) (`feature/ui-design-stage-loop` -> `main` at [`634f62b`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/)); worktree `.worktrees/ui-design-stage-loop` removed and pruned; `feature/ui-design-stage-loop` deleted locally and remotely; `agentic-coding-plugin` synchronized on `main`.

### Overview & Key Capabilities Delivered
Enforced a deterministic **UI Design Stage (`@coding-design`)** strictly between **Phase 2 (`coding-specify`)** and **Phase 3 (`coding-plan`)** in `agentic-coding-plugin`:
- **Strict Stage Placement & Dual-Condition Trigger (`DesignCheck`)**:
  - Removed conflicting *"or alongside `@coding-plan` and `@coding-develop`"* wording in [`skills/coding-orchestrator/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/coding-orchestrator/SKILL.md).
  - Invokes `@coding-design` only when **both** `agentic-design-plugin` is installed **and** the task involves UI/UX changes; non-UI tasks advance directly to Phase 3 (`coding-plan`).
- **Pre-Clarified Fast-Path Alignment**:
  - Fast-Path checks whether `design-spec.md` is already approved before entering `coding-plan`, otherwise routing through `DesignCheck`.
- **Dual-Mode Validation Gate & Two-Branch Change-Request Loop**:
  - Added `@coding-design` to worker subagent relay lists in [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/rules/AGENTS.md) Sections 6 & 7 (`[QUESTION_RELAY_REQUEST]` and `[ARTIFACT_APPROVAL_REQUEST]`).
  - On `design-spec.md` rejection (`DesignGate`), evaluates feedback against `specification.md`: **Branch A** (functional, scope, requirement, user-flow, or mixed feedback) routes to `@coding-specify` (`P2`) before re-invoking `@coding-design`; **Branch B** (visual, layout, or styling only) re-invokes `@coding-design` directly.
- **Downstream Handoffs & Automated Regression Coverage**:
  - Passes approved `design-spec.md` and PenPot references to both `@coding-plan` and `@coding-develop`.
  - Added `test_coding_orchestrator_ui_design_stage_and_change_request_loop` in [`tests/test_plugin.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/tests/test_plugin.py) (`27/27` tests passing).

### Merged Repositories & Release Commits
- [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/): [PR #7](https://github.com/gpfister/agentic-coding-plugin/pull/7) merged into `main` at [`634f62b`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/) (**27/27 tests passed**)

---

## [2026-11-21] — `agentic-design-plugin`: `@coding-design` with Official PenPot MCP Integration

> [!NOTE] Release Summary: `agentic-design-plugin` Merged & Released across 5 Repositories
> - **Architecture Decision**: [[architecture/adr-013-coding-design-plugin-and-penpot-mcp|ADR-013: Coding Design Plugin (@coding-design) with Official PenPot MCP Integration]]
> - **Feature Note**: [[features/coding-design-plugin|Coding Design Plugin (@coding-design) with Official PenPot MCP Integration]]
> - **Plugin Spec**: [[plugins/agentic-design-plugin|agentic-design-plugin]]
> - **Vault Archival Index**: [[designs/index|Designs Index]]
> - **Verification**: 7/7 tests passing in `agentic-design-plugin`, 26/26 in `agentic-coding-plugin`, 87/87 in `agentic-prompt-triage-plugin`, and 11/11 in `agentic-project-documenting-plugin`; `make status` exited 0 in `agentic` root.
> - **Branch & Worktree Lifecycle**: `agentic-design-plugin` locally merged (`feature/coding-design-plugin` -> `main`); GitHub PRs merged into `main` across `agentic` (#4), `agentic-coding-plugin` (#6), `agentic-prompt-triage-plugin` (#1), and `agentic-project-documenting-plugin` (#2); all worktrees removed & pruned; feature branches deleted; primary repositories synchronized on `main`.

### Overview & Key Capabilities Delivered
Introduced **`agentic-design-plugin`** hosting the Dual-Mode **`@coding-design`** subagent with official **PenPot MCP** integration (`${PENPOT_MCP_URL:-http://localhost:4401/mcp}`):
- **Standalone Design Plugin (`agentic-design-plugin`)**:
  - Bidirectional PenPot authoring and inspection via MCP tools, synthesizing `<appDataDir>/brain/<conversation-id>/design-spec.md`.
  - Enforces Exclusive Code-Change Authority ([[architecture/adr-012-exclusive-code-change-authority|ADR-012]]): `edit_file` and `run_command` omitted; `write_file` restricted strictly to `design-spec.md`.
  - Complies with Dual-Mode Question & Artifact Approval Relay Protocols ([[architecture/adr-005-question-relay-protocol|ADR-005]], [[architecture/adr-006-artifact-approval-relay-protocol|ADR-006]]), with optional `@coding` implementation handoff in Standalone Mode.
- **Cross-Plugin Integration**:
  - **Root Workspace (`agentic`)**: Registered `agentic-design-plugin` in `Makefile`, `.gitignore`, and `AGENTS.md`.
  - **`agentic-coding-plugin` (`@coding`)**: Dynamic detection of `agentic-design-plugin` and optional Phase 2/3 `@coding-design` delegation for UI/UX tasks.
  - **`agentic-prompt-triage-plugin`**: Dynamic detection of `agentic-design-plugin` and `design` intent routing to `@coding-design`.
  - **`agentic-project-documenting-plugin` (`@coding-document`)**: Documented archival of approved `design-spec.md`, PenPot links, and design tokens into `agentic-obsidian/designs/<design-slug>.md` and `agentic-obsidian/designs/index.md`.

### Merged Repositories & Release Commits
- [`agentic-design-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-design-plugin/): Local merge `feature/coding-design-plugin` -> `main` at [`9b434b4`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-design-plugin/) (**7/7 tests passed**)
- [`agentic` (root)](file:///Users/gpfister/Development/Projects/gpfister/agentic/): [PR #4](https://github.com/gpfister/agentic/pull/4) merged into `main` at [`f234f6a`](file:///Users/gpfister/Development/Projects/gpfister/agentic/) (`make status` PASS)
- [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/): [PR #6](https://github.com/gpfister/agentic-coding-plugin/pull/6) merged into `main` at [`2348713`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/) (**26/26 tests passed**)
- [`agentic-prompt-triage-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/): [PR #1](https://github.com/gpfister/agentic-prompt-triage-plugin/pull/1) merged into `main` at [`bca48d6`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/) (**87/87 tests passed**)
- [`agentic-project-documenting-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/): [PR #2](https://github.com/gpfister/agentic-project-documenting-plugin/pull/2) merged into `main` at [`c2d3b19`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/) (**11/11 tests passed**)

---

## [2026-11-18] — Worktree-Based Development & Exclusive Code-Change Authority

> [!NOTE] Release Summary: Worktree-Based Development & Exclusive Code-Change Authority Merged & Released across 7 Repositories
> - **Architecture Decisions**:
>   - [[architecture/adr-011-worktree-based-development-lifecycle|ADR-011: Worktree-Based Development Lifecycle (.worktrees/<slug> Isolation)]]
>   - [[architecture/adr-012-exclusive-code-change-authority|ADR-012: Exclusive Code-Change Authority for @coding-develop]]
> - **Feature Note**: [[features/worktree-based-development-and-exclusive-code-change-authority|Worktree-Based Development & Exclusive Code-Change Authority]]
> - **Architecture Note**: [[architecture/worktree-based-development-and-exclusive-code-change-authority|Worktree-Based Development & Exclusive Code-Change Authority Architecture Plan]]
> - **Verification**: 26/26 tests passing in `agentic-coding-plugin`, 18/18 in `agentic-project-grooming-plugin`, and 12/12 in `agentic-task-monitoring-plugin`; global `grep -rn "edit_file"` audit confirms `edit_file` is declared exclusively on `@coding-develop`.
> - **Branch & Worktree Lifecycle**: Merged via 7 GitHub Pull Requests into `main`; all 7 isolated worktrees (`.worktrees/worktree-exclusive-code-authority`) removed & pruned; all `feature/worktree-exclusive-code-authority` branches deleted; primary repositories synchronized on `main`.

### Overview & Key Capabilities Delivered
Enforced two structural invariants across the `agentic` plugin ecosystem:
- **Worktree-Based Development Lifecycle (ADR-011)**:
  - Primary repository working trees remain strictly on `main` or `master` at all times during development, deployment, and review.
  - `@coding-develop` creates or reuses `<repo_root>/.worktrees/<slug>` via `git worktree add`, ensures `.worktrees/` is ignored, performs all code/test/config/repo-local doc edits exclusively inside `worktree_path`, and returns `worktree_path` + `branch` in `DevelopReport`.
  - `@coding` captures and propagates `worktree_path` to `@coding-deploy`, `@coding-review`, and `@coding-merge`.
  - `@coding-merge` merges into `main` in the primary repository and cleans up via `git worktree remove` + `git worktree prune`.
- **Exclusive Code-Change Authority (ADR-012)**:
  - `@coding-develop` is the sole agent authorized to declare `edit_file` and modify repository source code, configuration files, test suites, or repo-local documentation.
  - Removed `edit_file` from `@coding-deploy` and `@coding-document` (and their agent registry tables).
  - Restricted `write_file` across all other agents strictly to session artifacts (`<appDataDir>/brain/<conversation-id>/*.md`, or `*-obsidian/` for `@coding-document`) and restricted `run_command` to non-mutating / read-only / operational commands.
  - Updated `/project-grooming` fallback (when `agentic-coding-plugin` is absent) to output manual scaffolding guidance instead of writing repository files directly.

### Merged Repositories & Pull Requests
- [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/): [PR #5](https://github.com/gpfister/agentic-coding-plugin/pull/5) (`MERGED`, **26/26 tests passed**)
- [`agentic-project-grooming-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/): [PR #1](https://github.com/gpfister/agentic-project-grooming-plugin/pull/1) (`MERGED`, **18/18 tests passed**)
- [`agentic-project-documenting-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/): [PR #1](https://github.com/gpfister/agentic-project-documenting-plugin/pull/1) (`MERGED`)
- [`agentic-issue-investigation-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/): [PR #2](https://github.com/gpfister/agentic-issue-investigation-plugin/pull/2) (`MERGED`)
- [`agentic-task-monitoring-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-task-monitoring-plugin/): [PR #1](https://github.com/gpfister/agentic-task-monitoring-plugin/pull/1) (`MERGED`, **12/12 tests passed**)
- [`agentic-topic-researching-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-topic-researching-plugin/): [PR #1](https://github.com/gpfister/agentic-topic-researching-plugin/pull/1) (`MERGED`)
- [`agentic` (root)](file:///Users/gpfister/Development/Projects/gpfister/agentic/): [PR #3](https://github.com/gpfister/agentic/pull/3) (`MERGED`)

---

## [2026-11-17] — Plan-vs-Changes Audit, `review-<slug>.md` Artifact, and Unplanned Changes Validation Gate for @coding-review (SPEC-2026-REVIEW-001)

> [!NOTE] Release Summary: SPEC-2026-REVIEW-001 Merged & Released in agentic-coding-plugin
> - **Specification**: `SPEC-2026-REVIEW-001`
> - **Architecture Decision**: [[architecture/adr-010-coding-review-plan-audit-and-unplanned-changes-gate|ADR-010: Plan-vs-Changes Audit, Non-Blocking Review Artifact, and Unplanned Changes Validation Gate for @coding-review]]
> - **Feature Note**: [[features/coding-review-plan-audit-and-unplanned-changes-gate|Plan-vs-Changes Audit, review-<slug>.md Artifact, and Unplanned Changes Validation Gate for @coding-review]]
> - **Architecture Note**: [[architecture/coding-review-plan-audit-and-unplanned-changes-gate|Plan-vs-Changes Audit, review-<slug>.md Artifact, and Unplanned Changes Validation Gate Architecture Plan]]
> - **Verification**: 25/25 automated unit tests passing in `agentic-coding-plugin` (100% pass rate).
> - **Branch Lifecycle**: Merged via GitHub [PR #4](https://github.com/gpfister/agentic-coding-plugin/pull/4) into `main` at [`151f203`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/); feature branch `feature/review-plan-audit` deleted locally and remotely; remote tracking pruned; working tree clean on `main`.

### Overview & Key Capabilities Delivered
Upgraded Stage 6 ([`@coding-review`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-review/agent.md)) and [`@coding`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding/agent.md) in `agentic-coding-plugin` to enforce plan-vs-changes verification, persistent review artifacts, and user validation of unplanned changes prior to merge confirmation:
- **Plan-vs-Changes Git Audit**:
  - `@coding` passes the approved `plan.md` path alongside `specification.md`, `DevelopReport`s, and `DeployReport`s to `@coding-review`.
  - `@coding-review` inspects `git status` and `git diff` across all participating repositories against `plan.md` and classifies any out-of-plan modifications as *unplanned changes*.
- **Non-Blocking `review-<slug>.md` Artifact**:
  - Generated on every review run in `<appDataDir>/brain/<conversation-id>/review-<slug>.md` without `RequestFeedback=True` or `[ARTIFACT_APPROVAL_REQUEST]`, containing all 5 mandatory audit sections (reporting `"None"` when no unplanned changes exist).
  - `write_file` added to `coding-review`'s allowed tools strictly scoped to `<appDataDir>/brain/<conversation-id>/review-<slug>.md`.
- **Conditional Unplanned Changes Validation Gate & `[REVIEW_ARTIFACT]` Ordering**:
  - When unplanned changes are detected, `@coding-review` sends `[REVIEW_ARTIFACT]` to `@coding` (forwarded to `caller_id` in Subagent Mode, or presented/linked before `ask_question` in Standalone Mode) **before** dispatching `[QUESTION_RELAY_REQUEST]` (`"Do you approve the changes ?"` with `(Recommended) Yes`, `No, revert all these changes`, and native custom write-in support).
  - Reverts and custom revisions are returned in `ReviewReport` for `@coding` to delegate to `@coding-develop` before re-running `@coding-review`.
  - Merge Confirmation remains a separate final gate executed only after unplanned changes (if any) are approved and all quality checks pass.

### Merged Repositories & Release Commits
- [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/): [PR #4](https://github.com/gpfister/agentic-coding-plugin/pull/4) merged into `main` at [`151f203`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/). 25/25 tests passed; `feature/review-plan-audit` deleted locally and remotely; `main` synchronized.

---

## [2026-11-08] — Complete Branch Cleanup, Base Branch Checkout, and Remote Tracking Prune for @coding-merge (SPEC-2026-MERGE-002)

> [!NOTE] Release Summary: SPEC-2026-MERGE-002 Merged & Released in agentic-coding-plugin
> - **Specification**: `SPEC-2026-MERGE-002`
> - **Architecture Decision**: [[architecture/adr-009-coding-merge-complete-branch-cleanup-and-remote-sync|ADR-009: Complete Branch Cleanup, Base Branch Checkout, and Remote Tracking Prune for @coding-merge]]
> - **Feature Note**: [[features/coding-merge-complete-branch-cleanup-and-remote-sync|Complete Branch Cleanup, Base Branch Checkout, and Remote Tracking Prune for @coding-merge]]
> - **Architecture Note**: [[architecture/coding-merge-complete-branch-cleanup-and-remote-sync|Complete Branch Cleanup, Base Branch Checkout, and Remote Tracking Synchronization Architecture Plan]]
> - **Verification**: 22/22 automated unit tests passing in `agentic-coding-plugin` (100% pass rate). Pre-merge clean code review approved (0 blockers).
> - **Branch Lifecycle**: Merged via GitHub Pull Request #3 (Squash & Merge) into `main`; upstream fast-forward synchronized; feature branch `feature/merge-branch-cleanup-and-remote-sync` deleted locally and remotely; remote tracking references pruned (`git fetch --prune origin`); repository working tree clean on `main`.

### Overview & Key Capabilities Delivered
Upgraded Stage 7 release engineering subagent ([`@coding-merge`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-merge/agent.md)) in `agentic-coding-plugin` to enforce complete post-merge branch hygiene, working tree assurance, and remote tracking reference synchronization:
- **Local Release Strategy Branch Cleanup & Base Switch**:
  - Mandated that `@coding-merge` verifies working tree checkout on `<base>` (`git checkout <base>`, `git branch --show-current`).
  - Merges the feature branch into `<base>` and immediately deletes the local feature branch safely (`git branch -d <branch>`).
  - Emits structured `MergeReport` with `cleanup: true` and `deleted_branches: { "local": "<branch>" }`, eliminating stale local branch accumulation in repositories without remotes.
- **Remote Release Strategy Complete Cleanup & Pruning**:
  - Following confirmed Pull Request merge (`gh pr view` reports `state == "MERGED"`), checks out `<base>` and pulls remote updates (`git pull --ff-only origin <base>`).
  - Deletes the remote feature branch on `origin` (`git push origin --delete <branch>`), handling pre-deleted branches idempotently.
  - Deletes the local feature branch (`git branch -d <branch>`, with fallback to `-D` only when PR merge is verified and base branch is synchronized).
  - Executes `git fetch --prune origin` to purge obsolete remote-tracking branches from `.git/refs/remotes/origin/`, eliminating lingering ghost branches in `git branch -a`.
  - Verifies active branch is `<base>` and emits structured `MergeReport` with `cleanup: true`, `deleted_branches: { "local": "<branch>", "remote": "<branch>" }`, and `pruned: true`.
- **Documentation & Agent Specification Alignment**:
  - Updated [`agents/coding-merge/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-merge/agent.md) Sections 2 & 5 with complete cleanup steps and updated `MergeReport` schemas.
  - Updated Stage 7 descriptions and Mermaid sequence diagrams in [`agents/coding/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding/agent.md) and [`agents/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/agent.md).
  - Updated Phase 7 instructions in [`skills/coding-orchestrator/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/coding-orchestrator/SKILL.md).
  - Added "Release & Merge Hygiene" section to [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/README.md).
- **Automated Test Verification**:
  - Expanded `tests/test_plugin.py` to assert `git push origin --delete`, `git fetch --prune`, local merge cleanup, and base branch checkout.
  - Achieved 100% test pass rate (22/22 unit tests passing).

### Merged Repositories & Release Commits
- [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/): [PR #3](https://github.com/gpfister/agentic-coding-plugin/pull/3) merged into `main` via squash commit [`b201d28`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/) — `feat(coding-merge): implement complete branch cleanup and remote tracking synchronization (#3)`. 22/22 tests passed; feature branch deleted locally and remotely; `main` fast-forward synced; remote tracking pruned.
- [`agentic` (root)](file:///Users/gpfister/Development/Projects/gpfister/agentic/): Commit [`ef13d6c`](file:///Users/gpfister/Development/Projects/gpfister/agentic/) on `main` — `docs: pre-merge documentation tidy-up for coding-merge complete branch cleanup and remote sync`.
- [`agentic-obsidian`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/): Commits [`9d0de36`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/) and [`f6147ad`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/) on `main` — `docs(plan): document architecture plan, feature spec, and formalize ADR-009` and `docs(vault): pre-merge documentation tidy-up`.

---

## [2026-11-06] — Dual-Mode Coding Orchestrator with Multi-Hop Relay and Anti-Direct-Coding Guardrails (SPEC-2026-DUAL-MODE-001)

> [!NOTE] Release Summary: SPEC-2026-DUAL-MODE-001 Merged & Released across 3 Repositories
> - **Specification**: `SPEC-2026-DUAL-MODE-001`
> - **Architecture Decision**: [[architecture/adr-008-dual-mode-coding-orchestrator|ADR-008: Dual-Mode Coding Orchestrator with Multi-Hop Relay and Anti-Direct-Coding Guardrails]]
> - **Feature Note**: [[features/dual-mode-coding-orchestrator|Dual-Mode Coding Orchestrator with Multi-Hop Relay and Anti-Direct-Coding Guardrails]]
> - **Architecture Note**: [[architecture/dual-mode-coding-orchestrator|Dual-Mode Coding Orchestrator Architecture & Implementation Plan]]
> - **Verification**: 21/21 automated unit tests passing in `agentic-coding-plugin` (100%), 14/14 automated unit tests passing in `agentic-issue-investigation-plugin` (100%). Pre-merge clean code review approved (0 blockers).
> - **Branch Lifecycle**: Merged via GitHub Pull Requests (Squash & Merge) into `master`; upstream fast-forward synchronized; feature branches `feat/dual-mode-coding-orchestrator` safely deleted.

### Overview & Key Capabilities Delivered
Upgraded `@coding` (the central software development lifecycle orchestrator) in `agentic-coding-plugin` to eliminate host 60-second execution timeouts, prevent duplicate approval gate UI modals, enforce architectural integrity via anti-direct-coding guardrails, and provide seamless automated handoffs:
- **Dual-Mode Orchestration (`caller_id` Detection)**:
  - Dynamically distinguishes between Subagent Mode (`caller_id` present) and Standalone Mode (no `caller_id`).
  - In **Subagent Mode**: `@coding` operates as a transparent relay node, forwarding worker requests upward to `caller_id` via `send_message` and never calling `write_file(RequestFeedback=True)` or `ask_question` directly.
  - In **Standalone Mode**: `@coding` directly renders the single interactive approval modal at the root turn via `write_file(RequestFeedback=True)` and interactively prompts the user via `ask_question`.
- **Multi-Hop Relay Protocol**:
  - Implemented multi-hop forwarding for both interactive approval gates (`[ARTIFACT_APPROVAL_REQUEST]`) and user clarification inquiries (`[QUESTION_RELAY_REQUEST]`).
  - Halts execution into an idle state while awaiting parent caller responses (`[ARTIFACT_APPROVAL_RESPONSE]`, `[QUESTION_RELAY_RESPONSE]`), waking reactively to relay decisions downward to worker subagents.
- **Hard Anti-Direct-Coding Guardrails**:
  - Established hard negative constraints preventing `@coding` from directly editing application code files or running build/test tools.
  - Restricts `write_file` strictly to interactive approval gate rendering in standalone mode; restricts `run_command` strictly to non-destructive workspace inspection (`git status`).
  - Enforces 100% lifecycle delegation to worker subagents (`@coding-develop`, `@coding-deploy`, `@coding-review`, `@coding-merge`).
- **Pre-Clarified Fast-Path**:
  - Automatically detects pre-clarified requirements (such as handoffs from `@coding-investigate` or `/project-grooming`) and skips Phase 1 (`coding-clarify`), advancing straight to Phase 2 (`coding-specify`) and Phase 3 (`coding-plan`) while strictly preserving their human approval gates.
- **Standardized Upstream Invocation Template**:
  - Standardized caller prompt template (`Orchestrate the 7-stage software development lifecycle to implement the following feature / correction based on the approved assessment: <details/path>. Do not implement directly; delegate all phases through worker subagents.`) across workspace documentation and companion subagents.

### Merged Repositories & Release Commits
- [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/): [PR #2](https://github.com/gpfister/agentic-coding-plugin/pull/2) merged into `master` via squash commit [`3e04ac1`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/) — `feat(coding): implement dual-mode orchestrator with multi-hop relay protocols and hard anti-direct-coding boundary (#2)`. 21/21 tests passed; feature branch deleted; `master` fast-forward synced.
- [`agentic` (root)](file:///Users/gpfister/Development/Projects/gpfister/agentic/): [PR #2](https://github.com/gpfister/gp-agentic/pull/2) merged into `master` via squash commit [`fa5164a`](file:///Users/gpfister/Development/Projects/gpfister/agentic/) — `feat(orchestrator): document dual-mode coding orchestrator, artifact approval relay, and standardized upstream invocation (#2)`. Feature branch deleted; `master` fast-forward synced.
- [`agentic-issue-investigation-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/): [PR #1](https://github.com/gpfister/agentic-issue-investigation-plugin/pull/1) merged into `master` via squash commit [`34629b3`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/) — `feat(investigate): standardize upstream @coding invocation prompt and subagent relay delegation (#1)`. 14/14 tests passed; feature branch deleted; `master` fast-forward synced.

---

## [2026-11-06] — Remote PR Creation, Question Relay Waiting Gate, and Post-Merge Synchronization for @coding-merge (SPEC-2026-MERGE-001)

> [!NOTE] Release Summary: SPEC-2026-MERGE-001 Merged & Released across 2 Repositories
> - **Specification**: `SPEC-2026-MERGE-001`
> - **Architecture Decision**: [[architecture/adr-007-coding-merge-remote-pr-and-sync|ADR-007: Remote PR Creation, Question Relay Waiting Gate, and Post-Merge Synchronization for @coding-merge]]
> - **Feature Note**: [[features/coding-merge-remote-pr-and-sync|Remote PR Creation, Question Relay Waiting Gate, and Post-Merge Synchronization for @coding-merge]]
> - **Architecture Note**: [[architecture/coding-merge-remote-pr-and-sync|Remote PR Creation, Question Relay Waiting Gate, and Post-Merge Synchronization Architecture & Implementation Plan]]
> - **Verification**: 17/17 automated unit tests passing in `agentic-coding-plugin` (100% pass rate). Pre-merge clean code review approved (0 blockers).
> - **Branch Lifecycle**: Merged via GitHub Pull Requests (Squash & Merge) into `master`; upstream fast-forward synchronized; feature branches `feature/coding-merge-pr-waiting-gate` safely deleted.

### Overview & Key Capabilities Delivered
Upgraded Stage 7 release engineering subagent (`@coding-merge`) in `agentic-coding-plugin` to support remote Pull Request workflows, eliminating 60-second host timeout failures during human review and automating post-merge cleanup:
- **Remote Repository Detection & Validation**:
  - Automatically inspects remotes (`git remote -v`); selects local merge strategy for local-only repositories and remote PR strategy for repositories with configured origins.
- **Resilient Remote PR Creation & Fallback**:
  - Validates GitHub CLI authentication (`gh auth status`) and creates PR via `gh pr create`.
  - When `gh` is unauthenticated or missing, constructs a web browser compare URL (`https://github.com/<owner>/<repo>/compare/...`) and guides the user via the Question Relay Protocol.
- **Subagent-Safe PR Waiting Gate**:
  - Polls PR status non-blockingly via `gh pr view --json state,reviewDecision`. When `state == "OPEN"`, halts subagent execution and delegates review options (`Re-check`, `Enable auto-merge`, `Proceed pending`) to the parent orchestrator via `[QUESTION_RELAY_REQUEST]`, completely eliminating Antigravity's 60-second subagent timeouts.
- **Post-Merge Base Synchronization & Local Branch Cleanup**:
  - Upon remote merge confirmation (`state == "MERGED"`), checks out base branch (`git checkout <base>`), fast-forward pulls upstream changes (`git pull --ff-only origin <base>`), and safely prunes the local feature branch (`git branch -d <branch>`).
- **Tooling & Pure Subagent Matrix Alignment**:
  - Added `send_message` to `coding-merge/agent.md` frontmatter tools.
  - Classified `@coding-merge` as a Pure Subagent alongside `@coding-clarify`, `@coding-deploy`, and `@coding-review` across rules and workspace `AGENTS.md`.

### Merged Repositories & Release Commits
- [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/): [PR #1](https://github.com/gpfister/agentic-coding-plugin/pull/1) merged into `master` via squash commit [`2061642`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/) — `feat(coding-merge): PR creation, waiting gate, and branch sync (#1)`. Feature branch deleted and `master` fast-forward synced.
- [`agentic` (root)](file:///Users/gpfister/Development/Projects/gpfister/agentic/): [PR #1](https://github.com/gpfister/agentic/pull/1) merged into `master` via squash commit [`3f09b7c`](file:///Users/gpfister/Development/Projects/gpfister/agentic/) — `docs(agents): document coding-merge PR waiting gate and sync in AGENTS.md (#1)`. Feature branch deleted and `master` fast-forward synced.

---

## [2026-11-05] — Artifact Approval Relay Protocol for Subagent Gates (SPEC-2026-RELAY-002)

> [!NOTE] Release Summary: SPEC-2026-RELAY-002 Merged across 5 Repositories
> - **Specification**: `SPEC-2026-RELAY-002`
> - **Architecture Decision**: [[architecture/adr-006-artifact-approval-relay-protocol|ADR-006: Artifact Approval Relay Protocol for Subagent Approval Gates]]
> - **Feature Note**: [[features/artifact-approval-relay-protocol|Artifact Approval Relay Protocol for Subagent Gates]]
> - **Architecture Note**: [[architecture/artifact-approval-relay-protocol|Artifact Approval Relay Protocol Architecture & Implementation Plan]]
> - **Verification**: 158/158 automated unit tests passing workspace-wide (60/60 across feature repositories, 100% pass rate). Pre-merge clean code reviews approved.
> - **Branch Topology**: Fast-forward merged into `master` across all 5 participating repositories; `feature/artifact-approval-relay` branches deleted.

### Overview & Key Capabilities Delivered
Standardized an event-driven Artifact Approval Relay Protocol (`[ARTIFACT_APPROVAL_REQUEST]` / `[ARTIFACT_APPROVAL_RESPONSE]`) across all companion plugins to completely eliminate host 60-second execution timeout failures when background subagents author milestone deliverables requiring human approval:
- **Pure Subagents vs. Dual-Mode Subagents**:
  - Pure subagents (`@coding-specify`, `@coding-plan`, `@coding-deploy`) write deliverables (`specification.md`, `plan.md`, `deployment-issue.md`) directly to disk without `RequestFeedback=True` and relay interactive approval gates to the parent orchestrator via `send_message`.
  - Dual-Mode subagents (`@coding-investigate`, `/project-grooming` `@coding-groom`, `@coding-research`) inspect caller context dynamically: they relay milestone approval gates (`issue-assessment.md`, `Project_Request_Grooming.md`, `research-report.md`) via `[ARTIFACT_APPROVAL_REQUEST]` when operating under an orchestrator, or author directly with `RequestFeedback=True` as the sole and final tool call when invoked in standalone root sessions.
- **Top-Level Orchestration Relay Handling**:
  - The root orchestrator (`@coding`) intercepts `[ARTIFACT_APPROVAL_REQUEST]`, verifies the deliverable on disk, renders the interactive feedback gate (`write_file(RequestFeedback=True)`) at the session root without background execution timeouts, and returns `[ARTIFACT_APPROVAL_RESPONSE]` to reactively awaken the subagent.
- **Workspace Coordination Guidelines**:
  - Formalized Section 3.3 (Artifact Approval Relay Protocol) in root [`AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/AGENTS.md), plugin rules, and skills runbooks.

### Merged Repositories & Commits
- [`agentic` (root)](file:///Users/gpfister/Development/Projects/gpfister/agentic/): Commit `419cd50` on `master` — `docs: add Artifact Approval Relay Protocol to AGENTS.md` (branch `feature/artifact-approval-relay` deleted)
- [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/): Commit `8302109` on `master` — `feat(approval-relay): implement Artifact Approval Relay Protocol across subagents and orchestrator` (16/16 tests passed; branch `feature/artifact-approval-relay` deleted)
- [`agentic-issue-investigation-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/): Commit `0939f82` on `master` — `feat(relay): implement dual-mode Artifact Approval Relay Protocol in issue investigator` (13/13 tests passed; branch `feature/artifact-approval-relay` deleted)
- [`agentic-project-grooming-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/): Commit `2729c34` on `master` — `feat(relay): implement dual-mode Artifact Approval Relay Protocol for coding-groom` (16/16 tests passed; branch `feature/artifact-approval-relay` deleted)
- [`agentic-topic-researching-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-topic-researching-plugin/): Commit `5c0931f` on `master` — `feat(relay): implement dual-mode Artifact Approval Relay Protocol for coding-research` (15/15 tests passed; branch `feature/artifact-approval-relay` deleted)

---

## [2026-11-05] — Question Relay Protocol for Subagent User Inquiries (SPEC-2026-RELAY-001)

> [!NOTE] Release Summary: SPEC-2026-RELAY-001 Merged across 6 Repositories
> - **Specification**: `SPEC-2026-RELAY-001`
> - **Architecture Decision**: [[architecture/adr-005-question-relay-protocol|ADR-005: Question Relay Protocol for Subagent User Inquiries]]
> - **Feature Note**: [[features/question-relay-protocol|Question Relay Protocol for Subagent User Prompts]]
> - **Architecture Note**: [[architecture/question-relay-protocol|Question Relay Protocol Architecture & Implementation Plan]]
> - **Verification**: 147/147 automated unit tests passing workspace-wide (60/60 across feature repositories, 100% pass rate).
> - **Branch Topology**: Fast-forward merged into `master` across all 6 participating repositories; `feature/question-relay-protocol` branches deleted.

### Overview & Key Capabilities Delivered
Standardized an event-driven Question Relay Protocol (`[QUESTION_RELAY_REQUEST]` / `[QUESTION_RELAY_RESPONSE]`) across the multi-agent workspace to completely eliminate premature 60-second host execution timeout terminations on subagents invoking user-prompting tools:
- **Pure Subagents vs. Dual-Mode Subagents**:
  - Pure subagents (`@coding-clarify`, `@coding-deploy`, `@coding-review`) omit `ask_question` from tools frontmatter and relay all inquiries through their parent caller via `send_message`.
  - Dual-mode subagents (`@coding-investigate`, `/project-grooming` `@coding-groom`, `@coding-monitor`, `@coding-research`) check caller ID dynamically: they dispatch relay envelopes when spawned under an orchestrator, or prompt directly via `ask_question` in standalone root sessions.
- **Top-Level Orchestration Relay Handling**:
  - Orchestrator (`@coding`) intercepts `[QUESTION_RELAY_REQUEST]`, prompts the user interactively at the root session turn without host timeout limits, and dispatches `[QUESTION_RELAY_RESPONSE]` to reactively awaken worker subagents.
- **Workspace Coordination Guidelines**:
  - Formalized Question Relay Protocol in workspace-wide rules (`AGENTS.md`), plugin rules, and skills runbooks.

### Merged Repositories & Commits
- [`agentic` (root)](file:///Users/gpfister/Development/Projects/gpfister/agentic/): Commit `695e1b1` — `feat(relay): formalize Question Relay Protocol across workspace coordination rules and documentation`
- [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/): Commit `d2c822c` — `feat(relay): implement Question Relay Protocol across coding subagents` (15/15 tests passed)
- [`agentic-issue-investigation-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/): Commit `df03d3c` — `feat(relay): implement dual-mode Question Relay Protocol for coding-investigate` (10/10 tests passed)
- [`agentic-project-grooming-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/): Commit `a2e7932` — `feat(relay): implement dual-mode Question Relay Protocol for project grooming` (12/12 tests passed)
- [`agentic-task-monitoring-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-task-monitoring-plugin/): Commit `23a101e` — `feat(relay): implement dual-mode Question Relay Protocol for coding-monitor` (11/11 tests passed)
- [`agentic-topic-researching-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-topic-researching-plugin/): Commit `efb067d` — `feat(relay): implement dual-mode Question Relay Protocol for coding-research` (12/12 tests passed)

---

## [2026-11-02] — Multi-Repo Plugin Standardization & AC-8 Remediation

> [!NOTE] Release Summary: SPEC-2026-PLUGIN-RENAME-001 Merged across 5 Plugins
> - **Specification**: `SPEC-2026-PLUGIN-RENAME-001`
> - **Architecture Decision**: [[architecture/adr-004-standardize-plugin-repository-names|ADR-004: Standardize Plugin Repository Names]]
> - **Feature Note**: [[features/rename-plugin-repositories|Rename Plugin Repositories to Descriptive Action-Oriented Names]]
> - **Verification**: 135/135 automated unit tests passing across all plugins (100%).

### Merged Repositories & Commits
- [`agentic-prompt-triage-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/): Commit `9c5c19b` (77/77 tests passed)
- [`agentic-project-documenting-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/): Commit `00b7d03` (10/10 tests passed)
- [`agentic-issue-investigation-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/): Commit `8f46739` (7/7 tests passed)
- [`agentic-task-monitoring-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-task-monitoring-plugin/): Commit `76f7002` (8/8 tests passed)
- [`agentic-topic-researching-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-topic-researching-plugin/): Commit `ee5bfa0` (9/9 tests passed)

---

## [2026-11-01] — `agentic-project-grooming-plugin`: Interactive Scoping

> [!NOTE] Release Summary: Project Grooming Plugin & Skill (/project-grooming)
> - **Repository**: [`agentic-project-grooming-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/)
> - **Architecture Decision**: [[architecture/adr-003-project-grooming-plugin|ADR-003]]
> - **Feature Note**: [[features/project-grooming-plugin|Project Grooming Plugin]]
> - **Verification**: 8/8 automated unit tests passing (100%).
