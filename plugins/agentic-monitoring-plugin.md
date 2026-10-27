---
plugin_name: "agentic-monitoring-plugin"
status: "active"
tags:
  - antigravity-plugin
  - component-spec
  - telemetry
  - stream-surveillance
  - subagents
version: "0.1.0"
---

# Plugin: agentic-monitoring-plugin

## Overview
`agentic-monitoring-plugin` is the telemetry, log observation, and stream surveillance companion plugin within the `agentic` ecosystem. It bundles a dedicated declarative markdown subagent ([`@monitoring`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-monitoring-plugin/agents/monitoring/agent.md)), a progressive-disclosure monitoring skill ([`skills/monitoring/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-monitoring-plugin/skills/monitoring/SKILL.md)), and an interactive pause gate protocol that halts log cascades upon anomaly detection to prompt the user with remediation choices.

---

## Components

### 1. Manifest, Metadata & Rules
- **Manifest**: [`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-monitoring-plugin/plugin.json) declaring name `agentic-monitoring-plugin`, version `0.1.0`, license `MIT`.
- **Environment**: Managed via `uv` with `pyproject.toml` dependencies (`pydantic>=2.10.0`, `pytest>=8.0.0`).
- **Rules**: [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-monitoring-plugin/rules/AGENTS.md) enforcing non-destructive stream observation, bounded log buffers, and interactive anomaly pause gate protocols.

### 2. Declarative Markdown Subagents (`agents/`)
Subagents are organized in dedicated directories with frontmatter metadata:

| Agent Directory / File | Handle / Name | Role & Capabilities | Tool Permissions |
| :--- | :--- | :--- | :--- |
| [`agents/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-monitoring-plugin/agents/agent.md) | `@agent` | **Monitoring Subagents Overview**: Telemetry architecture, subagent catalog, and anomaly pause gate protocols. | `write=True`, `subagent=False`, `mcp=True` |
| [`agents/monitoring/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-monitoring-plugin/agents/monitoring/agent.md) | `@monitoring` | **Telemetry Monitor**: Continuous surveillance of application logs, Docker stdout, and serial ports (`/dev/tty*`) with anomaly pause gates. | `write=True`, `subagent=False`, `mcp=True` |

### 3. Progressive-Disclosure Skills (`skills/`)
| Skill Name | Path | Description | Invocation |
| :--- | :--- | :--- | :--- |
| `monitoring` | [`skills/monitoring/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-monitoring-plugin/skills/monitoring/SKILL.md) | Runbooks and procedures for streaming logs, Docker containers, and serial ports with automatic anomaly detection. | Progressive disclosure / `@monitoring` prompt |

---

## Telemetry & Anomaly Pause Gate Protocol

```text
Stream Target (Log / Container / Serial)
         │
         ▼
Continuous Surveillance Scan (Chunk by chunk)
         │
    Anomaly Detected?
   ├── (No) ──> Continue streaming
   └── (Yes) ──> Immediately PAUSE surveillance
                      │
                      ▼
            Synthesize MonitoringAlert JSON
                      │
                      ▼
         Prompt User via ask_question
         - Inspect code around stack trace
         - Restart process / container
         - Ignore and continue
         - Abort monitoring session
```

---

## Installation & Usage

```bash
# Enable in Antigravity session:
agy plugin enable agentic-monitoring-plugin

# Invoke monitoring directly:
@monitoring watch server logs at /var/log/app.log and alert on 5xx errors
```
