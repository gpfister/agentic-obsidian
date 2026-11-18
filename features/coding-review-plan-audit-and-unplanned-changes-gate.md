---
title: "Plan-vs-Changes Audit, review-<slug>.md Artifact, and Unplanned Changes Validation Gate for @coding-review"
date: "2026-11-17"
status: "Merged / Released"
phase: "Phase 7: Merge & Release Complete (Post-Merge Synchronized)"
specification_id: "SPEC-2026-REVIEW-001"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/62cf3f67-ad0d-4389-b4db-06c25e813360/specification.md"
vault_specification_path: "sessions/attachments/62cf3f67-ad0d-4389-b4db-06c25e813360/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/62cf3f67-ad0d-4389-b4db-06c25e813360/plan.md"
vault_plan_path: "sessions/attachments/62cf3f67-ad0d-4389-b4db-06c25e813360/plan.md"
review_artifact_path: "/Users/gpfister/.gemini/antigravity/brain/21467160-4bf7-47c9-beed-69014ecd2ea6/review-review-plan-audit.md"
vault_review_artifact_path: "sessions/attachments/62cf3f67-ad0d-4389-b4db-06c25e813360/review-review-plan-audit.md"
adr: "[[architecture/adr-010-coding-review-plan-audit-and-unplanned-changes-gate|ADR-010: Plan-vs-Changes Audit, Non-Blocking Review Artifact, and Unplanned Changes Validation Gate for @coding-review]]"
tags:
  - feature
  - coding-review
  - plan-audit
  - unplanned-changes
  - question-relay
  - specification
  - approved-spec
  - implementation-plan
  - approved-plan
  - phase-1-completed
  - phase-2-completed
  - phase-3-completed
  - phase-4-completed
  - phase-5-completed
  - phase-6-completed
  - phase-7-completed
  - review-passed
  - merged
  - released
affected_plugins:
  - agentic-coding-plugin
aliases:
  - coding-review-plan-audit-and-unplanned-changes-gate
  - SPEC-2026-REVIEW-001
---

# Feature: Plan-vs-Changes Audit, `review-<slug>.md` Artifact, and Unplanned Changes Validation Gate for `@coding-review`

