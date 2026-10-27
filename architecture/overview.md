---
title: "Agentic Workspace Architecture Overview"
date: "2026-10-26"
tags:
  - architecture
  - multi-repo
---

# Agentic Workspace Architecture Overview

## Motivation & Principles
The **Agentic** workspace is designed to author, test, and maintain a federation of **Google Antigravity** plugins while maintaining an indexed Obsidian knowledge base.

### Key Tenets
1. **Multi-Repo Federation**: Each plugin is self-contained with its own Git history, `uv`-managed virtual environment, tests, and configuration. The parent directory acts as an orchestrator.
2. **Declarative Multi-Agent Orchestration**: Plugins define specialized subagents as markdown files (`agents/*.md`) registered at `SessionStart` and routed at `PreInvocation` via the **Laya Router** System 1 classifier.
3. **Interactive Artifact Gates**: Key lifecycle milestones (`specification.md`, `plan.md`, `issue-assessment.md`) use Antigravity's interactive `RequestFeedback=True` approval modals before advancing execution.
4. **Knowledge Continuity**: Agent interactions, reasoning steps, tool usage, and generated artifacts from both `~/.gemini/antigravity/brain` and `~/.gemini/antigravity-cli/brain` are exported to `agentic-obsidian/sessions/` using structured callouts and frontmatter metadata.
5. **Progressive Disclosure**: Skills and rules adhere strictly to Antigravity guidelines, minimizing context overhead while providing targeted guidance.
6. **Standard Antigravity Packaging**: All plugins comply with native manifest and lifecycle specs (`plugin.json`, `agents/`, `rules/AGENTS.md`, `skills/`, `hooks.json`, `mcp_config.json`).

---

## Repository Map

| Repository / Path | Purpose |
| :--- | :--- |
| [`AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/AGENTS.md) | Root workspace coordination instructions and operational rules. |
| [`scripts/sync_sessions.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/scripts/sync_sessions.py) | Exports Antigravity session transcripts and artifacts into `agentic-obsidian/sessions/`. |
| [`scripts/worktree.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/scripts/worktree.py) | Manages isolated Git worktrees across root and sub-repositories for parallel sessions. |
| [`agentic-coding-plugin/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/) | Primary coding companion plugin (`@coding`, `@coding-investigate`, `@monitoring`, Laya Router triage). |
| [`agentic-obsidian/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/) | Centralized Obsidian vault for plugin specs, architecture docs, and session logs. |
