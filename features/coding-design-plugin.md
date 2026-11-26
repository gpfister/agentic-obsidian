---
title: "Coding Design Plugin (@coding-design) with Official PenPot MCP Integration"
date: "2026-11-21"
status: "Merged / Completed"
phase: "Phase 7: Merge & Release Complete (Post-Merge Synchronized)"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/2b36f2db-6da8-42bd-90a7-3c40cc5d8059/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/590edead-39dc-4a1e-af1d-821b8249862f/plan.md"
review_path: "/Users/gpfister/.gemini/antigravity/brain/dc440826-17b4-4ec9-9f5c-c81e97c68e17/review-coding-design-plugin.md"
adrs:
  - "[[architecture/adr-013-coding-design-plugin-and-penpot-mcp|ADR-013: Coding Design Plugin (@coding-design) with Official PenPot MCP Integration]]"
tags:
  - feature
  - agentic-design-plugin
  - coding-design
  - penpot
  - mcp
  - dual-mode-relay
  - exclusive-code-change-authority
  - merged
  - released
affected_plugins:
  - agentic-design-plugin
  - agentic
  - agentic-coding-plugin
  - agentic-prompt-triage-plugin
  - agentic-project-documenting-plugin
aliases:
  - coding-design-plugin
---

# Feature: `agentic-design-plugin` (`@coding-design` with PenPot MCP)

> [!NOTE] Lifecycle Status: Phase 7 Complete — Merged & Post-Merge Synchronized
> - **Approved Specification**: [specification.md](file:///Users/gpfister/.gemini/antigravity/brain/2b36f2db-6da8-42bd-90a7-3c40cc5d8059/specification.md)
> - **Approved Plan**: [plan.md](file:///Users/gpfister/.gemini/antigravity/brain/590edead-39dc-4a1e-af1d-821b8249862f/plan.md)
> - **QA Review**: [review-coding-design-plugin.md](file:///Users/gpfister/.gemini/antigravity/brain/dc440826-17b4-4ec9-9f5c-c81e97c68e17/review-coding-design-plugin.md)
> - **Decision Record**: [[architecture/adr-013-coding-design-plugin-and-penpot-mcp|ADR-013]]
> - **Plugin Spec**: [[plugins/agentic-design-plugin|agentic-design-plugin]]
> - **Git Status & Changelog**: [[git-status|Workspace Git Status]] / [[changelog|Workspace Release Changelog]]

---

## 1. Summary

Introduces **`agentic-design-plugin`**, a standalone Antigravity plugin repository hosting the **`@coding-design`** Dual-Mode subagent with official **PenPot MCP** integration (`${PENPOT_MCP_URL:-http://localhost:4401/mcp}`). `@coding-design` provides bidirectional UI/UX design authoring and inspection in PenPot, synthesizes `<appDataDir>/brain/<conversation-id>/design-spec.md`, and integrates across the `agentic` workspace while strictly respecting Exclusive Code-Change Authority ([[architecture/adr-012-exclusive-code-change-authority|ADR-012]]) and Dual-Mode Relay Protocols ([[architecture/adr-005-question-relay-protocol|ADR-005]], [[architecture/adr-006-artifact-approval-relay-protocol|ADR-006]]).

---

## 2. Affected Repositories & Merge Ledger

| # | Repository | Merge / PR | `main` Commit | Summary of Changes | Verification |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | `agentic-design-plugin` | Local merge (`feature/coding-design-plugin` -> `main`) | `9b434b4` | New standalone plugin (`plugin.json`, `mcp_config.json`, `rules/AGENTS.md`, `agents/coding-design/agent.md`, `skills/coding-design/SKILL.md`, `pyproject.toml`, `uv.lock`, `tests/test_plugin.py`, `README.md`, `LICENSE`, `.gitignore`). | `7 passed, 0 failed` (`uv run pytest -v`) |
| 2 | `agentic` (Root) | [PR #4](https://github.com/gpfister/agentic/pull/4) (`MERGED`) | `f234f6a` | Registered `agentic-design-plugin` in `Makefile` (`REPOS`), `.gitignore`, and `AGENTS.md` (layout, catalog, interoperability matrix). | `make status` exited 0 |
| 3 | `agentic-coding-plugin` | [PR #6](https://github.com/gpfister/agentic-coding-plugin/pull/6) (`MERGED`) | `2348713` | Added dynamic detection of `agentic-design-plugin` and optional Phase 2/3 `@coding-design` delegation in `rules/AGENTS.md` and `skills/coding-orchestrator/SKILL.md`. | `26 passed, 0 failed` (`uv run pytest -v`) |
| 4 | `agentic-prompt-triage-plugin` | [PR #1](https://github.com/gpfister/agentic-prompt-triage-plugin/pull/1) (`MERGED`) | `bca48d6` | Added `agentic-design-plugin` detection and `design` intent routing to `@coding-design` in `hooks/prompt_triage.py`, `rules/AGENTS.md`, and `tests/test_prompt_triage.py`. | `87 passed, 0 failed` (`uv run pytest -v`) |
| 5 | `agentic-project-documenting-plugin` | [PR #2](https://github.com/gpfister/agentic-project-documenting-plugin/pull/2) (`MERGED`) | `c2d3b19` | Documented `agentic-obsidian/designs/<design-slug>.md` and `agentic-obsidian/designs/index.md` archival from `@coding-design` in `rules/AGENTS.md`, `skills/coding-document/SKILL.md`, and `tests/test_plugin.py`. | `11 passed, 0 failed` (`uv run pytest -v`) |

---

## 3. Architectural & Safety Guarantees

- **Exclusive Code-Change Authority ([[architecture/adr-012-exclusive-code-change-authority|ADR-012]])**: `edit_file` and `run_command` are omitted from `@coding-design`; `write_file` is restricted to `<appDataDir>/brain/<conversation-id>/design-spec.md`; UI code/CSS/token implementation is delegated to `@coding-develop` under `@coding`.
- **Dual-Mode Relay Protocols ([[architecture/adr-005-question-relay-protocol|ADR-005]], [[architecture/adr-006-artifact-approval-relay-protocol|ADR-006]])**: Supports both Subagent Mode (`[QUESTION_RELAY_REQUEST]` / `[ARTIFACT_APPROVAL_REQUEST]` with `stage: "DESIGN"`) and Standalone Mode (direct `ask_question` and `write_file(RequestFeedback=True)` as sole final tool call, plus optional `@coding` handoff).
- **Portable PenPot MCP Configuration**: `mcp_config.json` uses `${PENPOT_MCP_URL:-http://localhost:4401/mcp}` with zero hardcoded machine-local paths and fail-fast error reporting when the MCP server is unreachable.
- **Local Deployment**: Not required across all 5 repositories (no Dockerfiles, Compose files, or local deployment scripts).
