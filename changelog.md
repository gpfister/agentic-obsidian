---
title: "Workspace Release Changelog"
date: "2026-11-04"
status: "active"
tags:
  - changelog
  - releases
  - multi-repo
---

# Workspace Release Changelog

Chronological log of cross-repository releases, feature merges, and architectural upgrades across the `agentic` workspace.

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
