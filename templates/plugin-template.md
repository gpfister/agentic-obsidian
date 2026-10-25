---
plugin_name: "{{plugin_name}}"
status: "active"
tags:
  - antigravity-plugin
  - component-spec
version: "0.1.0"
---

# Plugin: {{plugin_name}}

## Overview
Brief description of the plugin's capability and purpose within the Antigravity ecosystem.

## Components

### 1. Rules (`rules/AGENTS.md`)
- Guidelines and system prompts injected when the plugin is enabled.

### 2. Skills (`skills/`)
| Skill Name | Description | Invocation |
| :--- | :--- | :--- |
| `example-skill` | What the skill does | Progressive disclosure / explicit |

### 3. MCP Servers (`mcp_config.json`)
- Exposed tools and services.

### 4. Lifecycle Hooks (`hooks/` / `hooks.json`)
- Hooks triggered during agent turns.

## Installation & Usage
```bash
# Enable in Antigravity
agy plugin enable {{plugin_name}}
```
