---
adr_id: "001"
title: "Standardize Subagent Handles, Aliases, and Skill Identifiers to Coding Namespace"
status: "accepted"
date: "2026-10-30"
tags:
  - architecture-decision
  - standardization
  - subagents
  - conventions
---

# ADR-001: Standardize Subagent Handles, Aliases, and Skill Identifiers to Coding Namespace

## Context
Across the `agentic` workspace and its plugin ecosystem (`agentic-coding-plugin`, `agentic-investigate-plugin`, `agentic-monitoring-plugin`, `agentic-obsidian-plugin`, `agentic-research-plugin`, `agentic-triage-plugin`), subagent handles, aliases, and directory names had diverged. In multiple plugins, legacy non-prefixed aliases (`@investigate`, `@monitoring`, `@document`, `@research`), duplicate agent directories (`agents/investigate/`, `agents/monitoring/`, `agents/research/`), and non-prefixed skill directories (`skills/investigate/`, `skills/monitoring/`, `skills/document/`, `skills/research/`) coexisted alongside canonical `coding-*` definitions.

This dual naming model created several architectural problems:
1. **Ambiguous Dispatch & Split Brain**: Multiple aliases and duplicate agent directories caused uncertainty during subagent resolution and dynamic routing.
2. **Inconsistent Developer Experience**: Orchestration instructions, prompt templates, and hook scripts referenced mixed `@<name>` and `@coding-<name>` handles.
3. **Drift in Verification Suites**: Unit tests tested disparate directories and names, leading to fragility across multi-repo CI.

## Decision
We mandate a strict architectural naming convention across all plugins and documentation in the `agentic` workspace:
1. **Canonical Prefix Requirement**: All subagent handles, agent directories, aliases, and skill directories must strictly begin with the `coding-` prefix.
2. **Permanent Eradication of Legacy Aliases**: All non-coding aliases (`investigate`, `monitoring`, `document`, `research`) and non-coding directories are permanently removed. Allowed aliases must also adhere to `^coding(-.*)?$` (e.g., `coding-monitoring` alongside `coding-monitor`).
3. **Directory and Frontmatter Alignment**:
   - Investigation: `agents/coding-investigate/` & `skills/coding-investigate/` (`name: coding-investigate`, handle `@coding-investigate`).
   - Monitoring: `agents/coding-monitor/` & `skills/coding-monitor/` (`name: coding-monitor`, handle `@coding-monitor`).
   - Documentation: `agents/coding-document/` & `skills/coding-document/` (`name: coding-document`, handle `@coding-document`).
   - Research: `agents/coding-research/` & `skills/coding-research/` (`name: coding-research`, handle `@coding-research`).
4. **Historical Session Log Immutability**: Historical session transcripts under `agentic-obsidian/sessions/` are treated as immutable runtime audit records and are not retroactively modified.

## Consequences

- **Positive:**
  - Unified namespace eliminates collision and routing ambiguity across triage hooks and subagent invokers.
  - Predictable directory layouts simplify testing, tooling, and plugin packaging standards.
  - Clear separation between standard user prompts and specialized `@coding-*` agentic subagent dispatch.
- **Negative:**
  - Breaking change for external scripts or user workflows relying directly on legacy `@investigate`, `@monitoring`, `@document`, or `@research` handles.
- **Risks & Mitigations:**
  - *Risk*: Broken links in documentation and rules pointing to old skill paths.
  - *Mitigation*: Systematic workspace-wide hyperlinking updates and test assertions validating path existence.

## References
- Approved Technical Specification: [`specification.md`](file:///Users/gpfister/.gemini/antigravity/brain/65353adb-b25d-4fcc-a7a2-f0e776093a2e/specification.md)
- Approved Implementation Plan: [`plan.md`](file:///Users/gpfister/.gemini/antigravity/brain/86b5a736-9acd-446d-b5c8-7deea83752fd/plan.md)
- Feature Note: [[features/standardize-agent-handles|Standardize Agent Handles & Remove Non-Coding Aliases]]
- Plugin Standards: [[architecture/plugin-standards|Antigravity Plugin Standards]]
