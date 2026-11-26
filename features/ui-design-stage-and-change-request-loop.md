---
title: "UI Design Stage & Change-Request Loop Between Specification and Planning"
date: "2026-11-25"
status: "Completed / Merged"
phase: "Phase 7: Merge & Release Complete"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/7c2e6aa5-d2ae-44bb-94f4-a5354ea85b6a/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/7c2e6aa5-d2ae-44bb-94f4-a5354ea85b6a/plan.md"
review_path: "/Users/gpfister/.gemini/antigravity/brain/7c2e6aa5-d2ae-44bb-94f4-a5354ea85b6a/review-ui-design-stage-loop.md"
adrs:
  - "[[architecture/adr-014-ui-design-stage-and-change-request-loop|ADR-014: UI Design Stage & Change-Request Loop Between Specification and Planning]]"
tags:
  - feature
  - agentic-coding-plugin
  - agentic-design-plugin
  - coding-design
  - dual-mode-relay
  - change-request-loop
  - merged
  - completed
affected_plugins:
  - agentic-coding-plugin
branch: "main"
merged_branch: "feature/ui-design-stage-loop"
pr_url: "https://github.com/gpfister/agentic-coding-plugin/pull/7"
commit: "634f62b"
aliases:
  - ui-design-stage-and-change-request-loop
---

# Feature: UI Design Stage & Change-Request Loop (`@coding` <-> `@coding-design`)

> [!NOTE] Lifecycle Status: Phase 7 Complete — Merged & Released ([PR #7](https://github.com/gpfister/agentic-coding-plugin/pull/7))
> - **Approved Specification**: [specification.md](file:///Users/gpfister/.gemini/antigravity/brain/7c2e6aa5-d2ae-44bb-94f4-a5354ea85b6a/specification.md)
> - **Approved Plan**: [plan.md](file:///Users/gpfister/.gemini/antigravity/brain/7c2e6aa5-d2ae-44bb-94f4-a5354ea85b6a/plan.md)
> - **Review & QA Audit**: [review-ui-design-stage-loop.md](file:///Users/gpfister/.gemini/antigravity/brain/7c2e6aa5-d2ae-44bb-94f4-a5354ea85b6a/review-ui-design-stage-loop.md) (**PASS**, `27/27` tests passing)
> - **Architecture Plan**: [[architecture/ui-design-stage-and-change-request-loop|Architecture & Implementation Plan]]
> - **Decision Record**: [[architecture/adr-014-ui-design-stage-and-change-request-loop|ADR-014]]
> - **Merge & Cleanup**: [PR #7](https://github.com/gpfister/agentic-coding-plugin/pull/7) merged `feature/ui-design-stage-loop` -> `main` at `634f62b` (implementation commit `314ce39`); worktree `.worktrees/ui-design-stage-loop` removed and local/remote branches deleted & pruned.

---

## 1. Summary

Establishes a deterministic **UI Design Stage (`@coding-design`)** strictly between **Phase 2 (`coding-specify`)** and **Phase 3 (`coding-plan`)** in `agentic-coding-plugin`. Resolves conflicting orchestration guidance, aligns Pre-Clarified Fast-Path execution, formalizes the Dual-Mode `design-spec.md` validation gate, and defines a two-branch **Design Change-Request Loop** on rejection.

---

## 2. Functional Requirements

- **FR-1 (Strict Stage Placement)**: `@coding-design` runs strictly between `coding-specify` and `coding-plan`; all *"alongside `@coding-plan` and `@coding-develop`"* wording is removed.
- **FR-2 (Dual-Condition `DesignCheck`)**: Invokes `@coding-design` only when **both** `agentic-design-plugin` is installed **and** the task involves UI/UX changes; non-UI tasks advance directly to `coding-plan`.
- **FR-3 (Dual-Mode Validation Gate)**: Relays `[QUESTION_RELAY_REQUEST]` and `[ARTIFACT_APPROVAL_REQUEST]` for `design-spec.md` via Dual-Mode Orchestrator rules ([[architecture/adr-008-dual-mode-coding-orchestrator|ADR-008]]).
- **FR-4 (Design Change-Request Loop)**:
  - **Branch A (Functional, Scope, Requirement, User-Flow, or Mixed Feedback)**: Routes to `@coding-specify` (`P2`) -> `specification.md` approval gate -> re-invokes `@coding-design` (or `SpecRejected` -> `coding-clarify` if `specification.md` is rejected).
  - **Branch B (Visual, Layout, or Styling Only)**: Re-invokes `@coding-design` directly with rejection feedback.
- **FR-5 (Pre-Clarified Fast-Path Consistency)**: Checks whether `design-spec.md` is already approved before entering `coding-plan`, otherwise routes through `DesignCheck`.
- **FR-6 (Downstream Handoff)**: Passes approved `design-spec.md` path and PenPot references to both `@coding-plan` and `@coding-develop`.
- **FR-7 (Cross-File Consistency)**: Updates `agentic-coding-plugin/skills/coding-orchestrator/SKILL.md` (Sections 3, 4, 5) and `agentic-coding-plugin/rules/AGENTS.md` (Sections 6 & 7).

---

## 3. Edge Cases & Mitigations

| Edge Case | Mitigation |
| :--- | :--- |
| Mixed rejection feedback (visual tweaks + new functional requirement) | Treated as Branch A: routed to `@coding-specify` first so `specification.md` remains the single source of truth. |
| `design-spec.md` already approved in Fast-Path handoff | Fast-Path detects approved `design-spec.md` and advances directly to `coding-plan`. |
| `specification.md` rejected during Change-Request Loop | Follows existing `SpecRejected` -> `coding-clarify` routing to prevent infinite loops. |
| `agentic-design-plugin` installed, task is backend/CLI | `DesignCheck` requires both plugin installation and UI/UX scope; skips directly to `coding-plan`. |

---

## 4. Implementation & Verification Summary

- **Merged Changes** (`agentic-coding-plugin`, [PR #7](https://github.com/gpfister/agentic-coding-plugin/pull/7) merged into `main` at `634f62b`):
  - [`skills/coding-orchestrator/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/coding-orchestrator/SKILL.md) (`+30 / -7` lines across Sections 3, 4, and 5).
  - [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/rules/AGENTS.md) (`+4 / -4` lines adding `@coding-design` to Sections 6 & 7).
  - [`tests/test_plugin.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/tests/test_plugin.py) (`+37` lines adding `test_coding_orchestrator_ui_design_stage_and_change_request_loop`).
- **Verification**: `uv run pytest -v` passed **27/27** tests in `0.03s`; zero unplanned changes.
