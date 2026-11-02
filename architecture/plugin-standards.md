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
├── agents/             # Declarative markdown subagent definitions (coding-*/ or coding-*.md)
├── rules/              # Plugin-scoped rules
│   └── AGENTS.md       # Consolidated plugin instructions & rules
├── skills/             # On-demand progressive skills prefixed with coding-*
│   └── coding-<skill-name>/
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
name: coding-my-subagent
description: Concise summary of the subagent's role and when to invoke it.
---

# Coding - Subagent Title (`@coding-my-subagent`)
System instructions, responsibilities, and handoff protocols...
```

> [!IMPORTANT] Subagent Technical Naming Standard (ADR-001)
> All subagent handles, aliases, and agent directories must strictly start with the prefix `coding-` (e.g., `@coding-investigate`, `@coding-monitor`, `@coding-document`, `@coding-research`). Aliases not prefixed with `coding-` are prohibited.

> [!NOTE] Subagent Display Title Standardization (ADR-002)
> All agent markdown definition files and catalogs must prepend `"Coding - "` to their primary Markdown `# ` heading (e.g., `# Coding - Workflow Orchestrator`, `# Coding - Monitor Expert`, `# Coding - Autonomous Documentation Specialist`). The technical identifier `name: coding-*` in YAML frontmatter remains invariant for 100% backward compatibility.

> [!TIP] Plugin Repository Naming Convention (ADR-004)
> All Antigravity companion plugin repositories follow the descriptive action grammar `agentic-<action-descriptor>-plugin` (e.g., `agentic-coding-plugin`, `agentic-prompt-triage-plugin`, `agentic-issue-investigation-plugin`, `agentic-task-monitoring-plugin`, `agentic-topic-researching-plugin`, `agentic-project-documenting-plugin`, `agentic-project-grooming-plugin`). While repository and manifest names reflect this grammar, subagent handles (`@coding-*`) and skill directories remain strictly invariant for runtime stability (see [[architecture/adr-004-standardize-plugin-repository-names|ADR-004]]).

## 4. Rules (`rules/AGENTS.md`)
Rules define project-level constraints and standards. Use Markdown alerts (`> [!NOTE]`, `> [!TIP]`, `> [!IMPORTANT]`, `> [!WARNING]`, `> [!CAUTION]`) and explicit code snippets.

## 5. Skills (`skills/<skill-name>/SKILL.md`)
Must begin with YAML frontmatter and reside in a folder matching the canonical `coding-*` skill identifier:
```markdown
---
name: coding-my-skill
description: Comprehensive summary of when and how this skill is used.
---
```
Skills are loaded on demand via progressive disclosure to save context tokens. All skill folders and identifiers must be prefixed with `coding-` (see [[architecture/adr-001-standardize-agent-handles|ADR-001]]).

## 6. Lifecycle Hooks (`hooks.json` & `hooks/`)
Hooks execute self-contained Python scripts via `uv run` and return `{"injectSteps": [{"systemMessage": {"systemMessage": "..."}}]}` payloads on stdout:
- **`SessionStart`**: Discovers markdown agents in `agents/*.md`, registers subagents with tool permissions (`enable_write_tools`, `enable_subagent_tools`, `enable_mcp_tools`), and injects orchestration and artifact approval protocols.
- **`PreInvocation`**: Triages incoming user prompts (`invocationNum == 0`) to classify intent and inject routing directives.