> [!NOTE] Lifecycle Status: Merged & Released — `MERGED` (`SPEC-2026-REVIEW-001`)
> - **Specification ID**: `SPEC-2026-REVIEW-001`
> - **Architecture Decision Record**: [[architecture/adr-010-coding-review-plan-audit-and-unplanned-changes-gate|ADR-010: Plan-vs-Changes Audit & Unplanned Changes Validation Gate]]
> - **Architecture Plan Note**: [[architecture/coding-review-plan-audit-and-unplanned-changes-gate|Plan-vs-Changes Audit & Review Artifact Architecture Plan]]
> - **Target Repository**: [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin) ([PR #4](https://github.com/gpfister/agentic-coding-plugin/pull/4) merged into `main` at [`151f203`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/); `feature/review-plan-audit` deleted locally and remotely)
> - **Current Phase**: Phase 7: Merge & Release Complete (Post-Merge Synchronized; 25/25 tests passing)
> - **Approved Spec**: [`specification.md`](file:///Users/gpfister/.gemini/antigravity/brain/62cf3f67-ad0d-4389-b4db-06c25e813360/specification.md) ([Vault Archive](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/sessions/attachments/62cf3f67-ad0d-4389-b4db-06c25e813360/specification.md))
> - **Approved Plan**: [`plan.md`](file:///Users/gpfister/.gemini/antigravity/brain/62cf3f67-ad0d-4389-b4db-06c25e813360/plan.md) ([Vault Archive](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/sessions/attachments/62cf3f67-ad0d-4389-b4db-06c25e813360/plan.md))
> - **Review Audit Artifact**: [`review-review-plan-audit.md`](file:///Users/gpfister/.gemini/antigravity/brain/21467160-4bf7-47c9-beed-69014ecd2ea6/review-review-plan-audit.md) ([Vault Archive](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/sessions/attachments/62cf3f67-ad0d-4389-b4db-06c25e813360/review-review-plan-audit.md))

---

## 1. Feature Summary & Motivation

In `agentic-coding-plugin`, Phase 6 ([`@coding-review`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-review/agent.md)) previously audited `DevelopReport`s, `DeployReport`s, specification compliance, cross-repo contracts, unit tests, and linters, then proceeded directly to the Merge Confirmation Gate. It did not systematically compare actual repository git changes (`git diff`, `git status`) against the approved `plan.md`, did not produce a persistent review artifact, and did not prompt the user when unplanned changes were introduced during implementation or deployment fixes.

This feature updates `@coding-review` and `@coding` to:
1. Audit all actual git changes across participating repositories against the initial approved `plan.md`.
2. Generate a persistent, non-blocking review artifact (`review-<slug>.md`) on every review execution.
3. Introduce a conditional **Unplanned Changes Validation Gate** via the Question Relay Protocol whenever changes outside `plan.md` are detected, prior to the separate **Merge Confirmation Gate**.

---

## 2. Functional Requirements

### FR-1: Plan-vs-Changes Audit
- **FR-1.1 Orchestrator Context Handoff**: When `@coding` invokes `@coding-review` in Phase 6, it **MUST** provide the path to the approved `plan.md` alongside `specification.md`, `DevelopReport`s, and `DeployReport`s.
- **FR-1.2 Git Change Inspection**: `@coding-review` **MUST** inspect actual repository changes across all participating repositories (`git status`, `git diff` against base branch / merge base) in addition to reviewing `SpecificationDocument`, `DevelopReport`s, `DeployReport`s, cross-repo contracts, unit tests, and linters.
- **FR-1.3 Unplanned Change Detection**: Compare observed file modifications, additions, deletions, and behavioral changes against the scope, affected files, and tasks defined in `plan.md`. Any change not covered by `plan.md` **MUST** be classified as an *unplanned change* with a concise justification (e.g., discovered dependency, bugfix during deployment, refactor, or unexplained drift).

### FR-2: Review Artifact (`review-<slug>.md`)
- **FR-2.1 Filename & Slug Derivation**: Single combined artifact per review named `review-<slug>.md`, where `<slug>` is derived in `kebab-case` (`[a-z0-9-]+`) from the feature, task, or branch name, stripping prefixes (`feature/`, `task/`, `bugfix/`) and falling back to `review-changes.md`.
- **FR-2.2 Artifact Location**: `<appDataDir>/brain/<conversation-id>/review-<slug>.md`.
- **FR-2.3 Generation Frequency**: Generated on **every** review run. When no unplanned changes are detected, the Plan vs. Actual Changes Audit section **MUST** explicitly report `"None"`.
- **FR-2.4 No Approval Gate Required**: Written directly using `write_file` **without** `RequestFeedback=True` and **without** dispatching `[ARTIFACT_APPROVAL_REQUEST]`.
- **FR-2.5 Required Artifact Contents**:
  1. **Plan vs. Actual Changes Audit** (`"None"` if no unplanned changes exist)
  2. **Specification Compliance Check**
  3. **Cross-Repo Interface & Schema Compatibility**
  4. **Unit Tests & Linter Verification Results**
  5. **Local Deployment Health Status**
- **FR-2.6 Tooling Permission**: `write_file` added to `coding-review`'s allowed tools, strictly scoped to `<appDataDir>/brain/<conversation-id>/review-<slug>.md`; repository source files, configurations, and tests remain strictly read-only.

### FR-3: Unplanned Changes Validation Gate
- **FR-3.1 Trigger Condition**: Triggered **only** when one or more unplanned changes are detected; skipped otherwise.
- **FR-3.2 Question & Options**:
  - **Question**: `"Do you approve the changes ?"`
  - **Options**: `(Recommended) Yes`, `No, revert all these changes` *(native custom write-in text supported)*
  - **Context**: Concise summary of detected unplanned changes and reference to `review-<slug>.md`.
- **FR-3.3 Relay Protocol & Mandatory `[REVIEW_ARTIFACT]` Ordering**:
  - **Subagent Mode**: Immediately after writing `review-<slug>.md` and **before** dispatching `[QUESTION_RELAY_REQUEST]`, `@coding-review` **MUST** send `[REVIEW_ARTIFACT]` (containing the absolute `Path` to `review-<slug>.md` and a concise `Summary`) to `@coding`, which immediately forwards `[REVIEW_ARTIFACT]` to `caller_id` via `send_message` so the parent/main agent can read the review artifact before the question reaches the user. `@coding-review` then dispatches `[QUESTION_RELAY_REQUEST]` to `@coding`.
  - **Standalone Fallback Mode**: Present/link `review-<slug>.md` (using a clickable `file://` link) to the user **before** calling `ask_question` directly.
- **FR-3.4 Decision Branching**:
  - `Yes` / `(Recommended) Yes`: Accept unplanned changes and proceed to the separate **Merge Confirmation Gate** (if all other checks pass).
  - `No, revert all these changes`: Return `ReviewReport` instructing `@coding` to delegate reverting all unplanned changes to `@coding-develop`, re-run tests/linters (and `@coding-deploy` if applicable), and re-verify via `@coding-review`.
  - Custom write-in text: Return `ReviewReport` with verbatim revision instructions for `@coding` -> `@coding-develop` -> verification -> `@coding-review`.

### FR-4: Merge Confirmation Gate
- **FR-4.1 Separation of Gates**: Remains a separate final gate executed **only after** unplanned changes (if any) are approved and all quality, test, linter, and deployment checks pass.

---

## 3. Non-Functional Requirements & Edge Cases

- **NFR-1 Timeout Safety**: Never call `ask_question` or `write_file(RequestFeedback=True)` directly in subagent mode.
- **NFR-2 Source Code Integrity**: `@coding-review` never modifies repository files or runs `git checkout` / `git restore` directly; all reverts and edits are delegated to `@coding-develop`.
- **NFR-3 Multi-Repository Consistency**: Covers every repository listed in `plan.md` plus any additional workspace repos modified during the session.
- **NFR-4 Deterministic Slug Formatting**: `[a-z0-9-]+`, stripping `feature/`, `task/`, `bugfix/`, fallback `review-changes.md`.
- **Re-review after revert or revision**: Overwrites/updates `review-<slug>.md` with the latest git diff and test results.
