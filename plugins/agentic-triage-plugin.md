---
plugin_name: "agentic-triage-plugin"
status: "active"
tags:
  - antigravity-plugin
  - component-spec
  - prompt-triage
  - laya-router
  - intent-classification
version: "0.1.0"
---

# Plugin: agentic-triage-plugin

## Overview
`agentic-triage-plugin` is the dynamic prompt triage and intent classification plugin within the `agentic` ecosystem. It runs a `PreInvocation` lifecycle hook using the non-autoregressive **Laya Router** System 1 model to classify incoming user prompts and dynamically adapt routing based on which plugins are currently installed:
- **Coding Requests** (`response 2` $\rightarrow$ `@coding`): Evaluated if [`agentic-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/plugin.json) is installed.
- **Task Monitoring Requests** (`@monitoring`): Evaluated if [`agentic-monitoring-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-monitoring-plugin/plugin.json) is installed.
- **Issue Investigation Requests** (`@investigate`): Evaluated if [`agentic-investigate-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-investigate-plugin/plugin.json) is installed.
- **Inquiries involving no changes** (`response 1`): Always active regardless of installed plugins.
- **Mixed requests**: Identified when prompts combine multiple intents across active capabilities.

---

## Components

### 1. Manifest, Metadata & Rules
- **Manifest**: [`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/plugin.json) declaring name `agentic-triage-plugin`, version `0.1.0`, license `MIT`.
- **Environment**: Managed via `uv` with `pyproject.toml` dependencies (`laya>=0.4.15`, `pydantic>=2.10.0`, `pytest>=8.0.0`).
- **Rules**: [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/rules/AGENTS.md) establishing dynamic plugin detection rules, confidence thresholds ($\ge 0.40$), prompt sanitization, and fallback protocols.
- **Lifecycle Hooks**: Registered in [`hooks.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/hooks.json):
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
The routing criteria adapts dynamically:
- `response 1` is always: `"one or more inquiries involving no changes"`.
- If `agentic-coding-plugin` is active: adds `"one or more code change request"` $\rightarrow$ directs main agent to invoke `@coding`.
- If `agentic-monitoring-plugin` is active: adds `"one or more task monitoring request"` $\rightarrow$ directs main agent to invoke `@monitoring`.
- If `agentic-investigate-plugin` is active: adds `"one or more issue investigation request"` $\rightarrow$ directs main agent to invoke `@investigate`.
- If any capability is active: appends the mixed request option $\rightarrow$ directs main agent to reject mixed prompts.

### 3. Execution & Routing Logic

```text
User Input Prompt (invocationNum == 0)
         │
         ▼
Sanitize Prompt (strip XML tags, timestamps, trailing file paths)
         │
         ▼
Detect Installed Plugins (coding, monitoring, investigate)
         │
         ▼
Laya Router System 1 Prediction ("What is this prompt about ?")
         │
   Confidence >= 0.40 ?
  ├── (Yes) ──> Inject systemMessage with Prompt Triage Directive
  │             ├── question: direct response, no code changes
  │             ├── coding: invoke @coding subagent
  │             ├── monitoring: invoke @monitoring subagent
  │             ├── investigation: invoke @investigate subagent
  │             └── mixed: reject prompt, ask user to split requests
  │
  └── (No) ───> Ambiguous (confidence < 0.40)
                Inject directive instructing main agent to ask user
                via ask_question with dynamic candidate choices
```

---

## Testing

Automated tests in [`agentic-triage-plugin/tests/`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/tests/):
- [`tests/test_plugin.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/tests/test_plugin.py): Manifest, license, rules, and hook definitions.
- [`tests/test_prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/tests/test_prompt_triage.py): Dynamic plugin detection, spec building across all combinations (all, only coding, only investigate, only monitoring, none), intent routing, prompt cleaning, CLI subprocess execution, and single-turn invocation guards.
