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
| [[transparent-terminal-background\|Transparent Terminal Background in agentic-coding-tui]] | Phase 4: Development | Developed (Ready for Review) | Complete 100% transparent pass-through background styling implemented on branch `feature/transparent-terminal-background` (Commit `4dea5b5`). All 6 work items completed across root screen, transcript stream & code fences, prompt inputs, task footer, and dialog modals. Preserves Truecolor `ansi_color=False` syntax highlighting with zero solid background clipping boxes and minimal high-contrast borders `$primary` and `$accent`. Verified with 30/30 unit tests passing (100%) and clean linter. Formalized in [[architecture/adr-009-tui-transparent-pass-through-rendering\|ADR-009]]. Ready for Phase 5: Review handoff. |
| [[rename-plugin-repositories\|Rename Plugin Repositories to Descriptive Action-Oriented Names]] | Phase 6: Merge & Release | Released / Merged | Standardize 5 companion plugin repositories to descriptive action-oriented names (`agentic-prompt-triage-plugin`, `agentic-issue-investigation-plugin`, `agentic-task-monitoring-plugin`, `agentic-topic-researching-plugin`, `agentic-project-documenting-plugin`). Renamed remote GitHub repositories, local directories, package manifests, and Obsidian documentation. Remediated AC-8 dynamic triage detection and purged stale companion references across 5 `bugfix/*` branches (`9c5c19b`, `00b7d03`, `8f46739`, `76f7002`, `ee5bfa0`). All branches merged into `master`, bugfix branches deleted, and all 135/135 unit tests passing workspace-wide. Formalized in [[architecture/adr-004-standardize-plugin-repository-names\|ADR-004]]. |
| [[project-grooming-plugin\|Project Grooming Plugin & Skill (/project-grooming)]] | Phase 6: Merge & Release | Released / Completed | Standalone Antigravity companion plugin (`agentic-project-grooming-plugin`) featuring skill `/project-grooming`. Conducts Matt Pocock `/grill-me` style single-question interrogation across 7 core requirements categories, writes interactive approval artifact `Project_Request_Grooming.md` (`RequestFeedback=True`, sole tool call), and orchestrates scaffolding across `@coding`, `@coding-document`, and `agentic-prompt-triage-plugin`. Formalized in [[architecture/adr-003-project-grooming-plugin\|ADR-003]]. Fast-forward merged into `master` across all repositories (`2bbd11f`, `c6fd560`, `d2e11c5`), feature branches deleted, and all 117/117 unit tests passing across all plugins. |
| [[agent-display-names-and-plugin-renames\|Agent Display Names & Plugin Renames (Monitor & Document)]] | Phase 6: Merge & Release | Released / Completed | Standardize agent display titles to prefix `Coding - ` (setting `@coding-monitor` to `Coding - Monitor Expert`), preserve technical identifiers (`name: coding-*`), rename `agentic-monitoring-plugin` -> `agentic-monitor-plugin` and `agentic-obsidian-plugin` -> `agentic-document-plugin`, update Git remote origins to github.com:gpfister/*, and execute fast-forward merge to `master` across all 8 repositories (101/101 tests passing on `master`). |
| [[standardize-agent-handles\|Standardize Agent Handles]] | Phase 6: Merge & Release | Released / Completed | Standardize all agent handles, aliases, skills, and agent directories to strictly use the `@coding-*` prefix and purge non-coding aliases across all 8 repositories (86/86 tests passing on master). |
