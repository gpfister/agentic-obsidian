# Walkthrough: Laya System 1 FastMCP Server & Intelligent Lifecycle Hooks

We have integrated the **Laya System 1** decision engine and **FastMCP** into [`gp-coding-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin), complete with automated virtual environment provisioning (`uv sync`) and an intelligent `PreToolUse` lifecycle hook.

---

## Changes Summary

### 1. Environment & Post-Install Automation
- **[`pyproject.toml`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/pyproject.toml)**: Configured dependencies: `fastmcp>=2.0.0`, `laya>=0.4.15`, `pydantic>=2.0.0`, and `pytest>=8.0.0`.
- **[`scripts/post_install.sh`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/scripts/post_install.sh)**: Executable bash script that validates `uv` and executes `uv sync` to generate or update `.venv`.
- **[`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/plugin.json)**: Declared post-install script hook in the plugin manifest.

### 2. Decision Engine & FastMCP Server
- **[`servers/laya_system1/engine.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/servers/laya_system1/engine.py)**: Implements `LayaSystem1Engine` supporting `evaluate_tool_risk` (using `noul`), `evaluate_choices` (using `choice`), `fast_decision`, and `fast_choice`.
- **[`servers/laya_system1/server.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/servers/laya_system1/server.py)**: FastMCP server registering the 4 tools under the name `laya_system1`.
- **[`mcp_config.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/mcp_config.json)**: Configured stdio transport invoking `.venv/bin/python servers/laya_system1/server.py`.

### 3. PreToolUse Lifecycle Hook
- **[`hooks/pre_tool.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/hooks/pre_tool.py)**: Intercepts all tool execution requests:
  - **`ask_question`**: Uses `choice` to rank options. If top confidence $\ge 0.90$, auto-selects and returns `decision: "deny"` to bypass user interaction and report the selected answer. If $< 0.90$, re-ranks options and adds `(Recommended)` if confidence $\ge 0.75$ via `overwrite: {"questions": ...}`.
  - **Tool Risk Evaluation**: Uses `noul` to evaluate risk. Blocks (`decision: "deny"`) if confidence $\ge 0.95$, requests user approval (`decision: "ask"`) if $0.80 \le \text{conf} < 0.95$, and executes without approval (`decision: "allow"`) if $< 0.80$.
- **[`hooks.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/hooks.json)**: Configured `PreToolUse` hook running `.venv/bin/python hooks/pre_tool.py`.

### 4. Tests & Documentation
- **[`tests/test_laya_system1.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/tests/test_laya_system1.py)**: 11 automated unit and integration tests covering risk thresholds, choice routing, FastMCP tools, and hook stdin/stdout JSON contracts.
- **[`gp-coding-plugin/README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/gp-coding-plugin/README.md)**: Updated setup and usage guide.
- **[`obsidian-vault/plugins/gp-coding-plugin.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/obsidian-vault/plugins/gp-coding-plugin.md)**: Synced plugin architecture specifications in the Obsidian vault.

---

## Verification Results

### Automated Test Suite
Ran `.venv/bin/pytest tests/test_laya_system1.py -v`:
```text
============================= test session starts ==============================
collected 11 items                                                             

tests/test_laya_system1.py::test_post_install_script_exists_and_executable PASSED [  9%]
tests/test_laya_system1.py::test_risk_evaluation_blocking PASSED         [ 18%]
tests/test_laya_system1.py::test_risk_evaluation_approval PASSED         [ 27%]
tests/test_laya_system1.py::test_risk_evaluation_safe_command PASSED     [ 36%]
tests/test_laya_system1.py::test_risk_evaluation_read_only_tool PASSED   [ 45%]
tests/test_laya_system1.py::test_choice_evaluation_single_option PASSED  [ 54%]
tests/test_laya_system1.py::test_choice_evaluation_decisive PASSED       [ 63%]
tests/test_laya_system1.py::test_fastmcp_tools_registration PASSED       [ 72%]
tests/test_laya_system1.py::test_hook_ask_question_auto_pick PASSED      [ 81%]
tests/test_laya_system1.py::test_hook_ask_question_re_ranking PASSED     [ 90%]
tests/test_laya_system1.py::test_hook_destructive_command_blocking PASSED [100%]

======================== 11 passed, 1 warning in 13.35s ========================
```

### Plugin Validation
Ran `agy plugin validate gp-coding-plugin --json`:
```json
{
  "dir": "gp-coding-plugin",
  "manifest": "plugin.json",
  "name": "gp-coding-plugin",
  "agentPluginsSpec": false,
  "components": {
    "agents": 0,
    "hooks": 1,
    "mcpServers": 1,
    "rules": 1,
    "sidecars": 0,
    "skills": 1
  },
  "findings": []
}
```
All components (1 hook, 1 MCP server, 1 rule set, 1 skill) detected cleanly with zero errors.
