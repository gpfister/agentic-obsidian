---
title: "Research Knowledge Base Index"
type: "research-index"
status: "active"
tags:
  - research
  - index
  - topics
  - requests
  - knowledge-base
last_updated: "2026-10-30"
---

# Research Knowledge Base Index

Welcome to the centralized **Research Knowledge Base** of the `agentic` workspace, maintained by the autonomous research subagent ([`@research`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-research-plugin/agents/research/agent.md)) provided by [`agentic-research-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/plugins/agentic-research-plugin.md).

---

## 1. Directory Structure & Taxonomy

The research vault organizes synthesized knowledge across two structural tiers:

```text
research/
├── index.md                 # Master index, taxonomy, and request registry (this file)
├── topics/                  # Consolidated thematic deep-dives and evolving subject syntheses
│   └── <topic-slug>/        # Dedicated directory per topic containing synthesis notes and references
└── requests/                # Point-in-time research request deliverables and investigation logs
    └── <request-id>-<slug>.md # Individual research findings with source citations and query trajectories
```

### 1.1 Topics (`research/topics/`)
Topics represent long-lived domain areas, architectures, frameworks, or problem spaces. As new research requests are processed, findings are synthesized into topic notes to form a cumulative knowledge repository.

### 1.2 Requests (`research/requests/`)
Requests capture the snapshot findings of specific user inquiries executed by `@research`. Each request file documents the original query, web search queries executed, retrieved sources, synthesized report, and links to relevant topic notes.

---

## 2. Topic Taxonomy

| Topic Slug | Category | Description | Topic Index File | Associated Requests | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `agentic-architectures` | System Architecture | Design patterns for autonomous agent coordination, subagents, and memory systems. | [topics/agentic-architectures/topic.md](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/research/topics/agentic-architectures/topic.md) | — | Planned |
| `intent-routing` | NLP & Routing | Fast non-autoregressive routing, System 1 classifiers, and dynamic intent discovery. | [topics/intent-routing/topic.md](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/research/topics/intent-routing/topic.md) | — | Planned |
| `telemetry-monitoring` | Observability | Application log tailing, container surveillance, serial consoles, and automated error detection. | [topics/telemetry-monitoring/topic.md](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/research/topics/telemetry-monitoring/topic.md) | — | Planned |
| `knowledge-synthesis` | Information Retrieval | Methodologies for web retrieval, source triangulation, literature review, and obsidian indexing. | [topics/knowledge-synthesis/topic.md](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/research/topics/knowledge-synthesis/topic.md) | — | Planned |

---

## 3. Research Request Registry

The following log tracks all completed research requests executed by `@research`:

| Request ID | Date | Topic | Title / Objective | Report Link | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| *No requests recorded yet* | — | — | Run `@research <topic>` to generate initial research report | — | — |

---

## 4. Operational Protocols for `@research`

When the `@research` agent executes a research task:
1. **Search & Synthesis**: Query the web (`search_web`), triangulate reputable sources, and synthesize findings with inline citations.
2. **Artifact Generation**: Generate `<appDataDir>/brain/<conversation-id>/research-report.md` with `RequestFeedback=True` as the sole and final tool call of that turn for user approval.
3. **Vault Integration**:
   - Save the request report in [`research/requests/<request-id>-<slug>.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/research/requests/).
   - Update or create corresponding topic notes in [`research/topics/<topic-slug>/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/research/topics/).
   - Append the entry into the **Research Request Registry** in this index file ([`research/index.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/research/index.md)).
