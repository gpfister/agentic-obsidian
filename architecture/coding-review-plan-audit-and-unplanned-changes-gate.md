---
title: "Plan-vs-Changes Audit, review-<slug>.md Artifact, and Unplanned Changes Validation Gate Architecture Plan"
date: "2026-11-17"
status: "Merged & Released"
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
  - architecture
  - implementation-plan
  - coding-review
  - plan-audit
  - unplanned-changes
  - question-relay
  - approved-plan
  - phase-3-completed
affected_plugins:
  - agentic-coding-plugin
aliases:
  - coding-review-plan-audit-and-unplanned-changes-gate
  - coding-review-plan-audit-architecture-plan
---

# Plan-vs-Changes Audit, `review-<slug>.md` Artifact, and Unplanned Changes Validation Gate Architecture Plan

> [!NOTE] Lifecycle Status: Merged & Released — `MERGED` (`SPEC-2026-REVIEW-001`)
> - **Specification ID**: `SPEC-2026-REVIEW-001`
> - **Architecture Decision Record**: [[architecture/adr-010-coding-review-plan-audit-and-unplanned-changes-gate|ADR-010: Plan-vs-Changes Audit & Unplanned Changes Validation Gate]]
> - **Feature Specification Note**: [[features/coding-review-plan-audit-and-unplanned-changes-gate|Feature: Plan-vs-Changes Audit & Unplanned Changes Validation Gate]]
> - **Target Repository**: [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin)
> - **Git Branch**: `main` ([PR #4](https://github.com/gpfister/agentic-coding-plugin/pull/4) merged at [`151f203`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/); `feature/review-plan-audit` deleted locally and remotely)
> - **Current Phase**: Phase 7: Merge & Release Complete (Post-Merge Synchronized; 25/25 tests passing)
> - **Approved Spec**: [`specification.md`](file:///Users/gpfister/.gemini/antigravity/brain/62cf3f67-ad0d-4389-b4db-06c25e813360/specification.md) ([Vault Archive](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/sessions/attachments/62cf3f67-ad0d-4389-b4db-06c25e813360/specification.md))
> - **Approved Plan**: [`plan.md`](file:///Users/gpfister/.gemini/antigravity/brain/62cf3f67-ad0d-4389-b4db-06c25e813360/plan.md) ([Vault Archive](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/sessions/attachments/62cf3f67-ad0d-4389-b4db-06c25e813360/plan.md))
> - **Review Audit Artifact**: [`review-review-plan-audit.md`](file:///Users/gpfister/.gemini/antigravity/brain/21467160-4bf7-47c9-beed-69014ecd2ea6/review-review-plan-audit.md) ([Vault Archive](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/sessions/attachments/62cf3f67-ad0d-4389-b4db-06c25e813360/review-review-plan-audit.md))

---

## 1. Repository & Scope Breakdown

- **Repository**: [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin) (`/Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin`)
- **Git Branch**: `feature/review-plan-audit`
- **Local Deployment**: Not Required (no Dockerfile, Docker Compose, Terraform, Kubernetes/Helm manifests, local deployment scripts, or Makefile deploy targets exist in this repository).

---

## 2. Concrete Implementation Plan

### Step 1: Update `@coding-review` Subagent Definition
- **File**: [`agents/coding-review/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-review/agent.md)
- **Changes**:
  1. **Tooling Permission (`FR-2.6`, `NFR-2`)**: Add `write_file` to `tools:` in YAML frontmatter with an explicit **Artifact-Only Write Guardrail** restricting `write_file` strictly to `<appDataDir>/brain/<conversation-id>/review-<slug>.md` and forbidding writes to repository source code, configurations, or tests.
  2. **Plan-vs-Changes Audit (`FR-1.2`, `FR-1.3`, `NFR-3`)**: Inspect `git status` and `git diff` against base branch / merge base across all repositories in `plan.md` (and any additional modified workspace repos); classify deviations from `plan.md` as *unplanned changes* with concise justifications.
  3. **Review Artifact Generation (`FR-2.1`–`FR-2.5`, `NFR-1`, `NFR-4`)**: Derive kebab-case `<slug>` (`[a-z0-9-]+`, stripping `feature/`, `task/`, `bugfix/`, fallback `review-changes.md`) and write `review-<slug>.md` on every review run **without** `RequestFeedback=True` and **without** `[ARTIFACT_APPROVAL_REQUEST]`, including all 5 mandatory sections (reporting `"None"` when no unplanned changes exist).
  4. **Unplanned Changes Validation Gate & Mandatory `[REVIEW_ARTIFACT]` Ordering (`FR-3.1`–`FR-3.4`, `NFR-1`)**: Trigger only when unplanned changes exist; immediately after writing `review-<slug>.md` and **before** dispatching `[QUESTION_RELAY_REQUEST]`, send `[REVIEW_ARTIFACT]` (with the absolute `Path` and `Summary`) to `@coding` in Subagent Mode (or present/link `review-<slug>.md` before `ask_question` in Standalone Mode). Then prompt `"Do you approve the changes ?"` with `(Recommended) Yes`, `No, revert all these changes`, and native custom write-in support via `[QUESTION_RELAY_REQUEST]` (subagent mode) or `ask_question` (standalone mode). Return `ReviewReport` for revert/revision delegation to `@coding-develop` (never run `git checkout`/`git restore` directly).
  5. **Separate Merge Confirmation Gate (`FR-4.1`)**: Execute only after unplanned changes (if any) are approved and all quality, test, linter, and deployment checks pass.

### Step 2: Update `@coding` Orchestrator Definition & Skill
- **Files**:
  - [`agents/coding/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding/agent.md)
  - [`skills/coding-orchestrator/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/coding-orchestrator/SKILL.md)
- **Changes**:
  1. **Phase 6 Handoff (`FR-1.1`)**: Pass the approved `plan.md` path alongside `specification.md`, `DevelopReport`s, and `DeployReport`s when invoking `@coding-review`.
  2. **`[REVIEW_ARTIFACT]` Forwarding Before `[QUESTION_RELAY_REQUEST]`**: When `@coding` receives `[REVIEW_ARTIFACT]` from `@coding-review` before `[QUESTION_RELAY_REQUEST]`, forward `[REVIEW_ARTIFACT]` immediately to `caller_id` via `send_message` in Subagent Mode (so the parent/main agent can read `review-<slug>.md` before the question arrives), or present/link `review-<slug>.md` to the user before `ask_question` in Standalone Mode.
  3. **Rework Routing (`FR-3.4`)**: Route `ReviewReport` outcomes for `No, revert all these changes` or custom write-in revision instructions to `@coding-develop` (and `@coding-deploy` if applicable), then re-run `@coding-review`.
  4. **Flowchart & Gate Separation (`FR-4.1`)**: Update the Phase 6 Mermaid flowchart and stage descriptions to reflect `review-<slug>.md` generation, the conditional Unplanned Changes Validation Gate, and the separate Merge Confirmation Gate.

### Step 3: Update Central Agent Catalog
- **File**: [`agents/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/agent.md)
- **Changes**:
  1. Update the `@coding-review` row in the Allowed Tools table to include `write_file` (scoped to `review-<slug>.md` artifacts) and summarize Plan-vs-Changes auditing.
  2. Update `### Stage 6: Review (coding-review)` to document `plan.md` handoff, `review-<slug>.md` generation, the Unplanned Changes Validation Gate, and revert/revision rework routing via `@coding-develop`.

### Step 4: Add Automated Verification Tests
- **File**: [`tests/test_plugin.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/tests/test_plugin.py)
- **Changes**:
  1. Verify `coding-review/agent.md` includes `write_file` in `tools:`, artifact-only write restrictions, `git status`/`git diff` comparison against `plan.md`, `review-<slug>.md` generation without `RequestFeedback=True`/`[ARTIFACT_APPROVAL_REQUEST]`, `"None"` reporting, all 5 mandatory sections, and kebab-case slug normalization.
  2. Verify the Unplanned Changes Validation Gate (`"Do you approve the changes ?"`, `(Recommended) Yes`, `No, revert all these changes`, custom write-in handling, `[QUESTION_RELAY_REQUEST]`) and separation from Merge Confirmation.
  3. Verify `agents/coding/agent.md`, `skills/coding-orchestrator/SKILL.md`, and `agents/agent.md` document `plan.md` handoff and rework routing to `@coding-develop`.

### Verification Command
```bash
cd /Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin && uv run pytest tests/test_plugin.py
```
