---
title: "Workspace Features Index"
date: "2026-10-31"
tags:
  - features
  - index
---

# Workspace Features Index

Directory of cross-cutting features and specifications documented across the `agentic` workspace.

| Feature Note | Phase | Status | Summary |
| :--- | :--- | :--- | :--- |
| [[question-relay-protocol\|Question Relay Protocol for Subagent User Prompts]] | Phase 7: Merge & Release | Released / Merged | Standardize an event-driven Question Relay Protocol (`[QUESTION_RELAY_REQUEST]` / `[QUESTION_RELAY_RESPONSE]`) across all companion plugins to eliminate the 60-second host timeout on subagents invoking `ask_question`. Defines Pure Subagents (`@coding-clarify`, `@coding-review`, `@coding-deploy`) vs Dual-Mode Subagents (`@coding-investigate`, `/project-grooming`, `@coding-monitor`, `@coding-research`), routes human inquiries through top-level orchestrators, and spans 6 repositories. Fast-forward merged into `master` across all repositories (`695e1b1`, `d2c822c`, `df03d3c`, `a2e7932`, `23a101e`, `efb067d`) with 147/147 unit tests passing workspace-wide (60/60 across feature plugins). Formalized in [[architecture/adr-005-question-relay-protocol\|ADR-005]]. |
| [[rename-plugin-repositories\|Rename Plugin Repositories to Descriptive Action-Oriented Names]] | Phase 6: Merge & Release | Released / Merged | Standardize 5 companion plugin repositories to descriptive action-oriented names (`agentic-prompt-triage-plugin`, `agentic-issue-investigation-plugin`, `agentic-task-monitoring-plugin`, `agentic-topic-researching-plugin`, `agentic-project-documenting-plugin`). Renamed remote GitHub repositories, local directories, package manifests, and Obsidian documentation. Remediated AC-8 dynamic triage detection and purged stale companion references across 5 `bugfix/*` branches (`9c5c19b`, `00b7d03`, `8f46739`, `76f7002`, `ee5bfa0`). All branches merged into `master`, bugfix branches deleted, and all 135/135 unit tests passing workspace-wide. Formalized in [[architecture/adr-004-standardize-plugin-repository-names\|ADR-004]]. |
| [[project-grooming-plugin\|Project Grooming Plugin & Skill (/project-grooming)]] | Phase 6: Merge & Release | Released / Completed | Standalone Antigravity companion plugin (`agentic-project-grooming-plugin`) featuring skill `/project-grooming`. Conducts Matt Pocock `/grill-me` style single-question interrogation across 7 core requirements categories, writes interactive approval artifact `Project_Request_Grooming.md` (`RequestFeedback=True`, sole tool call), and orchestrates scaffolding across `@coding`, `@coding-document`, and `agentic-prompt-triage-plugin`. Formalized in [[architecture/adr-003-project-grooming-plugin\|ADR-003]]. Fast-forward merged into `master` across all repositories (`2bbd11f`, `c6fd560`, `d2e11c5`), feature branches deleted, and all 117/117 unit tests passing across all plugins. |
| [[agent-display-names-and-plugin-renames\|Agent Display Names & Plugin Renames (Monitor & Document)]] | Phase 6: Merge & Release | Released / Completed | Standardize agent display titles to prefix `Coding - ` (setting `@coding-monitor` to `Coding - Monitor Expert`), preserve technical identifiers (`name: coding-*`), rename `agentic-monitoring-plugin` -> `agentic-monitor-plugin` and `agentic-obsidian-plugin` -> `agentic-document-plugin`, update Git remote origins to github.com:gpfister/*, and execute fast-forward merge to `master` across all 8 repositories (101/101 tests passing on `master`). |
| [[standardize-agent-handles\|Standardize Agent Handles]] | Phase 6: Merge & Release | Released / Completed | Standardize all agent handles, aliases, skills, and agent directories to strictly use the `@coding-*` prefix and purge non-coding aliases across all 8 repositories (86/86 tests passing on master). |
