---
plugin_name: "agentic-obsidian-plugin"
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

# Plugin: agentic-obsidian-plugin

## Overview
`agentic-obsidian-plugin` is the centralized documentation and Obsidian vault knowledge curator companion plugin within the `agentic` workspace. It establishes the single source of truth across projects, architecture designs (ADRs), feature specifications, Antigravity session trajectories, generated artifacts, code changes, and git repository statuses.

It introduces the autonomous subagent [`@coding-document`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian-plugin/agents/coding-document/agent.md), addressable independently by users and programmatically invoked by other pipeline subagents (`@coding`, `@investigate`, `@research`, `@monitoring`, `@triage`).

---

## Components

### 1. Manifest, Metadata & Rules
- **Manifest**: [`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian-plugin/plugin.json) declaring name `agentic-obsidian-plugin`, version `0.1.0`, license `MIT`.
- **Environment**: Managed via `uv` with `pyproject.toml` dependencies (`pydantic>=2.10.0`, `pytest>=8.0.0`).
- **Rules**: [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian-plugin/rules/AGENTS.md) enforcing:
  - **Comprehensive Obsidian Vault Coverage**: Projects, features, architecture, sessions, artifacts, git statuses.
  - **Clickable Links & Wikilinks**: Combining `[[Note Name]]` with `file://` scheme links.
  - **Permissions & Safety**: Explicit read, write, edit, git inspection, and subagent communication tools while strictly forbidding unauthorized edits to application logic.
  - **Cross-Plugin Coordination**: Standardized hooks for pre-merge documentation tidy-ups, post-merge git updates, and deliverable archival.

### 2. Declarative Markdown Subagents (`agents/`)
| Agent Directory / File | Handle / Name | Role & Capabilities | Tool Permissions |
| :--- | :--- | :--- | :--- |
| [`agents/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian-plugin/agents/agent.md) | `obsidian-catalog` | **Documentation Subagent Catalog**: Architecture, subagent registry, tool configurations, and operating models. | `write=False`, `subagent=False`, `mcp=False` |
| [`agents/coding-document/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian-plugin/agents/coding-document/agent.md) | `@coding-document` (Aliases: `coding-document`, `document`) | **Autonomous Documentation Specialist**: Documents projects, features, architecture, sessions, artifacts, code changes, and git status in Obsidian vaults. | `read_file`, `write_file`, `edit_file`, `list_dir`, `run_command`, `ask_question`, `send_message` |

### 3. Progressive-Disclosure Skills (`skills/`)
| Skill Name | Path | Description | Invocation |
| :--- | :--- | :--- | :--- |
| `document` | [`skills/document/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian-plugin/skills/document/SKILL.md) | Runbooks and procedures for project documentation, architecture ADRs, session and artifact archival, git status auditing, and multi-agent coordination. | Progressive disclosure / `@coding-document` invocation |

---

## Multi-Plugin Integration Architecture

```text
@coding (develop/review) ──(pre-merge)──> @coding-document (tidy docs, specs & ADRs)
                                                 │
@coding (release/merge)  ──(post-merge)─> @coding-document (update git status & changelog)
                                                 │
@investigate             ──(concluded)──> @coding-document (archive investigation notes)
                                                 │
@research                ──(concluded)──> @coding-document (archive research brief & citations)
                                                 │
@monitoring              ──(concluded)──> @coding-document (record telemetry & alerts)
                                                 │
@triage                  ──(doc-request)─> @coding-document (direct user request dispatch)
```
