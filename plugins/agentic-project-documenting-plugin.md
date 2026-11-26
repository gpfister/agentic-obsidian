---
plugin_name: "agentic-project-documenting-plugin"
status: "active"
tags:
  - antigravity-plugin
  - component-spec
  - documentation
  - obsidian-vault
  - knowledge-base
  - git-status
  - sessions
  - subagents
version: "0.1.0"
---

# Plugin: agentic-project-documenting-plugin

## Overview
`agentic-project-documenting-plugin` is the centralized documentation and Obsidian vault knowledge curator companion plugin within the `agentic` workspace. It establishes the single source of truth across projects, architecture designs (ADRs), feature specifications, UI/UX design specifications (`agentic-obsidian/designs/`), Antigravity session trajectories, generated artifacts, code changes, and git repository statuses.

It introduces the autonomous subagent [`@coding-document`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/agents/coding-document/agent.md), addressable independently by users and programmatically invoked by other pipeline subagents (`@coding`, `@coding-design`, `@coding-investigate`, `@coding-research`, `@coding-monitor`, `@coding-triage`).

---

## Components

### 1. Manifest, Metadata & Rules
- **Manifest**: [`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/plugin.json) declaring name `agentic-project-documenting-plugin`, version `0.1.0`, license `MIT`.
- **Environment**: Managed via `uv` with `pyproject.toml` dependencies (`pydantic>=2.10.0`, `pytest>=8.0.0`).
- **Rules**: [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/rules/AGENTS.md) enforcing:
  - **Comprehensive Obsidian Vault Coverage**: Projects, features, designs (`designs/`), architecture, sessions, artifacts, git statuses.
  - **Clickable Links & Wikilinks**: Combining `[[Note Name]]` with `file://` scheme links.
  - **Permissions & Safety**: Explicit read, Obsidian-scoped `write_file` (`*-obsidian/` vault documentation and session artifacts only; `edit_file` removed), git inspection, and subagent communication tools while strictly forbidding edits to application logic or code repositories (repo-local docs like `README.md` are delegated to `@coding-develop`).
  - **Cross-Plugin Coordination**: Standardized hooks for pre-merge documentation tidy-ups, post-merge git updates, and deliverable archival (including `@coding-design` `design-spec.md` archival into `agentic-obsidian/designs/<design-slug>.md` and `agentic-obsidian/designs/index.md`).

### 2. Declarative Markdown Subagents (`agents/`)
| Agent Directory / File | Handle / Name | Role & Capabilities | Tool Permissions |
| :--- | :--- | :--- | :--- |
| [`agents/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/agents/agent.md) | `coding-obsidian-catalog` | **Coding - Obsidian Vault Documentation Subagents & Architecture**: Subagent registry, tool configurations, and operating models. | `write=False`, `subagent=False`, `mcp=False` |
| [`agents/coding-document/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/agents/coding-document/agent.md) | `@coding-document` (Alias: `coding-document`) | **Coding - Autonomous Documentation Specialist**: Documents projects, features, designs, architecture, sessions, artifacts, code changes, and git status strictly in `*-obsidian/` vaults and session artifacts (`edit_file` removed; repo-local docs delegated to `@coding-develop`). | `read_file`, `write_file` (`*-obsidian/` & artifacts only), `list_dir`, `run_command` (read-only git & `sync_sessions.py`), `ask_question`, `send_message` |

### 3. Progressive-Disclosure Skills (`skills/`)
| Skill Name | Path | Description | Invocation |
| :--- | :--- | :--- | :--- |
| `coding-document` | [`skills/coding-document/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/skills/coding-document/SKILL.md) | Runbooks and procedures for project documentation, architecture ADRs, design specification archival (`designs/`), session and artifact archival, git status auditing, and multi-agent coordination. | Progressive disclosure / `@coding-document` invocation |

---

## Multi-Plugin Integration Architecture

```text
@coding (after each phase)       ──> @coding-document (maintain specs, plans, progress, ADRs, git status)
                                           │
@coding-design (approved spec)   ──> @coding-document (archive design-spec.md & tokens in designs/)
                                           │
@coding-investigate (concluded)  ──> @coding-document (archive investigation notes)
                                           │
@coding-research (concluded)     ──> @coding-document (record research, create/update topic)
                                           │
@coding-monitor (concluded)      ──> @coding-document (record telemetry & alerts)
                                           │
@triage                 ──(doc-request)─> @coding-document (direct user request dispatch)
```
