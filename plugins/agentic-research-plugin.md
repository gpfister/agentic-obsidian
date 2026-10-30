---
plugin_name: "agentic-research-plugin"
status: "active"
tags:
  - antigravity-plugin
  - component-spec
  - research
  - information-retrieval
  - web-search
  - subagents
  - obsidian-vault
version: "0.1.0"
---

# Plugin: agentic-research-plugin

## Overview
`agentic-research-plugin` is the autonomous web research, topic investigation, and knowledge synthesis companion plugin within the `agentic` ecosystem. It bundles a dedicated declarative subagent ([`@research`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-research-plugin/agents/research/agent.md)), a progressive-disclosure research skill ([`skills/research/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-research-plugin/skills/research/SKILL.md)), multi-source retrieval protocols, an interactive research assessment approval gate (`research-report.md`), and deep two-way integration with the [`agentic-obsidian`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/research/index.md) knowledge vault.

---

## Components

### 1. Manifest, Metadata & Rules
- **Manifest**: [`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-research-plugin/plugin.json) declaring name `agentic-research-plugin`, version `0.1.0`, license `MIT`.
- **Environment**: Managed via `uv` with `pyproject.toml` dependencies (`pydantic>=2.10.0`, `pytest>=8.0.0`).
- **Rules**: [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-research-plugin/rules/AGENTS.md) enforcing:
  - **Evidence-Backed Synthesis**: Every assertion must link to verifiable sources or official documentation.
  - **Read-Only / Non-Destructive Operation**: No modification of application source code or production resources.
  - **Interactive Gate Protocol**: Turn terminates solely on user-facing artifact presentation.
  - **Vault Indexing Discipline**: Systematic topic categorization and request logging.

### 2. Declarative Markdown Subagents (`agents/`)
Subagents are organized in dedicated directories with YAML frontmatter:

| Agent Directory / File | Handle / Name | Role & Capabilities | Tool Permissions |
| :--- | :--- | :--- | :--- |
| [`agents/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-research-plugin/agents/agent.md) | `research-catalog` | **Research Subagent Catalog**: Architecture, subagent registry, tool configurations, and research operating models. | `write=False`, `subagent=False`, `mcp=False` |
| [`agents/research/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-research-plugin/agents/research/agent.md) | `@research` (Aliases: `@coding-research`, `research`) | **Topic & Web Research Investigator**: Autonomous topic researcher executing web searches, reading sources, synthesizing structured markdown reports, archiving findings in Obsidian, and presenting approval gates. | `search_web`, `read_file`, `write_file`, `list_dir`, `ask_question`, `send_message` |

### 3. Progressive-Disclosure Skills (`skills/`)
| Skill Name | Path | Description | Invocation |
| :--- | :--- | :--- | :--- |
| `research` | [`skills/research/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-research-plugin/skills/research/SKILL.md) | Structured workflows for web search query expansion, source evaluation, citation formatting, synthesis generation, and Obsidian vault indexing. | Progressive disclosure / `@research` invocation |

---

## Research & Knowledge Synthesis Workflow

```text
User Prompt (@research)
          │
          ▼
Deconstruct Query & Identify Domain
          │
          ▼
Search Web (search_web across multiple angles)
          │
          ▼
Read & Evaluate Sources (relevance, recency, credibility)
          │
          ▼
Synthesize Structured Report & Extract Key Findings
          │
          ├────────────────────────────────────────┐
          ▼                                        ▼
Save to Obsidian Vault                   Write Session Artifact
- research/requests/<id>-<slug>.md       <appDataDir>/brain/<id>/research-report.md
- research/topics/<slug>/                (RequestFeedback=True as sole final call)
- Update research/index.md                         │
                                                   ▼
                                         [Interactive User Gate]
```

### Interactive Artifact Approval Gate
1. **Artifact Location**: `<appDataDir>/brain/<conversation-id>/research-report.md`.
2. **Metadata**: `ArtifactMetadata={"RequestFeedback": True, "UserFacing": True, "Summary": "..."}`.
3. **Sole & Final Tool Call Rule**: `write_file(..., RequestFeedback=True)` MUST be the sole and final tool call of the turn so Antigravity renders interactive **Approve / Reject** feedback buttons.

---

## Obsidian Vault Integration (`agentic-obsidian`)

When `agentic-obsidian` is present in the workspace, `@research` automatically catalogs findings into the vault hierarchy:

- **Master Index**: [`research/index.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/research/index.md) maintaining topic taxonomy and the completed request registry.
- **Topic Syntheses**: [`research/topics/<topic-slug>/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/research/topics/) aggregating long-term domain knowledge across multiple requests.
- **Request Records**: [`research/requests/<request-id>-<slug>.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/research/requests/) preserving raw queries, retrieved citations, and exact synthesized deliverables.

> [!NOTE]
> If `agentic-obsidian` is not present or inaccessible, the agent seamlessly falls back to session-only artifact emission without failing.

---

## Dynamic Prompt Triage Integration

When used alongside [`agentic-triage-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/plugins/agentic-triage-plugin.md), the Laya Router dynamically detects `agentic-research-plugin`'s presence and classifies incoming user research queries to `@research` automatically.

---

## Installation & Usage

```bash
# Enable plugin in Antigravity session:
agy plugin enable agentic-research-plugin

# Invoke research directly:
@research Investigate modern multi-agent consensus protocols

# Alias invocation:
@coding-research Deep dive into fast non-autoregressive routing models
```
