---
adr_id: "008"
title: "Agentic Code TUI Architecture: Textual UI, Google ADK Workflows, and Antigravity Agent Binding"
status: "accepted"
date: "2026-11-03"
tags:
  - architecture-decision
  - tui
  - textual
  - google-adk
  - google-antigravity
  - laya-router
aliases:
  - ADR-008
  - adr-008-agentic-code-tui-architecture
---

# ADR-008: Agentic Code TUI Architecture

## Context
While the individual plugins in the `agentic` ecosystem operate inside Google Antigravity sessions, developers frequently need an autonomous, terminal-native assistant capable of executing full multi-stage coding workflows outside of full IDE sessions.

A dedicated Terminal User Interface (TUI) was required to integrate:
1. Dynamic prompt intent classification.
2. Coordinated 6-stage software development workflows (`clarify -> specify -> plan -> develop -> review -> merge`).
3. Domain subagents for issue investigation, stream surveillance, literature research, documentation, and project grooming.
4. Non-blocking interactive approval gates.

---

## Decision

We establish the following architectural design for `agentic-code-tui`:

1. **Terminal UI Framework**: [Textual](https://textual.textualize.io/) is selected for its async reactive architecture, CSS-driven layout styling, robust widget system, and modal screen capabilities.
2. **Workflow Graph Runtime**: `google-adk` (`google.adk.workflow.Workflow`) is used to construct deterministic state graphs for the 6-stage coding lifecycle, enforcing invariants and approval gates without ad-hoc state tracking.
3. **Agent Runtime**: Domain workers are instantiated via `google-antigravity.Agent` and `LocalAgentConfig`.
4. **Tool Safety Invariant**: Antigravity tool bindings strictly omit `invoke_agent` / `invoke_subagent`, ensuring that routing and multi-agent coordination remain under the explicit authority of the Root Dispatcher and Google ADK.
5. **Intent Routing**: Laya Router System 1 model performs fast non-autoregressive prompt classification.
6. **Sub-repo Isolation**: The application lives in its own Git repository (`agentic-code-tui`) within the workspace.

---

## Consequences

- **Positive**:
  - Provides a complete, standalone terminal-native developer experience.
  - Strict phase transitions prevent skipping critical clarification, specification, or planning gates.
  - Native integration with existing Antigravity ecosystem and configurations.
- **Negative**:
  - Requires maintaining dependencies (`textual`, `google-adk`, `google-antigravity`, `laya`) across Python 3.11+ environments.
