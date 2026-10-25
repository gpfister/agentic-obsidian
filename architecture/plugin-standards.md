---
title: "Antigravity Plugin Standards & Structure"
date: "2026-10-25"
tags:
  - standards
  - antigravity-plugins
---

# Antigravity Plugin Standards

## 1. Directory Structure
A native Antigravity plugin repository follows this structure:

```text
plugin-repo/
├── plugin.json         # Manifest declaring plugin name and metadata
├── rules/              # Plugin-scoped rules
│   └── AGENTS.md       # Consolidated plugin instructions & rules
├── skills/             # On-demand progressive skills
│   └── <skill-name>/
│       └── SKILL.md    # Skill instructions with YAML frontmatter
├── hooks/              # Scripted lifecycle hooks
├── hooks.json          # Lifecycle hook registrations (root-level)
├── mcp_config.json     # MCP servers configuration (optional)
└── README.md
```

## 2. Manifest (`plugin.json`)
```json
{
  "name": "plugin-name",
  "version": "0.1.0",
  "description": "Short description of the plugin"
}
```

## 3. Rules (`rules/AGENTS.md`)
Rules define project-level constraints and standards. Use Markdown alerts (`> [!NOTE]`, `> [!TIP]`, `> [!IMPORTANT]`, `> [!WARNING]`, `> [!CAUTION]`) and explicit code snippets.

## 4. Skills (`skills/<skill-name>/SKILL.md`)
Must begin with YAML frontmatter:
```markdown
---
name: my-skill
description: Comprehensive summary of when and how this skill is used.
---
```
Skills are loaded on demand via progressive disclosure to save context tokens.
