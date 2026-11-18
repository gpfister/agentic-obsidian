---
adr_id: "010"
title: "Plan-vs-Changes Audit, Non-Blocking Review Artifact, and Unplanned Changes Validation Gate for @coding-review"
status: "accepted"
date: "2026-11-17"
specification_id: "SPEC-2026-REVIEW-001"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/62cf3f67-ad0d-4389-b4db-06c25e813360/specification.md"
vault_specification_path: "sessions/attachments/62cf3f67-ad0d-4389-b4db-06c25e813360/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/62cf3f67-ad0d-4389-b4db-06c25e813360/plan.md"
vault_plan_path: "sessions/attachments/62cf3f67-ad0d-4389-b4db-06c25e813360/plan.md"
tags:
  - architecture-decision
  - coding-review
  - plan-audit
  - unplanned-changes
  - question-relay
  - quality-gate
  - plan-approved
  - merged
  - released
merge_pr: "https://github.com/gpfister/agentic-coding-plugin/pull/4"
merge_commit: "151f203"
aliases:
  - ADR-010
  - adr-010-coding-review-plan-audit-and-unplanned-changes-gate
---

# ADR-010: Plan-vs-Changes Audit, Non-Blocking Review Artifact, and Unplanned Changes Validation Gate for `@coding-review`

## Context

In the `agentic-coding-plugin` 7-stage software development lifecycle, Stage 6 ([`@coding-review`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-review/agent.md)) audits `DevelopReport`s, `DeployReport`s, specification compliance, cross-repo contracts, unit tests, and linters before advancing to the Merge Confirmation Gate.

Previously, `@coding-review` exhibited three structural gaps:
1. **No Plan-vs-Changes Verification**: `@coding` did not pass the approved `plan.md` to `@coding-review`, and `@coding-review` did not systematically inspect actual repository git changes (`git status`, `git diff`) against `plan.md` to detect scope drift or unplanned modifications introduced during implementation or deployment fixes.
2. **No Persistent Review Artifact**: Review findings were returned only in an ephemeral `ReviewReport` message, leaving no persistent audit record in `<appDataDir>/brain/<conversation-id>/`.
3. **No Unplanned Changes Validation Gate**: Users were never explicitly asked to validate unplanned changes before reaching the Merge Confirmation Gate, nor was there a structured rework loop to revert unplanned changes or apply custom revision instructions via `@coding-develop`.

### Alternatives Considered

1. **Require Interactive Artifact Approval (`RequestFeedback=True`) on `review-<slug>.md`**:
   - *Rejection Rationale*: Adding an interactive artifact approval gate on `review-<slug>.md` would introduce redundant modal friction alongside the Question Relay gates (Unplanned Changes Validation Gate and Merge Confirmation Gate) and risk 60-second subagent timeouts if invoked directly by `@coding-review`.
2. **Allow `@coding-review` to Revert Unplanned Changes Directly via `git checkout` / `git restore`**:
   - *Rejection Rationale*: Violates the strict read-only boundary of `@coding-review` and bypasses `@coding-develop` / `@coding-deploy` test and deployment re-verification.
3. **Plan-vs-Changes Git Audit + Non-Blocking `review-<slug>.md` + Conditional Question Relay Validation Gate (Selected)**:
   - *Rationale*: Preserves repository read-only integrity by scoping `write_file` strictly to `<appDataDir>/brain/<conversation-id>/review-<slug>.md` without `RequestFeedback=True`, enforces multi-repo git diff auditing against `plan.md`, and uses the established Question Relay Protocol ([[architecture/adr-005-question-relay-protocol|ADR-005]]) to validate unplanned changes before the separate Merge Confirmation Gate.

---

## Decision

We formalize the **Plan-vs-Changes Audit, Non-Blocking Review Artifact, and Unplanned Changes Validation Gate** architecture for Stage 6 ([`@coding-review`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-review/agent.md)) and the [`@coding`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding/agent.md) orchestrator:

