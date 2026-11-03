---
plugin_name: "agentic-code-tui"
status: "active"
tags:
  - tui-application
  - terminal-ui
  - textual
  - google-adk
  - google-antigravity
  - component-spec
version: "0.1.0"
---

# Application: agentic-code-tui

## Overview
`agentic-code-tui` is a standalone, full-screen Terminal User Interface (TUI) application combining the **Google Antigravity SDK** (`google-antigravity`) and **Google ADK** (`google-adk`) workflow graph engine into a unified terminal assistant.

It enables developers to operate the full agentic ecosystem directly in their terminal without requiring a GUI IDE:
- **Top Prompt Input**: Persistent prompt and slash command bar with history and autocomplete.
- **Central Transcript Stream**: Scrollable conversation stream with rich Markdown rendering, syntax-highlighted code blocks, and collapsible thought traces.
- **Bottom Status Footer**: Live display of active subagent, lifecycle phase, tool status, and token metrics.
- **Interactive Approval Dialogs**: Responsive modals for approval gates without timeouts.

---

## Architecture & Components

```mermaid
flowchart TD
    UserPrompt["Top Prompt Input Box"] --> Dispatcher["Root Dispatcher"]
    Dispatcher --> Triage["Laya Router System 1 Classifier"]
    
    Triage -->|"coding"| ADKGraph["google-adk Workflow Graph<br/>(Clarify -> Specify -> Plan -> Dev -> Review -> Merge)"]
    Triage -->|"investigation"| InvestigateAgent["@coding-investigate Agent"]
    Triage -->|"monitoring"| MonitorAgent["@coding-monitor Agent"]
    Triage -->|"research"| ResearchAgent["@coding-research Agent"]
    Triage -->|"documentation"| DocumentAgent["@coding-document Agent"]
    Triage -->|"grooming"| GroomingAgent["@coding-groom Agent"]
    Triage -->|"inquiries"| InquiryAgent["Direct Conversational Response"]
    
    ADKGraph --> TUIStream["Central Transcript Stream & Footer"]
    InvestigateAgent --> TUIStream
    MonitorAgent --> TUIStream
    ResearchAgent --> TUIStream
    DocumentAgent --> TUIStream
    GroomingAgent --> TUIStream
    InquiryAgent --> TUIStream
```

### 1. Orchestration Engine (`google-adk Workflow`)
The coding lifecycle is orchestrated deterministically via `google.adk.workflow.Workflow` with explicit `FunctionNode` steps and transition invariants enforcing approval gates (`[GATE_APPROVED]` on `specification.md` and `plan.md`).

### 2. Domain Worker Agents (`google-antigravity Agent`)
Worker subagents are instantiated using `LocalAgentConfig` and standard Antigravity tools (with `invoke_agent` excluded).

### 3. Intent Triage (`laya Router`)
Fast non-autoregressive prompt classification routes requests to target agents or triggers disambiguation prompts when confidence is below 0.40.

---

## Configuration & Usage

Configuration is loaded from `config/agents.yaml`:

```yaml
default:
  model: "gemini-2.5-flash"
  temperature: 0.2
  seed: 42

agents:
  coding_orchestrator:
    model: "gemini-3.8-pro"
    temperature: 0.2
```

### CLI Commands
```bash
# Launch the TUI
make run
# or
agentic-code-tui --config config/agents.yaml
```
