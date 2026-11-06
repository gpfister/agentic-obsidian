---
plugin_name: "agentic-issue-investigation-plugin"
status: "active"
tags:
  - antigravity-plugin
  - component-spec
  - investigation
  - root-cause-analysis
  - subagents
version: "0.1.0"
---

# Plugin: agentic-issue-investigation-plugin

## Overview
`agentic-issue-investigation-plugin` is the autonomous multi-repository issue investigation and root-cause analysis companion plugin within the `agentic` ecosystem. It bundles a dedicated declarative markdown subagent ([`@coding-investigate`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/agents/coding-investigate/agent.md)), a progressive-disclosure investigation skill ([`skills/coding-investigate/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/skills/coding-investigate/SKILL.md)), user clarification protocols, an interactive issue assessment approval gate (`issue-assessment.md`), and a remediation confirmation handoff to `@coding`.

---

## Components

### 1. Manifest, Metadata & Rules
- **Manifest**: [`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/plugin.json) declaring name `agentic-issue-investigation-plugin`, version `0.1.0`, license `MIT`.
- **Environment**: Managed via `uv` with `pyproject.toml` dependencies (`pydantic>=2.10.0`, `pytest>=8.0.0`).
- **Rules**: [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/rules/AGENTS.md) enforcing evidence-based diagnosis, multi-repository search discipline, read-only diagnostic safety, and interactive artifact approval protocols.

### 2. Declarative Markdown Subagents (`agents/`)
Subagents are organized in dedicated directories with frontmatter metadata:

| Agent Directory / File | Handle / Name | Role & Capabilities | Tool Permissions |
| :--- | :--- | :--- | :--- |
| [`agents/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/agents/agent.md) | `coding-investigate-catalog` | **Investigation Subagents Overview**: Investigation architecture, subagent catalog, and remediation protocols. | `write=False`, `subagent=False`, `mcp=False` |
| [`agents/coding-investigate/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/agents/coding-investigate/agent.md) | `@coding-investigate` | **Issue Investigator**: Autonomous multi-repository root cause investigator producing `issue-assessment.md` artifacts, grilling users when needed, researching external issues, handing off approved fixes to `@coding` (if installed), and archiving to `@coding-document` (if installed). | `write=True`, `subagent=True`, `mcp=True` |

### 3. Progressive-Disclosure Skills (`skills/`)
| Skill Name | Path | Description | Invocation |
| :--- | :--- | :--- | :--- |
| `coding-investigate` | [`skills/coding-investigate/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/skills/coding-investigate/SKILL.md) | Runbooks and diagnostic procedures for tracing cross-repository bugs, user grilling, and issue assessment authoring. | Progressive disclosure / `@coding-investigate` prompt |

---

## Investigation & Remediation Workflow

```text
Multi-Repo Search ──> Solution Found? ──(No)──> Dual-Mode Grilling ([QUESTION_RELAY_REQUEST] / ask_question) ──┐
                            │                                                                                  │
                          (Yes) <──────────────────────────────────────────────────────────────────────────────┘
                            ▼
        Author issue-assessment.md (Dual-Mode: [ARTIFACT_APPROVAL_REQUEST] or write_file(RequestFeedback=True))
                            │
                     [Approved by User]
                            ▼
         Dual-Mode Remediation Confirmation Gate (Yes / No)
                     │                    │
                   (Yes)                 (No)
                     ▼                    ▼
   Invoke @coding (Standardized Template) Stop
```

### Dual-Mode Interactive Protocols
1. **Artifact Approval Relay (Dual-Mode)**:
   - **Subagent Mode (`caller_id` present)**: Writes `issue-assessment.md` to disk directly without feedback metadata and dispatches `[ARTIFACT_APPROVAL_REQUEST]` to `caller_id` via `send_message` to prevent 60-second host runtime timeouts.
   - **Standalone Mode (no `caller_id`)**: Authors `issue-assessment.md` directly via `write_file(RequestFeedback=True)` as the sole and final tool call of that turn.
2. **Question Relay Protocol (Dual-Mode)**:
   - **Subagent Mode**: Relays single-question inquiries and remediation confirmation gates via `[QUESTION_RELAY_REQUEST]` through `send_message`.
   - **Standalone Mode**: Prompts the user directly via `ask_question`.
3. **Standardized Upstream Invocation Template**:
   When handing off an approved remediation plan to `@coding`, `@coding-investigate` formats the invocation prompt using the mandatory template:
   ```text
   Orchestrate the 7-stage software development lifecycle to implement the following feature / correction based on the approved assessment: <details/path>. Do not implement directly; delegate all phases through worker subagents.
   ```
   This enforces the Pre-Clarified Fast-Path and activates `@coding`'s hard Anti-Direct-Coding Guardrails.

---

## Testing
Automated tests in [`agentic-issue-investigation-plugin/tests/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/tests/) validate manifest integrity, agent registration, dual-mode relay protocols, and standardized invocation prompts:
- [`tests/test_plugin.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/tests/test_plugin.py) (14/14 tests passing, 100% pass rate)

---

## Installation & Usage

```bash
# Enable in Antigravity session:
agy plugin enable agentic-issue-investigation-plugin

# Invoke investigation directly:
@coding-investigate Investigate why payments time out in checkout
```
