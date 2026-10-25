---
plugin_name: "gp-coding-plugin"
status: "active"
tags:
  - antigravity-plugin
  - component-spec
version: "0.1.0"
---

# Plugin: gp-coding-plugin

## Overview
`gp-coding-plugin` is the primary coding companion plugin within the `agentic` ecosystem. It bundles custom coding standards, reusable developer skills, environment verification hooks, and optional MCP tools to assist with coding and architecture tasks.

## Components

### 1. Manifest
- File: `plugin.json`
- Defines plugin name `gp-coding-plugin`, version `0.1.0`, and default status.

### 2. Rules (`rules/AGENTS.md`)
- Enforces modular code architecture, rigorous unit testing, and structured logging.
- Requires documentation updates and clean Git commits.

### 3. Skills (`skills/`)
- `clean-code-reviewer`: Systematic automated code review and refactoring assistant.

### 4. Lifecycle Hooks (`hooks/` & `hooks.json`)
- Native lifecycle hook support for pre-execution verification and post-action reporting.

### 5. MCP Tool Integrations (`mcp_config.json`)
- Ready for external tool and server configurations.
