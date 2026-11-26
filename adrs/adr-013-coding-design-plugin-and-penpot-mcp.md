---
adr_id: "013"
title: "Coding Design Plugin (@coding-design) with Official PenPot MCP Integration"
status: "accepted"
date: "2026-11-21"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/2b36f2db-6da8-42bd-90a7-3c40cc5d8059/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/590edead-39dc-4a1e-af1d-821b8249862f/plan.md"
tags:
  - architecture-decision
  - agentic-design-plugin
  - penpot
  - mcp
  - dual-mode-relay
  - exclusive-code-change-authority
  - plan-approved
aliases:
  - ADR-013
  - adr-013-coding-design-plugin-and-penpot-mcp
---

# ADR-013: Coding Design Plugin (`@coding-design`) with Official PenPot MCP Integration

## Context

The `agentic` workspace provides specialized plugins for full-lifecycle software engineering (`@coding`), root-cause investigation (`@coding-investigate`), topic research (`@coding-research`), monitoring (`@coding-monitor`), project grooming (`/project-grooming`), and documentation (`@coding-document`), but lacked a dedicated UI/UX design specialist and design-to-code bridge.

Specifically:
1. UI/UX features in `@coding` workflows lacked structured visual specifications, design tokens, and component hierarchies backed by a live design tool.
2. Developers could not bidirectionally author or inspect **PenPot** design files from Antigravity sessions under standardized workspace safety rules.
3. Cross-plugin routing (`agentic-prompt-triage-plugin`), lifecycle orchestration (`agentic-coding-plugin`), and Obsidian archival (`agentic-project-documenting-plugin`) had no design specialist target.
4. Any new design agent must respect Antigravity's 60-second background subagent timeout constraints ([[architecture/adr-005-question-relay-protocol|ADR-005]], [[architecture/adr-006-artifact-approval-relay-protocol|ADR-006]]) and Exclusive Code-Change Authority ([[architecture/adr-012-exclusive-code-change-authority|ADR-012]]).

### Alternatives Considered

1. **Generate UI Code Directly in `@coding-design` Without PenPot or `@coding-develop` Handoff**:
   - *Rejection Rationale*: Violates [[architecture/adr-012-exclusive-code-change-authority|ADR-012]] (Exclusive Code-Change Authority) and bypasses `.worktrees/<slug>` isolation and `@coding-review` Plan-vs-Changes auditing.
2. **Embed PenPot MCP Directly Inside `agentic-coding-plugin`**:
   - *Rejection Rationale*: Breaks modular plugin isolation. Users who do not use PenPot should not inherit PenPot MCP dependencies, and prompt triage needs an independent, dynamically detectable plugin boundary.
3. **Standalone `agentic-design-plugin` Hosting `@coding-design` with Configurable PenPot MCP (Selected)**:
   - *Rationale*: Preserves multi-repo modularity, isolates PenPot MCP configuration in `mcp_config.json`, enforces strict read-only workspace boundaries, and integrates cleanly via Dual-Mode Relay Protocols and dynamic companion detection.

---

## Decision

We introduce **`agentic-design-plugin`** as a standalone Antigravity plugin repository hosting the **`@coding-design`** subagent with official **PenPot MCP** integration:

1. **Standalone Plugin & Portable MCP Configuration**:
   - `agentic-design-plugin/` follows standard Antigravity conventions (`plugin.json`, `mcp_config.json`, `rules/AGENTS.md`, `agents/coding-design/agent.md`, `skills/coding-design/SKILL.md`, `pyproject.toml`, `tests/test_plugin.py`, `README.md`, `LICENSE`).
   - `mcp_config.json` configures the official PenPot MCP server using portable environment-variable-backed transport (e.g., `${PENPOT_MCP_URL:-http://localhost:4401/mcp}` or `npx` stdio) with zero hardcoded machine-local paths.
2. **Bidirectional PenPot Capabilities & Read-Only Workspace Boundary**:
   - `@coding-design` performs PenPot authoring (files, pages, artboards, frames, components, auto-layout, tokens) and inspection/asset extraction via PenPot MCP tools, synthesizing `<appDataDir>/brain/<conversation-id>/design-spec.md`.
   - Per [[architecture/adr-012-exclusive-code-change-authority|ADR-012]], `edit_file` and `run_command` are omitted from `agents/coding-design/agent.md`, and `write_file` is restricted strictly to `design-spec.md`. All repository UI/CSS/test changes are delegated to `@coding-develop` via `@coding`.
3. **Dual-Mode Relay Protocol Compliance**:
   - `@coding-design` operates in both **Standalone Mode** (direct `ask_question` and `write_file(RequestFeedback=True)` as sole final tool call) and **Subagent Mode** (`[QUESTION_RELAY_REQUEST]` and `[ARTIFACT_APPROVAL_REQUEST]` via `send_message` to `caller_id`).
   - In Standalone Mode, upon `design-spec.md` approval and if `agentic-coding-plugin` is installed, `@coding-design` optionally offers to invoke `@coding` using the standardized orchestration prompt.
4. **Dynamic Cross-Plugin Interoperability**:
   - **Root Workspace (`agentic`)**: Registered in `Makefile` (`REPOS`), `.gitignore`, and `AGENTS.md`.
   - **`agentic-coding-plugin` (`@coding`)**: Dynamically detects `agentic-design-plugin` and optionally delegates UI/UX design tasks to `@coding-design` during Phase 2/3 (Specification / Planning).
   - **`agentic-prompt-triage-plugin`**: Dynamically detects `agentic-design-plugin` and routes UI/UX, wireframe, mockup, design-token, and PenPot requests to `@coding-design` when installed, omitting it when uninstalled.
   - **`agentic-project-documenting-plugin` (`@coding-document`)**: Archives approved `design-spec.md` artifacts, PenPot links, and design tokens into `agentic-obsidian/designs/` (and `agentic-obsidian/designs/index.md`).

---

## Consequences

- **Positive:**
  - Establishes a first-class, bidirectional PenPot UI/UX design workflow within the `agentic` ecosystem.
  - Preserves 100% compliance with Exclusive Code-Change Authority ([[architecture/adr-012-exclusive-code-change-authority|ADR-012]]) and timeout-safe Dual-Mode Relay Protocols ([[architecture/adr-005-question-relay-protocol|ADR-005]], [[architecture/adr-006-artifact-approval-relay-protocol|ADR-006]]).
  - Offline `pytest` contract suite (`tests/test_plugin.py`) validates structure, frontmatter, and portability without requiring a live PenPot instance.
- **Negative:**
  - Requires users to run or connect an external PenPot MCP server endpoint to execute live PenPot tool calls.
- **Risks & Mitigations:**
  - *Risk*: PenPot MCP server is unreachable during a session.
  - *Mitigation*: `@coding-design` detects MCP connection/tool failures early, reports exact endpoint/command configuration errors with `README.md` setup guidance, and halts without fabricating design data.
  - *Risk*: Design requests mix visual design and code implementation.
  - *Mitigation*: Tool-level omission of `edit_file` and `run_command` forces code implementation to flow through `@coding` -> `@coding-develop`.
