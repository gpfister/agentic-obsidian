---
plugin_name: "agentic-prompt-triage-plugin"
status: "active"
tags:
  - antigravity-plugin
  - component-spec
  - prompt-triage
  - laya-router
  - intent-classification
version: "0.1.0"
---

# Plugin: agentic-prompt-triage-plugin

## Overview
`agentic-prompt-triage-plugin` is the dynamic prompt triage and intent classification plugin within the `agentic` ecosystem. It runs a `PreInvocation` lifecycle hook using the non-autoregressive **Laya Router** System 1 model (`Router(preload=False)` `default`, selected over `typed-decisions` via empirical evaluation) to classify incoming user prompts across semantic `choice` criteria keys (`inquiry`, `coding`, `monitoring`, `investigation`, `research`, `documentation`, `grooming`, `design`) and parallel `noul` checks (`requires_changes`, `is_mixed`):
- **Coding Requests** (`coding` $\rightarrow$ `@coding`): Evaluated if [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/plugin.json) is installed.
- **UI/UX & PenPot Design Requests** (`design` $\rightarrow$ `@coding-design`): Evaluated if [`agentic-design-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-design-plugin/plugin.json) is installed.
- **Project Grooming Requests** (`grooming` $\rightarrow$ `/project-grooming <prompt>`): Evaluated if [`agentic-project-grooming-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/plugin.json) is installed.
- **Task Monitoring Requests** (`monitoring` $\rightarrow$ `@coding-monitor`): Evaluated if [`agentic-task-monitoring-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-task-monitoring-plugin/plugin.json) is installed.
- **Issue Investigation Requests** (`investigation` $\rightarrow$ `@coding-investigate`): Evaluated if [`agentic-issue-investigation-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/plugin.json) is installed.
- **Topic Research Requests** (`research` $\rightarrow$ `@coding-research`): Evaluated if [`agentic-topic-researching-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-topic-researching-plugin/plugin.json) is installed.
- **Documentation Requests** (`documentation` $\rightarrow$ `@coding-document`): Evaluated if [`agentic-project-documenting-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/plugin.json) is installed.
- **Inquiries involving no changes** (`inquiry` $\rightarrow$ intent `"question"`): Always active regardless of installed plugins.
- **Mixed requests** (`is_mixed` `noul` $\rightarrow$ `"mixed"`): Detected when prompts combine multiple workflows across active capabilities (omitted from `choice` criteria).

---

## Components

### 1. Manifest, Metadata & Rules
- **Manifest**: [`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/plugin.json) declaring name `agentic-prompt-triage-plugin`, version `0.1.0`, license `MIT`.
- **Environment**: Managed via `uv` with `pyproject.toml` dependencies (`laya>=0.4.15`, `pydantic>=2.10.0`, `pytest>=8.0.0`).
- **Rules**: [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/rules/AGENTS.md) establishing `Router(preload=False)` default, parallel `choice` + `noul` decomposition, calibrated confidence thresholds ($\ge 0.80$), `usage` diagnostics, prompt sanitization, and fallback protocols.
- **Lifecycle Hooks**: Registered in [`hooks.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/hooks.json):
  - **`PreInvocation`**: Runs `uv run hooks/prompt_triage.py` on the initial turn (`invocationNum == 0`).

---

## Dynamic Triage Protocol (`hooks/prompt_triage.py`)

### 1. Plugin Discovery (`detect_installed_plugins`)
The triage engine scans for active capability plugins through:
1. Explicit custom plugins list (in tests).
2. Environment variable `INSTALLED_PLUGINS`.
3. Workspace paths inspection (scanning workspace directories and parent directories for valid `plugin.json` manifests).
4. Global Antigravity plugin paths (`~/.gemini/config/plugins`, `~/.gemini/antigravity-cli/plugins`).

