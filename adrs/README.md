---
title: "Architecture Decision Records Index"
date: "2026-11-26"
tags:
  - architecture-decisions
  - adrs
  - index
---

# Architecture Decision Records (ADRs)

Directory of formal Architecture Decision Records for the `agentic` workspace.

| ADR ID | Title | Status | Date | Primary Topics |
| :--- | :--- | :--- | :--- | :--- |
| [[architecture/adr-001-standardize-agent-handles\|ADR-001]] | Standardize Subagent Handles, Aliases, and Skill Identifiers to Coding Namespace | Accepted | 2026-10-26 | Agent handles, `@coding-*` namespace, skill standardization |
| [[architecture/adr-002-agent-display-names-and-plugin-renames\|ADR-002]] | Agent Display Names Standardization & Dual Plugin Repository Rename | Accepted | 2026-10-28 | Display names, `agentic-monitor-plugin`, `agentic-document-plugin` |
| [[architecture/adr-003-project-grooming-plugin\|ADR-003]] | Project Grooming Architecture, Skill-Only Execution, Grilling Protocol, Interactive Approval Gate, and Cross-Plugin Delegation | Accepted | 2026-10-31 | `/project-grooming`, 7-category grilling, interactive approval gate |
| [[architecture/adr-004-standardize-plugin-repository-names\|ADR-004]] | Standardize Antigravity Plugin Repository Names to Descriptive Action Grammar | Accepted | 2026-11-02 | Repository renames, descriptive action naming grammar |
| [[architecture/adr-005-question-relay-protocol\|ADR-005]] | Question Relay Protocol for Subagent User Inquiries | Accepted | 2026-11-05 | 60-second host timeout mitigation, question relay envelopes, subagent classification |
| [[architecture/adr-006-artifact-approval-relay-protocol\|ADR-006]] | Artifact Approval Relay Protocol for Subagent Approval Gates | Accepted | 2026-11-05 | 60-second host timeout mitigation, artifact approval relay envelopes, root orchestrator gate presentation |
| [[architecture/adr-007-coding-merge-remote-pr-and-sync\|ADR-007]] | Remote PR Creation, Question Relay Waiting Gate, and Post-Merge Synchronization for @coding-merge | Accepted | 2026-11-06 | Remote PR creation, GitHub CLI fallback compare URL, Question Relay waiting gate, post-merge base branch sync & branch deletion |
| [[architecture/adr-008-dual-mode-coding-orchestrator\|ADR-008]] | Dual-Mode Coding Orchestrator with Multi-Hop Relay and Anti-Direct-Coding Guardrails | Accepted | 2026-11-06 | Dual-mode orchestration, multi-hop question & artifact approval relay, anti-direct-coding guardrails, pre-clarified fast-path |
| [[architecture/adr-009-coding-merge-complete-branch-cleanup-and-remote-sync\|ADR-009]] | Complete Branch Cleanup, Base Branch Checkout, and Remote Tracking Prune for @coding-merge | Accepted | 2026-11-08 | Local branch cleanup, remote branch deletion (`git push origin --delete`), remote tracking prune (`git fetch --prune origin`), base branch verification |
| [[architecture/adr-010-coding-review-plan-audit-and-unplanned-changes-gate\|ADR-010]] | Plan-vs-Changes Audit, Non-Blocking Review Artifact, and Unplanned Changes Validation Gate for @coding-review | Accepted | 2026-11-17 | Plan-vs-Changes git audit, `review-<slug>.md` non-blocking artifact, Unplanned Changes Validation Gate via Question Relay, rework routing |
| [[architecture/adr-011-worktree-based-development-lifecycle\|ADR-011]] | Worktree-Based Development Lifecycle (.worktrees/<slug> Isolation) | Accepted | 2026-11-18 | Isolated `.worktrees/<slug>` development, `main`/`master` primary repo invariant, `worktree_path` propagation, `git worktree remove` & `prune` cleanup |
| [[architecture/adr-012-exclusive-code-change-authority\|ADR-012]] | Exclusive Code-Change Authority for @coding-develop Across All Agentic Plugins | Accepted | 2026-11-18 | Exclusive `edit_file` on `@coding-develop`, scoped `write_file`, non-mutating `run_command`, `/project-grooming` manual fallback |
| [[architecture/adr-013-coding-design-plugin-and-penpot-mcp\|ADR-013]] | Coding Design Plugin (@coding-design) with Official PenPot MCP Integration | Accepted | 2026-11-21 | `agentic-design-plugin`, `@coding-design`, PenPot MCP, Dual-Mode Relay, ADR-012 boundary, `@coding` / `@coding-document` / triage integration |
| [[architecture/adr-014-ui-design-stage-and-change-request-loop\|ADR-014]] | UI Design Stage & Change-Request Loop Between Specification and Planning | Accepted | 2026-11-25 | Strict `@coding-design` placement, dual-condition `DesignCheck`, Dual-Mode `design-spec.md` gate, Branch A/B change-request loop, Fast-Path consistency |
| [[architecture/adr-015-minigma-ai-web-app-spartan-ui-and-screen-cleanup\|ADR-015]] | Minigma AI Web App: spartan/ui + Tailwind CSS 4 Migration, Dual-Theme Tokens, and Route Surface Reduction | Accepted | 2026-11-25 | `@angular/material` phase-out, `spartan/ui` + Tailwind CSS 4, Late Sun / Twilight dual-theme tokens, Auth/Onboarding/Chat route reduction, `.worktrees/penpot-ui-alignment` |
| [[architecture/adr-016-obsidian-vault-worktree-and-pr-lifecycle\|ADR-016]] | Obsidian Vault Git Worktree & PR Lifecycle for @coding-document | Accepted | 2026-11-26 | Self-managed `*-obsidian/` `.worktrees/<slug>` on `docs/<slug>`, `defer_pr` multi-phase accumulation, Remote/Local/Non-Git release, Hybrid PR Waiting Gate, Dual-Mode Question Relay |

| [[architecture/adr-017-semantic-laya-triage-and-parallel-choice-noul-decomposition\|ADR-017]] | Semantic Laya Triage & Parallel `choice` + `noul` Decomposition | Accepted | 2026-11-26 | `agentic-prompt-triage-plugin`, semantic `criteria` keys, parallel `choice` + `noul`, `requires_changes` / `is_mixed` gating, `usage` diagnostics, calibrated `confidence` |