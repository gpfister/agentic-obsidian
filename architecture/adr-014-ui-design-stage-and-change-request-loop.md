---
adr_id: "014"
title: "UI Design Stage & Change-Request Loop Between Specification and Planning"
status: "accepted"
date: "2026-11-25"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/7c2e6aa5-d2ae-44bb-94f4-a5354ea85b6a/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/7c2e6aa5-d2ae-44bb-94f4-a5354ea85b6a/plan.md"
tags:
  - architecture-decision
  - agentic-coding-plugin
  - agentic-design-plugin
  - coding-design
  - workflow-orchestration
  - dual-mode-relay
  - change-request-loop
  - plan-approved
aliases:
  - ADR-014
  - adr-014-ui-design-stage-and-change-request-loop
---

# ADR-014: UI Design Stage & Change-Request Loop Between Specification and Planning

## Context

Following the introduction of `agentic-design-plugin` (`@coding-design`) in [[architecture/adr-013-coding-design-plugin-and-penpot-mcp|ADR-013]], the `@coding` orchestrator workflow in `agentic-coding-plugin/skills/coding-orchestrator/SKILL.md` contained conflicting and incomplete guidance:

1. **Stage Placement Ambiguity**: Section 4's Mermaid flowchart placed `DesignCheck` after `SpecApproved` and before `Phase 3: Planning`, whereas Section 5 (`Phase Handoffs & Execution Rules`, item 2) stated that `@coding-design` may be invoked *"before or alongside `@coding-plan` and `@coding-develop`"*.
2. **Missing Approval Gate & Change-Request Loop**: The flowchart did not model the Dual-Mode `[ARTIFACT_APPROVAL_REQUEST]` gate for `design-spec.md` or how rejections at that gate should route back to `@coding-specify` vs. `@coding-design`.
3. **Pre-Clarified Fast-Path Bypass**: Section 3 (`Pre-Clarified Fast-Path Execution`) advances directly from an already-approved `specification.md` to `Phase 3 (coding-plan)`, bypassing `DesignCheck` even for UI/UX tasks.
4. **Protocol List Consistency**: `agentic-coding-plugin/rules/AGENTS.md` (Sections 6 & 7) did not explicitly list `@coding-design` alongside the worker subagents whose `[QUESTION_RELAY_REQUEST]` and `[ARTIFACT_APPROVAL_REQUEST]` envelopes are intercepted and relayed by `@coding`.

### Alternatives Considered

1. **Keep `@coding-design` Optional Alongside `@coding-plan` / `@coding-develop`**:
   - *Rejection Rationale*: Produces non-deterministic handoffs; `@coding-plan` cannot decompose UI component tasks without an approved `design-spec.md` and PenPot references, and parallel execution risks conflicting assumptions.
2. **Route All `design-spec.md` Rejections Back to `@coding-specify` Unconditionally**:
   - *Rejection Rationale*: Forces unnecessary `specification.md` churn and extra approval gates when user feedback is purely visual, layout, or styling-related within the already-approved functional specification.
3. **Deterministic UI Design Stage Strictly Between Phase 2 and Phase 3 with Two-Branch Change-Request Loop (Selected)**:
   - *Rationale*: Establishes a single source of truth (`specification.md` -> `design-spec.md` -> `plan.md` -> `develop`), eliminates unnecessary overhead for non-UI tasks via dual-condition `DesignCheck`, prevents 60-second subagent timeouts via Dual-Mode Relay, and cleanly separates functional/scope changes (Branch A) from visual/styling iterations (Branch B).

---

## Decision

We establish a deterministic **UI Design Stage (`@coding-design`)** strictly between **Phase 2 (`coding-specify`)** and **Phase 3 (`coding-plan`)** in `agentic-coding-plugin`:

1. **Strict Stage Placement & Dual-Condition Trigger (`DesignCheck`)**:
   - All references suggesting `@coding-design` runs *"alongside `@coding-plan` and `@coding-develop`"* are removed.
   - `DesignCheck` is evaluated after `SpecApproved` (and in Pre-Clarified Fast-Path when `design-spec.md` is not yet approved).
   - `@coding-design` is invoked **only when BOTH** conditions hold:
     1. `agentic-design-plugin` is installed.
     2. The task involves UI/UX changes (wireframes, mockups, layouts, frontend components, design tokens, or PenPot inspections).
   - Non-UI tasks or environments without `agentic-design-plugin` skip directly to Phase 3 (`coding-plan`).
2. **Dual-Mode Validation Gate (`design-spec.md`)**:
   - `@coding-design` uses the Dual-Mode `[ARTIFACT_APPROVAL_REQUEST]` and `[QUESTION_RELAY_REQUEST]` protocols ([[architecture/adr-005-question-relay-protocol|ADR-005]], [[architecture/adr-006-artifact-approval-relay-protocol|ADR-006]], [[architecture/adr-008-dual-mode-coding-orchestrator|ADR-008]]).
   - `agentic-coding-plugin/rules/AGENTS.md` (Sections 6 & 7) explicitly includes `@coding-design` in worker subagent relay lists.
3. **Structured Design Change-Request Loop**:
   - Upon `status: "REJECTED"` at `DesignGate`, `@coding` evaluates the feedback against the approved `specification.md`:
     - **Branch A (Functional, Scope, Requirement, User-Flow, or Mixed Feedback)**: Route to `@coding-specify` (`P2`) with rejection feedback and current `specification.md` -> `specification.md` Dual-Mode gate -> if approved, re-invoke `@coding-design` -> `design-spec.md` Dual-Mode gate (if `specification.md` is rejected, follow `SpecRejected` -> `coding-clarify`).
     - **Branch B (Purely Visual, Layout, or Styling Feedback)**: Re-invoke `@coding-design` directly with the rejection feedback to update PenPot and `design-spec.md`, then re-present `DesignGate`.
4. **Pre-Clarified Fast-Path Consistency & Downstream Handoffs**:
   - When `specification.md` is already approved in Fast-Path, `@coding` checks whether `design-spec.md` is already approved (e.g., handoff from standalone `@coding-design`): if yes, advance to `P3`; otherwise route through `DesignCheck`.
   - Approved `design-spec.md` and PenPot references are passed to both `@coding-plan` (Phase 3) and `@coding-develop` (Phase 4).

---

## Consequences

- **Positive:**
  - 1:1 alignment between `SKILL.md` Section 3 (Fast-Path), Section 4 (Mermaid flowchart), Section 5 (Phase Handoffs), and `rules/AGENTS.md`.
  - Zero extra prompts or overhead for backend/CLI tasks or environments without `agentic-design-plugin`.
  - Clean separation between functional scope updates (Branch A) and visual iterations (Branch B) without infinite loops.
  - Preserves 100% timeout safety under Antigravity's 60-second subagent limit.
- **Negative:**
  - Adds a classification step (`DesignFeedbackEval`) for `@coding` when `design-spec.md` is rejected.
- **Risks & Mitigations:**
  - *Risk*: Mixed rejection feedback (both visual tweaks and new functional requirements) could leave `specification.md` stale if routed only to `@coding-design`.
  - *Mitigation*: Mixed feedback is explicitly classified as Branch A so `specification.md` remains the single source of truth before `@coding-design` regenerates `design-spec.md`.