### 2. Dynamic Specification (`build_dynamic_triage_spec`)
The routing criteria adapts dynamically using semantic keys:
- `inquiry` (always enabled, maps to intent `"question"`): `"Read-only question, explanation, or code review making no file, code, design, or documentation changes."`
- If `agentic-coding-plugin` is active: `coding` $\rightarrow$ directs main agent to invoke `@coding`.
- If `agentic-design-plugin` is active: `design` $\rightarrow$ directs main agent to invoke `@coding-design`.
- If `agentic-project-grooming-plugin` is active: `grooming` $\rightarrow$ directs main agent to execute `/project-grooming <prompt>`.
- If `agentic-task-monitoring-plugin` (or `agentic-monitor-plugin` / `agentic-monitoring-plugin`) is active: `monitoring` $\rightarrow$ directs main agent to invoke `@coding-monitor`.
- If `agentic-issue-investigation-plugin` (or `agentic-investigate-plugin`) is active: `investigation` $\rightarrow$ directs main agent to invoke `@coding-investigate`.
- If `agentic-topic-researching-plugin` (or `agentic-research-plugin`) is active: `research` $\rightarrow$ directs main agent to invoke `@coding-research`.
- If `agentic-project-documenting-plugin` (or `agentic-document-plugin` / `agentic-obsidian-plugin`) is active: `documentation` $\rightarrow$ directs main agent to invoke `@coding-document`.
- If any capability is active: registers `response_to_intent["mixed"]` and `response_to_directive["mixed"]` (evaluated via `is_mixed` `noul`, excluded from `choice` criteria).

### 3. Single-Pass Parallel `choice` + `noul` Execution & Routing Logic

```text
User Input Prompt (invocationNum == 0)
         │
         ▼
Sanitize Prompt (strip XML tags, timestamps, trailing file paths)
         │
         ▼
Detect Installed Plugins (coding, design, grooming, monitoring, investigate, research, document)
         │
         ▼
Single-Pass Laya Router System 1 Prediction (Router(preload=False), default model)
  ├── "about" (choice, TRIAGE_QUESTION = "Which single agent or action should handle `prompt`?")
  ├── "requires_changes" (noul, mutating intent check)
  └── "is_mixed" (noul, multi-workflow check; omitted if only inquiry enabled)
         │
         ▼
Inspect res["usage"] (truncated / options warnings) + Extract calibrated ans["confidence"]
Apply is_mixed & requires_changes gating (exempting monitoring, investigation, research)
         │
   Confidence >= 0.80 ?
  ├── (Yes) ──> Inject systemMessage with Prompt Triage Directive
  │             ├── question (inquiry): direct response, no code changes
  │             ├── coding: invoke @coding subagent
  │             ├── design: invoke @coding-design subagent
  │             ├── grooming: run /project-grooming <prompt>
  │             ├── monitoring: invoke @coding-monitor subagent
  │             ├── investigation: invoke @coding-investigate subagent
  │             ├── research: invoke @coding-research subagent
  │             ├── documentation: invoke @coding-document subagent
  │             └── mixed: reject prompt, ask user to split requests
  │
  └── (No) ───> Ambiguous (confidence < 0.80)
                Inject directive instructing main agent to ask user
                via ask_question with dynamic semantic candidate choices
```

---

## Testing & Empirical Evaluation

Automated tests and benchmarks in [`agentic-prompt-triage-plugin/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/):
- [`tests/test_plugin.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/tests/test_plugin.py): Manifest, license, rules, and hook definitions (`12/12` passing).
- [`tests/test_prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/tests/test_prompt_triage.py): Dynamic plugin detection, semantic spec building, parallel `choice` + `noul` decomposition, `requires_changes` gating & non-mutating exemption, `is_mixed` rejection, calibrated confidence, `usage` warnings, prompt cleaning, CLI subprocess execution, and single-turn invocation guards (`89/89` passing; `101/101` total).
- [`scripts/eval_laya_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/scripts/eval_laya_triage.py): Empirical benchmark across 10 prompt categories comparing `default` (`Router(preload=False)`) vs `typed-decisions`, confirming `default` as the superior router configuration.
