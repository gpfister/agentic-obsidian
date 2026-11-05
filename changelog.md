---
title: "Workspace Release Changelog"
date: "2026-11-05"
status: "active"
tags:
  - changelog
  - releases
  - multi-repo
  - question-relay
  - artifact-approval-relay
---

# Workspace Release Changelog

Chronological log of cross-repository releases, feature merges, and architectural upgrades across the `agentic` workspace.

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

### Overview & Key Capabilities Delivered
Standardized an event-driven Question Relay Protocol (`[QUESTION_RELAY_REQUEST]` / `[QUESTION_RELAY_RESPONSE]`) across the multi-agent workspace to completely eliminate premature 60-second host execution timeout terminations on subagents invoking user-prompting tools:
- **Pure Subagents vs. Dual-Mode Subagents**:
  - Pure subagents (`@coding-clarify`, `@coding-deploy`, `@coding-review`) omit `ask_question` from tools frontmatter and relay all inquiries through their parent caller via `send_message`.
  - Dual-mode subagents (`@coding-investigate`, `/project-grooming` `@coding-groom`, `@coding-monitor`, `@coding-research`) check caller ID dynamically: they dispatch relay envelopes when spawned under an orchestrator, or prompt directly via `ask_question` in standalone root sessions.
- **Top-Level Orchestration Relay Handling**:
  - Orchestrator (`@coding`) intercepts `[QUESTION_RELAY_REQUEST]` envelopes, prompts the user interactively at the root session turn without host timeout limits, and dispatches `[QUESTION_RELAY_RESPONSE]` to reactively awaken worker subagents.
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
