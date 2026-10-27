# Walkthrough: Subagents & Pre-Prompt Triage Hooks for `gp-coding-plugin`

The first set of specialized subagents, the ADK 2.0 coding orchestrator workflow, the telemetry monitoring agent, and the Laya Router pre-invocation prompt triage hook have been implemented and verified in [`gp-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin).

---

## 1. What Was Built

### A. Pre-Invocation Prompt Triage Hook
- **Hook Manifest**: [`hooks.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/hooks.json) registers the `prompt-triage` handler on `PreInvocation`.
- **Triage Script**: [`hooks/prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/hooks/prompt_triage.py) and engine [`src/gp_coding/triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/src/gp_coding/triage.py).
- **Laya Router Pipeline**:
  1. **Stage 1 (Session Scope Guard)**: Evaluates if input is within software engineering session scope (threshold $\ge 0.80$; bypassed on first turn). Prompts user if out-of-scope.
  2. **Stage 2 (Subagent Waiting Route)**: Detects if input is for an active workflow subagent awaiting input (`coding-specify`, `coding-plan`, `coding-review`) and directs message to that subagent via `send_message`.
  3. **Stage 3 (Intent Classification)**:
     - **Question**: Injects `"Prompt Triage Directive: Do not make any changes."`
     - **Coding Request**: Injects instruction to invoke `@coding` (`coding-orchestrator`), or alerts about an active workflow.
     - **Monitoring Request**: Injects instruction to invoke `@monitoring` (`coding-monitoring`).

### B. Coding Orchestrator ADK Workflow (`@coding`)
- **Workflow Graph**: [`src/gp_coding/agents/orchestrator.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/src/gp_coding/agents/orchestrator.py) implements Google ADK 2.0 `Workflow` with 6 interconnected stages:
  `coding-clarify` $\rightarrow$ `coding-specify` $\rightarrow$ `coding-plan` $\rightarrow$ `coding-develop` $\rightarrow$ `coding-review` $\rightarrow$ `coding-merge`.
- **Concurrency & State Management**: [`src/gp_coding/state.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/src/gp_coding/state.py) enforces that **only one coding workflow runs at a time** and tracks step transitions and background waiting subagents.

### C. Worker Subagents (`src/gp_coding/agents/`)
All workers are implemented using `google-antigravity` (`LocalAgentConfig` + `Agent`) with matched complexity models and tools:

| Subagent | Role | Model | Key Tools |
| :--- | :--- | :--- | :--- |
| `coding-clarify` | Matt Pocock `/grill-me` interviewer | `gemini-3.8-flash` | `ask_question`, `view_file`, `search_web` |
| `coding-specify` | Consolidates requirements into `specification.md` | `gemini-3.8-flash` | `create_file`, `edit_file`, `view_file`, `ask_question` |
| `coding-plan` | Architecture & multi-repo `plan.md` generator | `gemini-3.8-pro` | `create_file`, `edit_file`, `view_file`, `run_command`, `ask_question` |
| `coding-develop` | Per-repo isolated developer (`coding-develop-<repo>`) | `gemini-3.8-pro` | `run_command`, `create_file`, `edit_file`, `view_file` |
| `coding-review` | Cross-repo consistency & spec compliance verifier | `gemini-3.8-pro` | `view_file`, `run_command`, `ask_question` |
| `coding-merge` | Local merge or GitHub PR creator | `gemini-3.8-flash-lite` | `run_command`, `view_file`, `ask_question` |
| `coding-monitoring` (`@monitoring`) | Stream & log watcher with pause gates | `gemini-3.8-flash-lite` | `run_command`, `view_file`, `schedule`, `ask_question` |

### D. Structured Output Schemas & Markdown Converters
- [`src/gp_coding/schemas.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/src/gp_coding/schemas.py) defines rigid Pydantic models for inter-agent communication:
  - `ClarificationSummary`
  - `SpecificationDocument`
  - `PlanDocument`
  - `DevelopReport`
  - `ReviewReport`
  - `MergeReport`
  - `MonitoringAlert`
- Includes dedicated Markdown converters that render structured JSON into formatted markdown blocks and artifacts.

### E. Documentation & Skills
- [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/rules/AGENTS.md): Updated to declare mentionable subagents, invocation rules, background awaiting protocols, and hook behaviors.
- [`skills/coding-orchestrator/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/skills/coding-orchestrator/SKILL.md): Orchestrator skill reference.
- [`skills/coding-monitoring/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/skills/coding-monitoring/SKILL.md): Monitoring skill reference.
- [`obsidian-vault/plugins/gp-coding-plugin.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/obsidian-vault/plugins/gp-coding-plugin.md): Updated knowledge base documentation.

