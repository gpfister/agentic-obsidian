---
plugin_name: "agentic-investigate-plugin"
status: "active"
tags:
  - antigravity-plugin
  - component-spec
  - investigation
  - root-cause-analysis
  - subagents
version: "0.1.0"
---

# Plugin: agentic-investigate-plugin

## Overview
`agentic-investigate-plugin` is the autonomous multi-repository issue investigation and root-cause analysis companion plugin within the `agentic` ecosystem. It bundles a dedicated declarative markdown subagent ([`@investigate`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-investigate-plugin/agents/investigate/agent.md)), a progressive-disclosure investigation skill ([`skills/investigate/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-investigate-plugin/skills/investigate/SKILL.md)), user clarification protocols, an interactive issue assessment approval gate (`issue-assessment.md`), and a remediation confirmation handoff to `@coding`.

---

## Components

### 1. Manifest, Metadata & Rules
- **Manifest**: [`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-investigate-plugin/plugin.json) declaring name `agentic-investigate-plugin`, version `0.1.0`, license `MIT`.
- **Environment**: Managed via `uv` with `pyproject.toml` dependencies (`pydantic>=2.10.0`, `pytest>=8.0.0`).
- **Rules**: [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-investigate-plugin/rules/AGENTS.md) enforcing evidence-based diagnosis, multi-repository search discipline, read-only diagnostic safety, and interactive artifact approval protocols.

### 2. Declarative Markdown Subagents (`agents/`)
Subagents are organized in dedicated directories with frontmatter metadata:

| Agent Directory / File | Handle / Name | Role & Capabilities | Tool Permissions |
| :--- | :--- | :--- | :--- |
| [`agents/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-investigate-plugin/agents/agent.md) | `@agent` | **Investigation Subagents Overview**: Investigation architecture, subagent catalog, and remediation protocols. | `write=True`, `subagent=True`, `mcp=True` |
| [`agents/investigate/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-investigate-plugin/agents/investigate/agent.md) | `@investigate` | **Issue Investigator**: Autonomous multi-repository root cause investigator producing `issue-assessment.md` artifacts, grilling users when needed, and handing off approved fixes to `@coding`. | `write=True`, `subagent=True`, `mcp=True` |

### 3. Progressive-Disclosure Skills (`skills/`)
| Skill Name | Path | Description | Invocation |
| :--- | :--- | :--- | :--- |
| `investigate` | [`skills/investigate/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-investigate-plugin/skills/investigate/SKILL.md) | Runbooks and diagnostic procedures for tracing cross-repository bugs, user grilling, and issue assessment authoring. | Progressive disclosure / `@investigate` prompt |

---

## Investigation & Remediation Workflow

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

### Interactive Artifact Approval Protocol
1. **Artifact Location**: `<appDataDir>/brain/<conversation-id>/issue-assessment.md`.
2. **Metadata**: `ArtifactMetadata={"RequestFeedback": True, "UserFacing": True}`.
3. **Sole & Final Tool Call Rule**: `write_file(..., RequestFeedback=True)` MUST be the sole and final tool call of that turn so Antigravity renders the interactive **Approve / Reject** buttons.
4. **Correction Confirmation Gate**: Once approved, prompts the user via `ask_question` whether to start the correction. If confirmed, invokes `@coding` (`invoke_subagent`).

---

## Installation & Usage

```bash
# Enable in Antigravity session:
agy plugin enable agentic-investigate-plugin

# Invoke investigation directly:
@investigate Investigate why payments time out in checkout
```
