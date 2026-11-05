---
title: "Workspace Release Changelog"
date: "2026-11-05"
status: "active"
tags:
  - changelog
  - releases
  - multi-repo
  - question-relay
---

# Workspace Release Changelog

Chronological log of cross-repository releases, feature merges, and architectural upgrades across the `agentic` workspace.

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