---

## 2. Verification & Test Results

Executed automated unit test suite with 32 comprehensive tests across 5 test modules:

```bash
cd gp-coding-plugin && .venv/bin/pytest -v
```

```text
============================= test session starts ==============================
platform darwin -- Python 3.13.0, pytest-9.1.1, pluggy-1.7.0
rootdir: /Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin
configfile: pyproject.toml
plugins: anyio-4.15.1
collected 32 items

tests/test_agents.py::test_clarify_agent_config PASSED                   [  3%]
tests/test_agents.py::test_specify_agent_config PASSED                   [  6%]
tests/test_agents.py::test_plan_agent_config PASSED                      [  9%]
tests/test_agents.py::test_develop_agent_config PASSED                   [ 12%]
tests/test_agents.py::test_review_agent_config PASSED                    [ 15%]
tests/test_agents.py::test_merge_agent_config PASSED                     [ 18%]
tests/test_agents.py::test_monitoring_agent_config PASSED                [ 21%]
tests/test_agents.py::test_coding_orchestrator_workflow_graph PASSED     [ 25%]
tests/test_agents.py::test_orchestrator_stage_events PASSED              [ 28%]
tests/test_plugin.py::test_post_install_script_exists_and_executable PASSED [ 31%]
tests/test_plugin.py::test_plugin_manifest_valid PASSED                  [ 34%]
tests/test_plugin.py::test_hooks_manifest_valid PASSED                   [ 37%]
tests/test_plugin.py::test_skills_exist PASSED                           [ 40%]
tests/test_plugin.py::test_rules_exist PASSED                            [ 43%]
tests/test_prompt_triage.py::test_triage_stage1_session_start_always_in_scope PASSED [ 46%]
tests/test_prompt_triage.py::test_triage_stage1_out_of_scope_flagged PASSED [ 50%]
tests/test_prompt_triage.py::test_triage_stage2_waiting_subagent_routed PASSED [ 53%]
tests/test_prompt_triage.py::test_triage_stage3_question_intent PASSED   [ 56%]
tests/test_prompt_triage.py::test_triage_stage3_coding_intent PASSED     [ 59%]
tests/test_prompt_triage.py::test_triage_stage3_monitoring_intent PASSED [ 62%]
tests/test_prompt_triage.py::test_triage_hook_script_cli PASSED          [ 65%]
tests/test_schemas.py::test_clarification_schema_and_markdown PASSED     [ 68%]
tests/test_schemas.py::test_specification_schema_and_markdown PASSED     [ 71%]
tests/test_schemas.py::test_plan_schema_and_markdown PASSED              [ 75%]
tests/test_schemas.py::test_develop_report_schema_and_markdown PASSED    [ 78%]
tests/test_schemas.py::test_review_report_schema_and_markdown PASSED     [ 81%]
tests/test_schemas.py::test_merge_report_schema_and_markdown PASSED      [ 84%]
tests/test_schemas.py::test_monitoring_alert_schema_and_markdown PASSED  [ 87%]
tests/test_state.py::test_initial_state_is_idle PASSED                   [ 90%]
tests/test_state.py::test_start_workflow_success_and_conflict PASSED     [ 93%]
tests/test_state.py::test_workflow_transitions_and_waiting PASSED        [ 96%]
tests/test_state.py::test_state_persistence_roundtrip PASSED             [100%]

============================== 32 passed in 4.15s ==============================
```

### Live Hook CLI Verification
Executed real-time predictions with live payloads:
- **Coding Request**: Injects instruction to invoke `@coding` (`coding-orchestrator`).
- **Question**: Injects `"Prompt Triage Directive: Do not make any changes."`
- **Monitoring**: Injects instruction to invoke `@monitoring` (`coding-monitoring`).
- **Awaiting Subagent Feedback**: Successfully routes user response to `coding-specify` via `send_message`.
