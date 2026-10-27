---
title: "Antigravity Plugin Standards & Structure"
date: "2026-10-26"
tags:
  - standards
  - antigravity-plugins
  - subagents
  - hooks
---

# Antigravity Plugin Standards

## 1. Directory Structure
A native Antigravity plugin repository follows this structure:

```text
plugin-repo/
├── plugin.json         # Manifest declaring plugin name, version, and post-install script
├── pyproject.toml      # Python dependencies and test configuration managed via uv
├── uv.lock             # Locked dependency graph for reproducible hook execution
├── agents/             # Declarative markdown subagent definitions (*.md)
├── rules/              # Plugin-scoped rules
│   └── AGENTS.md       # Consolidated plugin instructions & rules
├── skills/             # On-demand progressive skills
│   └── <skill-name>/
│       └── SKILL.md    # Skill instructions with YAML frontmatter
├── hooks/              # Scripted lifecycle hooks (e.g., SessionStart, PreInvocation)
├── hooks.json          # Lifecycle hook registrations (root-level)
├── scripts/            # Setup and post-install automation (e.g., post_install.sh)
├── tests/              # Automated unit and integration test suite
├── mcp_config.json     # MCP servers configuration (optional)
└── README.md
```

## 2. Manifest (`plugin.json`)
```json
{
  "name": "plugin-name",
  "version": "0.1.0",
  "description": "Short description of the plugin",
  "scripts": {
    "post_install": "scripts/post_install.sh"
  }
}
```

## 3. Declarative Subagents (`agents/<agent-name>.md`)
Subagents are defined as markdown files with YAML frontmatter (`name`, `description`) and registered dynamically at `SessionStart` via `define_subagent` / `define_agent`:
```markdown
---
name: my-subagent
description: Concise summary of the subagent's role and when to invoke it.
---

# Subagent Title (`@my-subagent`)
System instructions, responsibilities, and handoff protocols...
```

## 4. Rules (`rules/AGENTS.md`)
Rules define project-level constraints and standards. Use Markdown alerts (`> [!NOTE]`, `> [!TIP]`, `> [!IMPORTANT]`, `> [!WARNING]`, `> [!CAUTION]`) and explicit code snippets.

## 5. Skills (`skills/<skill-name>/SKILL.md`)
Must begin with YAML frontmatter:
```markdown
---
name: my-skill
description: Comprehensive summary of when and how this skill is used.
---
```
Skills are loaded on demand via progressive disclosure to save context tokens.

## 6. Lifecycle Hooks (`hooks.json` & `hooks/`)
Hooks execute self-contained Python scripts via `uv run` and return `{"injectSteps": [{"systemMessage": {"systemMessage": "..."}}]}` payloads on stdout:
- **`SessionStart`**: Discovers markdown agents in `agents/*.md`, registers subagents with tool permissions (`enable_write_tools`, `enable_subagent_tools`, `enable_mcp_tools`), and injects orchestration and artifact approval protocols.
- **`PreInvocation`**: Triages incoming user prompts (`invocationNum == 0`) to classify intent and inject routing directives.
