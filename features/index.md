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
| [[project-grooming-plugin\|Project Grooming Plugin & Skill (/project-grooming)]] | Phase 5: Review / Pre-Merge | Review Completed (Ready for Merge) | Standalone Antigravity companion plugin (`agentic-project-grooming-plugin`) featuring skill `/project-grooming`. Conducts Matt Pocock `/grill-me` style single-question interrogation across 7 core requirements categories, writes interactive approval artifact `Project_Request_Grooming.md` (`RequestFeedback=True`, sole tool call), and orchestrates scaffolding across `@coding`, `@coding-document`, and `agentic-triage-plugin`. Formalized in [[architecture/adr-003-project-grooming-plugin\|ADR-003]]. Phase 5 Review completed: 100% acceptance criteria AC-1 through AC-5 verified, clean code review passed, and all 117/117 unit tests passing across all plugins. Ready for Phase 6 merge to `master`. |
| [[agent-display-names-and-plugin-renames\|Agent Display Names & Plugin Renames (Monitor & Document)]] | Phase 6: Merge & Release | Released / Completed | Standardize agent display titles to prefix `Coding - ` (setting `@coding-monitor` to `Coding - Monitor Expert`), preserve technical identifiers (`name: coding-*`), rename `agentic-monitoring-plugin` -> `agentic-monitor-plugin` and `agentic-obsidian-plugin` -> `agentic-document-plugin`, update Git remote origins to github.com:gpfister/*, and execute fast-forward merge to `master` across all 8 repositories (101/101 tests passing on `master`). |
| [[standardize-agent-handles\|Standardize Agent Handles]] | Phase 6: Merge & Release | Released / Completed | Standardize all agent handles, aliases, skills, and agent directories to strictly use the `@coding-*` prefix and purge non-coding aliases across all 8 repositories (86/86 tests passing on master). |