```mermaid
flowchart TD
    A["Phase 6: @coding passes plan.md, specification.md, DevelopReports, DeployReports to @coding-review"] --> B["Inspect git status & git diff across all repos vs. plan.md"]
    B --> C["Verify Spec Compliance, Cross-Repo Contracts, Tests, Linters, Deploy Health"]
    C --> D["Write review-&lt;slug&gt;.md (No RequestFeedback, No ARTIFACT_APPROVAL_REQUEST)"]
    D --> E{"Unplanned Changes Detected?"}
    E -- No ("None" in artifact) --> F{"All Quality & Deploy Checks Pass?"}
    E -- Yes --> G0["Send [REVIEW_ARTIFACT] to @coding (forwarded to caller_id)<br/>OR present review-&lt;slug&gt;.md in Standalone Mode"]
    G0 --> G["Unplanned Changes Validation Gate via Question Relay<br/>'Do you approve the changes ?'"]
    G -- "(Recommended) Yes" --> F
    G -- "No, revert all these changes" --> H["ReviewReport -> @coding -> @coding-develop (Revert + Re-test)"]
    G -- "Custom write-in text" --> I["ReviewReport -> @coding -> @coding-develop (Apply revisions + Re-test)"]
    H --> A
    I --> A
    F -- Yes --> J["Separate Merge Confirmation Gate"]
    F -- No --> K["ReviewReport -> @coding -> @coding-develop (Fix defects)"]
    K --> A
```

### Key Architectural Rules
1. **Orchestrator Context Handoff**: `@coding` **MUST** pass the approved `plan.md` path alongside `specification.md`, `DevelopReport`s, and `DeployReport`s when invoking `@coding-review`.
2. **Scoped `write_file` Permission**: `coding-review` is granted `write_file` strictly restricted to `<appDataDir>/brain/<conversation-id>/review-<slug>.md`. Repository source files, configurations, and tests remain strictly read-only.
3. **Deterministic Non-Blocking Review Artifact**: `@coding-review` **MUST** write `review-<slug>.md` (kebab-case `[a-z0-9-]+`, stripping `feature/`, `task/`, `bugfix/`, fallback `review-changes.md`) on every review run **without** `RequestFeedback=True` and **without** `[ARTIFACT_APPROVAL_REQUEST]`, containing all 5 mandatory sections (reporting `"None"` when no unplanned changes exist).
4. **Conditional Unplanned Changes Validation Gate & Mandatory `[REVIEW_ARTIFACT]` Ordering**: Triggered **only** when unplanned changes exist. Immediately after writing `review-<slug>.md` and **before** dispatching `[QUESTION_RELAY_REQUEST]`, `@coding-review` sends `[REVIEW_ARTIFACT]` (with `Path` and `Summary`) to `@coding`, which forwards it to `caller_id` in Subagent Mode (or presents/links `review-<slug>.md` before `ask_question` in Standalone Mode) so the parent/main agent can read the review artifact before the question reaches the user. `@coding-review` then asks `"Do you approve the changes ?"` with `(Recommended) Yes`, `No, revert all these changes`, and native custom write-in support via `[QUESTION_RELAY_REQUEST]` (subagent mode) or `ask_question` (standalone mode).
5. **Delegated Rework & Separate Merge Gate**: Reverts and custom revisions are never executed by `@coding-review`; they are returned in `ReviewReport` so `@coding` delegates them to `@coding-develop` (and `@coding-deploy` if applicable) before re-running `@coding-review`. Merge Confirmation remains a separate final gate.

---

## Consequences

- **Positive:**
  - Guarantees systematic detection and documentation of scope drift across all participating repositories.
  - Produces a persistent, auditable `review-<slug>.md` artifact for every review cycle without introducing timeout risks or extra approval modals.
  - Gives users explicit control to accept, revert, or refine unplanned changes prior to merge confirmation.
- **Negative:**
  - Adds a conditional interactive Question Relay turn whenever unplanned changes are detected.
- **Risks:**
  - *Risk*: `@coding-review` accidentally writing to repository files now that `write_file` is in `tools:`.
  - *Mitigation*: Explicit Artifact-Only Write Guardrail in `coding-review/agent.md` and automated unit tests in `tests/test_plugin.py` verifying the restriction.
