---
plugin_name: "agentic-coding-plugin"
status: "active"
tags:
  - antigravity-plugin
  - component-spec
  - subagents
  - markdown-agents
  - laya-router
version: "0.1.0"
---

# Plugin: agentic-coding-plugin

## Overview
`agentic-coding-plugin` is the primary coding companion plugin within the `agentic` ecosystem. It bundles declarative markdown-defined subagents under [`agentic-coding-plugin/agents/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/), progressive-disclosure skills under [`agentic-coding-plugin/skills/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/), an instruction-driven 6-stage development lifecycle executed via `invoke_subagent`, a standalone cross-repository issue investigator (`@coding-investigate`), a background telemetry monitor (`@monitoring`), a `SessionStart` subagent registration hook, and a non-autoregressive `PreInvocation` **Laya Router** System 1 prompt triage hook.

---

## Components

### 1. Manifest, Environment & Setup (`plugin.json`, `pyproject.toml`, `hooks.json`, `LICENSE`, `CONTRIBUTING.md`)
- **Manifest**: [`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/plugin.json) declaring name `agentic-coding-plugin`, version `0.1.0`, license `MIT`, and `post_install` script ([`scripts/post_install.sh`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/scripts/post_install.sh)).
- **License**: [`LICENSE`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/LICENSE) specifying MIT License (Copyright (c) 2026 Greg PFISTER).
- **Contributing Guidelines**: [`CONTRIBUTING.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/CONTRIBUTING.md) detailing subagent/skill addition workflows, Conventional Commits, testing requirements, and PR review checklists.
- **Dependencies**: [`pyproject.toml`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/pyproject.toml) managed via `uv` (`laya>=0.4.15`, `pydantic>=2.10.0`, `pytest>=8.0.0`).
- **Rules**: [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/rules/AGENTS.md) enforcing core engineering principles (clarity over cleverness, fail fast, single responsibility, preserving comments/docstrings, and clickable `file://` links).
- **Lifecycle Hooks**: [`hooks.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/hooks.json) executed via `uv run`:
  - **`SessionStart`**: Runs [`hooks/register_subagents.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/hooks/register_subagents.py) (`uv run hooks/register_subagents.py`) to discover all markdown agent definitions, register `@coding`, `@coding-investigate`, `@monitoring`, and worker subagents with appropriate tool permissions, and inject orchestration & interactive artifact approval guidance into the session.
  - **`PreInvocation`**: Runs [`hooks/prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/hooks/prompt_triage.py) (`uv run hooks/prompt_triage.py`) using the Laya Router System 1 model for 5-way prompt intent classification and routing.

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
| [`coding-investigate/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-investigate/) | `@coding-investigate` | **Issue Investigator**: Standalone multi-repository root cause investigator producing `issue-assessment.md`, grilling the user when needed, and handing off approved fixes to `@coding`. | `write=True`, `subagent=True`, `mcp=True` |
| [`coding-monitoring/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-monitoring/) | `@coding-monitoring` | **Telemetry Monitor**: Continuous log, Docker, and serial stream (`/dev/tty*`) surveillance with anomaly pause gates. | `write=True`, `subagent=False`, `mcp=True` |
| [`monitoring/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/monitoring/) | `@monitoring` | **Telemetry Monitor (Alias)**: Short alias for `coding-monitoring`. | `write=True`, `subagent=False`, `mcp=True` |

### 3. Progressive-Disclosure Skills (`skills/`)
| Skill | File | Description |
| :--- | :--- | :--- |
| `coding` | [`skills/coding-orchestrator/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/coding-orchestrator/SKILL.md) | Guides full-lifecycle software development tasks through the 6-stage coordinated workflow (`@coding`). |
| `coding-investigate` | [`skills/coding-investigate/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/coding-investigate/SKILL.md) | Guides multi-repository root cause diagnosis, `issue-assessment.md` approval, and remediation handoff (`@coding-investigate`). |
| `coding-monitoring` | [`skills/coding-monitoring/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/coding-monitoring/SKILL.md) | Guides background telemetry, log surveillance, and serial console monitoring (`@monitoring`). |
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

### 2. Standalone Issue Investigation Pipeline (`@coding-investigate`)
```text
Multi-Repo Search ──> Solution Found? ──(No)──> Grill User (ask_question) ──┐
                            │                                                │
                          (Yes) <────────────────────────────────────────────┘
                            ▼
                Write issue-assessment.md (RequestFeedback=True)
                            │
                     [Approved by User]
                            ▼
            Ask Correction Confirmation (Yes / No)
                     │                    │
                   (Yes)                 (No)
                     ▼                    ▼
              Invoke @coding            Stop
```

### 3. Antigravity Interactive Artifact Approval Protocol
- **Mandatory Approval Gates**: `specification.md` (Stage 2), `plan.md` (Stage 3), and `issue-assessment.md` (`@coding-investigate`) require interactive user approval before proceeding.
- **Artifact Metadata**: Written to `<appDataDir>/brain/<conversation-id>/<artifact>.md` with `ArtifactMetadata={"RequestFeedback": True, "UserFacing": True}`.
- **Sole & Final Tool Call Rule**: `write_file(..., RequestFeedback=True)` must be the sole and final tool call of that turn so Antigravity presents the interactive **Approve / Reject** UI modal.

### 4. Laya Router System 1 Prompt Triage (`hooks/prompt_triage.py`)
- **Single-Fire Guard**: Only executes on the initial user prompt (`invocationNum == 0`); skips subsequent tool-step re-invocations (`invocationNum > 0`).
- **Prompt Sanitization**: `clean_user_prompt` strips `<USER_REQUEST>`, `<ADDITIONAL_METADATA>`, XML tags, and trailing concatenated file paths before classification.
- **5-Way Classification** (`"What is this prompt about ?"`):
  1. **`response 1`** (*inquiries involving no changes*): Main agent responds directly (`"Do not make any changes."`).
  2. **`response 2`** (*code change request*): Main agent invokes `@coding` via `invoke_subagent`.
  3. **`response 3`** (*task monitoring request*): Main agent invokes `@monitoring` via `invoke_subagent`.
  4. **`response 4`** (*issue investigation request*): Main agent invokes `@coding-investigate` via `invoke_subagent`.
  5. **`response 5`** (*mix of requests*): Main agent rejects the mixed prompt and asks the user to separate their requests.
- **Confidence Threshold** ($\ge 0.40$):
  - If confidence $\ge 0.40$, the corresponding directive is injected immediately via `systemMessage`.
  - If confidence $< 0.40$ (ambiguous), the main agent is instructed to present the 5 options to the user via `ask_question`.
  - Includes a deterministic keyword heuristic fallback when Laya Router is offline or disabled.

---

## Testing
Automated tests in [`agentic-coding-plugin/tests/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/tests/) validate manifest integrity, agent frontmatter & registration, and prompt triage routing:
- [`tests/test_plugin.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/tests/test_plugin.py)
- [`tests/test_register_subagents.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/tests/test_register_subagents.py)
- [`tests/test_prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/tests/test_prompt_triage.py)
