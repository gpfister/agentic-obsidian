---
adr_id: "002"
title: "Agent Display Names Standardization & Dual Plugin Repository Rename"
status: "accepted"
date: "2026-10-30"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/2b726f54-496d-4a46-8461-49d0a86efe79/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/3355c861-6d22-4e70-99cd-7580458d338c/plan.md"
tags:
  - architecture-decision
  - standardization
  - display-names
  - repository-rename
  - conventions
---

# ADR-002: Agent Display Names Standardization & Dual Plugin Repository Rename

## Context
Following the successful migration of technical agent handles to the canonical `coding-*` namespace (see [[architecture/adr-001-standardize-agent-handles|ADR-001]]), two architectural inconsistencies remained across the `agentic` workspace:

1. **Inconsistent Agent UI Display Titles**:
   - In Antigravity's agent picker, dropdowns, and chat headers, agent display names are derived from the top-level Markdown `# Header` in `agent.md` definition files.
   - Headers lacked a uniform prefix (e.g., `# Requirement Clarifier`, `# Issue Investigator`, `# Autonomous Documentation Specialist`, `# Telemetry Monitor`), leading to fragmented visual branding.
2. **Repository Naming Mismatches**:
   - The telemetry plugin directory was named `agentic-monitoring-plugin`, while its primary worker agent is `@coding-monitor` and its remote repository was updated to `agentic-monitor-plugin`.
   - The documentation plugin directory was named `agentic-obsidian-plugin`, which caused naming confusion with the separate documentation vault repository `agentic-obsidian/`. Its primary worker is `@coding-document`, and its remote repository was updated to `agentic-document-plugin`.
   - Local `.git/config` remote URLs for both repositories required explicit synchronization to their official GitHub remotes.

## Decision
1. **Mandate Display Heading Prefix**:
   - All agent definition files and catalogs across the workspace must prepend `"Coding - "` to their top-level Markdown `# ` heading.
   - For `@coding-monitor`, the designated display title is explicitly `# Coding - Monitor Expert (@coding-monitor)`.
2. **Preserve Technical Frontmatter Handles (`name:`)**:
   - All YAML frontmatter `name: <handle>` keys remain completely invariant (e.g., `name: coding`, `name: coding-monitor`, `name: coding-document`). This guarantees 100% backward compatibility for subagent invocation, dispatching, and tool routing.
3. **Dual Plugin Repository Rename**:
   - Rename `agentic-monitoring-plugin/` -> `agentic-monitor-plugin/` across directories, manifests (`plugin.json`), project metadata (`pyproject.toml`), and workspace orchestration (`Makefile`, `.gitignore`, `AGENTS.md`, `README.md`).
   - Rename `agentic-obsidian-plugin/` -> `agentic-document-plugin/` across directories, manifests, project metadata, and workspace orchestration.
   - Retain the Obsidian markdown vault directory `agentic-obsidian/` strictly untouched.
4. **Synchronize Git Remote Origin URLs**:
   - Configure `agentic-monitor-plugin/.git/config` remote origin to `git@github.com:gpfister/agentic-monitor-plugin.git`.
   - Configure `agentic-document-plugin/.git/config` remote origin to `git@github.com:gpfister/agentic-document-plugin.git`.

## Consequences

- **Positive:**
  - Clear visual cohesion across all Antigravity agent dropdowns and chat interfaces.
  - Aligns directory names (`agentic-monitor-plugin`, `agentic-document-plugin`) directly with agent handles (`@coding-monitor`, `@coding-document`).
  - Completely resolves semantic ambiguity between the documentation plugin (`agentic-document-plugin`) and the Obsidian vault repository (`agentic-obsidian`).
  - Seamless Git remotes without redirect overhead.
  - Zero breakage of agent tool routing or user prompt aliases due to handle invariance.
- **Negative:**
  - Local clones require running migration steps (`mv` and `git remote set-url`).
- **Risks & Mitigations:**
  - *Risk*: Broken path references in root `Makefile` or `.gitignore`.
  - *Mitigation*: Comprehensive file modification matrix and automated verification using `make status`.

## References
- Approved Technical Specification: [`specification.md`](file:///Users/gpfister/.gemini/antigravity/brain/2b726f54-496d-4a46-8461-49d0a86efe79/specification.md)
- Vault Archived Specification: [`sessions/attachments/2b726f54-496d-4a46-8461-49d0a86efe79/specification.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/sessions/attachments/2b726f54-496d-4a46-8461-49d0a86efe79/specification.md)
- Approved Implementation Plan: [`plan.md`](file:///Users/gpfister/.gemini/antigravity/brain/3355c861-6d22-4e70-99cd-7580458d338c/plan.md)
- Vault Archived Implementation Plan: [`sessions/attachments/3355c861-6d22-4e70-99cd-7580458d338c/plan.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/sessions/attachments/3355c861-6d22-4e70-99cd-7580458d338c/plan.md)
- Feature Specification & Implementation Note: [[features/agent-display-names-and-plugin-renames|Agent Display Names & Plugin Renames]]
- Plugin Standards: [[architecture/plugin-standards|Antigravity Plugin Standards]]
