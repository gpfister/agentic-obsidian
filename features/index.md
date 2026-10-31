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
| [[agent-display-names-and-plugin-renames\|Agent Display Names & Plugin Renames (Monitor & Document)]] | Phase 6: Merge & Release | Released / Completed | Standardize agent display titles to prefix `Coding - ` (setting `@coding-monitor` to `Coding - Monitor Expert`), preserve technical identifiers (`name: coding-*`), rename `agentic-monitoring-plugin` -> `agentic-monitor-plugin` and `agentic-obsidian-plugin` -> `agentic-document-plugin`, update Git remote origins to github.com:gpfister/*, and execute fast-forward merge to `master` across all 8 repositories (101/101 tests passing on `master`). |
| [[standardize-agent-handles\|Standardize Agent Handles]] | Phase 6: Merge & Release | Released / Completed | Standardize all agent handles, aliases, skills, and agent directories to strictly use the `@coding-*` prefix and purge non-coding aliases across all 8 repositories (86/86 tests passing on master). |
