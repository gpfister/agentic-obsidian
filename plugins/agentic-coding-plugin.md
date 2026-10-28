---
plugin_name: "agentic-coding-plugin"
status: "active"
tags:
  - antigravity-plugin
  - component-spec
  - subagents
  - markdown-agents
  - coding-workflow
version: "0.1.0"
---

# Plugin: agentic-coding-plugin

## Overview
`agentic-coding-plugin` is the primary coding companion plugin within the `agentic` ecosystem. It bundles declarative markdown-defined subagents under [`agentic-coding-plugin/agents/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/), progressive-disclosure skills under [`agentic-coding-plugin/skills/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/), an instruction-driven 6-stage software development lifecycle executed via `invoke_subagent`, and clean code engineering standards for Google Antigravity.

---

## Components

### 1. Manifest, Environment & Setup (`plugin.json`, `pyproject.toml`, `hooks.json`, `LICENSE`, `CONTRIBUTING.md`)
- **Manifest**: [`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/plugin.json) declaring name `agentic-coding-plugin`, version `0.1.0`, license `MIT`, and `post_install` script ([`scripts/post_install.sh`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/scripts/post_install.sh)).
- **License**: [`LICENSE`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/LICENSE) specifying MIT License (Copyright (c) 2026 Greg PFISTER).
- **Contributing Guidelines**: [`CONTRIBUTING.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/CONTRIBUTING.md) detailing subagent/skill addition workflows, Conventional Commits, testing requirements, and PR review checklists.
- **Dependencies**: [`pyproject.toml`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/pyproject.toml) managed via `uv` (`pytest>=8.0.0`).
- **Rules**: [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/rules/AGENTS.md) enforcing core engineering principles (clarity over cleverness, fail fast, single responsibility, preserving comments/docstrings, and clickable `file://` links).
- **Lifecycle Hooks**: [`hooks.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/hooks.json):
  - **`SessionStart`**: Deactivated (`enabled: false`) to avoid race conditions with native markdown discovery.

### 2. Declarative Markdown Subagents (`agents/`)
Subagent instructions are authored as modular markdown specifications within dedicated directories (`agents/<agent>/agent.md`), described and orchestrated by the master [`agents/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/agent.md) specification:

| Agent Directory / File | Handle / Name | Role & Capabilities | Tool Permissions |
| :--- | :--- | :--- | :--- |
| [`agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/agent.md) | `@agent` | **Subagents & Workflow Specification**: Architecture, subagent registry, tool privilege model, and coding workflow guide. | `write=True`, `subagent=True`, `mcp=True` |
| [`coding/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding/) | `@coding` | **Coding Workflow Orchestrator**: Coordinates the 6-stage software development lifecycle across repositories using `invoke_subagent`. | `write=True`, `subagent=True`, `mcp=True` |
| [`coding-clarify/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-clarify/) | `@coding-clarify` | **Requirement Clarifier**: Matt Pocock `/grill-me` style requirement interrogation asking one question at a time via `ask_question`. | `write=True`, `subagent=False`, `mcp=True` |
| [`coding-specify/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-specify/) | `@coding-specify` | **Specification Architect**: Synthesizes clarified requirements into `specification.md` and manages the interactive approval gate. | `write=True`, `subagent=False`, `mcp=True` |
| [`coding-plan/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-plan/) | `@coding-plan` | **Architecture Planner**: Deconstructs approved specifications into per-repository tasks and branch names (`feature/*`, `task/*`, `bugfix/*`) in `plan.md`. | `write=True`, `subagent=False`, `mcp=True` |
| [`coding-develop/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-develop/) | `@coding-develop` | **Software Engineer**: Isolated single-repository implementation, unit tests, linting, documentation, and CI/CD. | `write=True`, `subagent=False`, `mcp=True` |
| [`coding-review/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-review/) | `@coding-review` | **Quality Assurance Lead**: Cross-repository verification against `specification.md` and discrepancy routing. | `write=True`, `subagent=False`, `mcp=True` |
| [`coding-merge/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-merge/) | `@coding-merge` | **Release Engineer**: Executes local git merge (when no remote exists) or opens a GitHub Pull Request via `gh pr create`. | `write=True`, `subagent=False`, `mcp=True` |

### 3. Progressive-Disclosure Skills (`skills/`)
| Skill | File | Description |
| :--- | :--- | :--- |
| `coding` | [`skills/coding-orchestrator/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/coding-orchestrator/SKILL.md) | Guides full-lifecycle software development tasks through the 6-stage coordinated workflow (`@coding`). |
| `clean-code-reviewer` | [`skills/clean-code-reviewer/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/clean-code-reviewer/SKILL.md) | Systematic code quality, readability, error handling, and architectural boundary reviewer. |

---

## Workflows & Protocols

### 1. 6-Stage Coding Lifecycle Pipeline (`@coding`)
Phase transitions are orchestrated via `invoke_subagent`:
```text
coding-clarify ──> coding-specify ──> coding-plan ──> coding-develop (parallel per repo)
                          ▲               ▲                      │
                          │ (reject/fb)   │ (reject/fb)          ▼
                          └───────────────┴────────────── coding-review
                                                                 │ (approved)
                                                                 ▼
                                                           coding-merge
```

### 2. Antigravity Interactive Artifact Approval Protocol
- **Mandatory Approval Gates**: `specification.md` (Stage 2) and `plan.md` (Stage 3) require interactive user approval before proceeding.
- **Artifact Metadata**: Written to `<appDataDir>/brain/<conversation-id>/<artifact>.md` with `ArtifactMetadata={"RequestFeedback": True, "UserFacing": True}`.
- **Sole & Final Tool Call Rule**: `write_file(..., RequestFeedback=True)` must be the sole and final tool call of that turn so Antigravity presents the interactive **Approve / Reject** UI modal.

---

## Testing
Automated tests in [`agentic-coding-plugin/tests/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/tests/) validate manifest integrity, agent frontmatter & registration, and lifecycle rules:
- [`tests/test_plugin.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/tests/test_plugin.py)
- [`tests/test_register_subagents.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/tests/test_register_subagents.py)
