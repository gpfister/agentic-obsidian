# Implementation Plan: Dual-Mode Multi-Hop Relay & Anti-Direct-Coding Guardrails in `@coding` Orchestrator

## 1. Executive Summary

This plan operationalizes the architectural solutions defined in [`specification.md`](file:///Users/gpfister/.gemini/antigravity/brain/99b212d5-05a0-498b-921e-81cc1ab9c56b/specification.md) to eliminate two critical operational defects:
1. **60-Second Subagent Timeout & Duplicate Approval Gates**: Elevates [`@coding`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding/agent.md) from a Pure Subagent / standalone gatekeeper into a **Dual-Mode Orchestrator** supporting Multi-Hop Relay for both interactive artifact approval gates (`[ARTIFACT_APPROVAL_REQUEST]`) and user clarification questions (`[QUESTION_RELAY_REQUEST]`). When running under an upstream orchestrator (caller ID present), `@coding` relays requests upward without executing `write_file(RequestFeedback=True)` or `ask_question`, avoiding duplicate interactive UI modals and background timeout terminations.
2. **Workflow Bypass & Direct Source Code Mutation**: Enforces hard **Anti-Direct-Coding Guardrails** in `@coding`, restricting `write_file` strictly to gate rendering in standalone mode, restricting `run_command` to workspace inspection, and providing a deterministic **Pre-Clarified Fast-Path** that bypasses Phase 1 (`coding-clarify`) while strictly maintaining Phase 2 (`coding-specify`) and Phase 3 (`coding-plan`) gates before development.

---

## 2. Git Branching Strategy

Consistent branch naming across all three affected repositories:

| Repository | Path | Base Branch | Working Branch | Branch Purpose |
| :--- | :--- | :--- | :--- | :--- |
| `agentic-coding-plugin` | `agentic-coding-plugin/` | `master` | `feat/dual-mode-coding-orchestrator` | Core orchestrator agent, skill, rules, catalog, and pytest test suite |
| `agentic` | `./` (workspace root) | `master` | `feat/dual-mode-coding-orchestrator` | Workspace coordination rules, subagent tier classification, upstream invocation |
| `agentic-issue-investigation-plugin` | `agentic-issue-investigation-plugin/` | `master` | `feat/dual-mode-coding-orchestrator` | Standardized upstream orchestration prompt for remediation confirmation gate |

---

## 3. Local Deployment Assessment (Per Repository)

In accordance with Section 4 of the Planning Protocol, each affected repository was inspected for local deployment mechanisms (Dockerfiles, Docker Compose, `./scripts/deployment_local.sh`, Kubernetes/Helm manifests, Minikube setups, or Makefile local deployment targets):

| Repository | Inspected Deployment Files | Deployment Requirement | Coding-Deploy Action |
| :--- | :--- | :--- | :--- |
| `agentic-coding-plugin` | No `Dockerfile`, no `docker-compose.yml`, no `scripts/deployment_local.sh`. Only `scripts/post_install.sh` (uv sync virtualenv) and pytest test suite. | **Local Deployment: Not Required** | Skip `@coding-deploy`. Verification performed via automated unit test suite (`pytest`). |
| `agentic` (Workspace Root) | No containerization or local service manifests. Only `Makefile` for multi-repo synchronization. | **Local Deployment: Not Required** | Skip `@coding-deploy`. Verification performed via documentation cross-reference audit. |
| `agentic-issue-investigation-plugin` | No containerization or local deployment instructions. Python plugin with pytest tests. | **Local Deployment: Not Required** | Skip `@coding-deploy`. Verification performed via documentation consistency audit. |

> [!NOTE]
> Since no local containerized or long-running service deployments exist for these markdown and Python plugin configurations, Phase 5 (`coding-deploy`) will be marked as skipped for all three repositories, advancing directly to QA verification and code review.

---

## 4. Multi-Repository Work Breakdown & Execution Steps

### Repository 1: `agentic-coding-plugin` (Sub-Repository)

Branch: `feat/dual-mode-coding-orchestrator`

#### 4.1 Step 1.1: Git Branch Initialization
- **Action**: Check out branch `feat/dual-mode-coding-orchestrator` from `master` in `agentic-coding-plugin/`.
- **Command**:
  ```bash
  git -C agentic-coding-plugin checkout -b feat/dual-mode-coding-orchestrator
  ```

#### 4.2 Step 1.2: Update `@coding` Subagent Definition ([`agents/coding/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding/agent.md))
- **Objective**: Implement Dual-Mode execution logic, multi-hop relay rules, anti-direct-coding guardrails, tool restriction clauses, and pre-clarified fast-path instructions.
- **Modifications**:
  1. **Dual-Mode Orchestrator Role & Mode Detection**:
     - Document runtime caller check: inspect `<subagent_reminder>`, context prompt, and caller metadata.
     - Define **Subagent Mode** (`caller_id` present): Forward `[ARTIFACT_APPROVAL_REQUEST]` and `[QUESTION_RELAY_REQUEST]` upward to `caller_id` via `send_message`. **MUST NOT** invoke `write_file(RequestFeedback=True)` or `ask_question`. Halt turn immediately to await `[ARTIFACT_APPROVAL_RESPONSE]` / `[QUESTION_RELAY_RESPONSE]`.
     - Define **Standalone Mode** (no `caller_id`): Render interactive gate via `write_file(RequestFeedback=True)` as the sole and final tool call of that turn; prompt user directly via `ask_question`.
  2. **Lifecycle Flowchart Update**:
     - Update Mermaid diagram to reflect Dual-Mode branching at approval gates and clarification gates.
  3. **Hard Anti-Direct-Coding Guardrails**:
     - Add explicit negative constraints: `@coding` is strictly a meta-orchestrator and MUST NEVER write, edit, or delete application source code, configuration files, Makefiles, or test suites directly.
     - Add tool constraint clauses:
       * `write_file`: Strictly permitted ONLY for presenting interactive approval gates (`<appDataDir>/brain/<id>/*.md`) in Standalone Mode. Prohibited for workspace files.
       * `run_command`: Strictly restricted to workspace inspection/status queries (`git status`). Prohibited for build (`make`, `npm build`) and test execution (`pytest`).
       * Delegation imperative: All implementation delegated to `@coding-develop`; deployment to `@coding-deploy`; review to `@coding-review`; release/PR to `@coding-merge`.
  4. **Pre-Clarified Fast-Path**:
     - Explicitly specify that when invoked with a pre-clarified technical specification, approved architecture document, or approved issue assessment report:
       * Skip Phase 1 (`coding-clarify`).
       * Transition directly to Phase 2 (`coding-specify`) or Phase 3 (`coding-plan`).
       * Mandatory requirement: Never skip approval gates before dispatching `@coding-develop`.

#### 4.3 Step 1.3: Update Skill Instructions ([`skills/coding-orchestrator/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/coding-orchestrator/SKILL.md))
- **Objective**: Align skill runbook with Dual-Mode execution, multi-hop relay sequencing, anti-direct-coding constraints, and pre-clarified fast-path handling.
- **Modifications**:
  1. Add Section on Dual-Mode Orchestration & Multi-Hop Relay Architecture.
  2. Update workflow flowchart and phase handoff descriptions.
  3. Include anti-direct-coding negative constraints and tool boundary rules.
  4. Document Pre-Clarified Fast-Path execution sequence.

#### 4.4 Step 1.4: Update Plugin Rules ([`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/rules/AGENTS.md))
- **Objective**: Update Question Relay and Artifact Approval Relay standards in plugin rules.
- **Modifications**:
  1. **Section 6 (Question Relay Protocol)**:
     - Classify `@coding` as Dual-Mode Orchestrator.
     - Document Multi-Hop Question Relay: Subagent Mode forwards `[QUESTION_RELAY_REQUEST]` upward; Standalone Mode calls `ask_question` directly.
  2. **Section 7 (Artifact Approval Relay Protocol)**:
     - Document Multi-Hop Artifact Relay: In Subagent Mode, `@coding` intercepts `[ARTIFACT_APPROVAL_REQUEST]` from worker subagents, does NOT invoke `write_file(RequestFeedback=True)`, and forwards the envelope directly to `caller_id` via `send_message`.
     - Explicitly document single interactive UI gate rule at session root.

#### 4.5 Step 1.5: Update Central Catalog ([`agents/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/agent.md))
- **Objective**: Synchronize central catalog tables, subagent matrices, and protocol explanations.
- **Modifications**:
  1. Update Section 1.2 Tool Privilege Model table to denote `@coding`'s dual-mode status and constrained tool usage.
  2. Document multi-hop relay behavior across Stages 1, 2, 3, and 5.
  3. Document anti-direct-coding boundaries and pre-clarified fast-path.

#### 4.6 Step 1.6: Expand Automated Unit Test Suite ([`tests/test_plugin.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/tests/test_plugin.py))
- **Objective**: Add comprehensive unit tests verifying Dual-Mode classification, multi-hop relay, anti-direct-coding guardrails, and fast-path.
- **New Test Functions**:
  1. `test_coding_orchestrator_dual_mode_classification`:
     - Asserts `@coding` is classified as Dual-Mode Orchestrator across `rules/AGENTS.md`, `agents/coding/agent.md`, and `skills/coding-orchestrator/SKILL.md`.
  2. `test_coding_orchestrator_multi_hop_relay_contracts`:
     - Asserts multi-hop relay instructions for both `[ARTIFACT_APPROVAL_REQUEST]` and `[QUESTION_RELAY_REQUEST]` are present in `agents/coding/agent.md`, `skills/coding-orchestrator/SKILL.md`, and `rules/AGENTS.md`.
     - Asserts prohibition of `write_file(RequestFeedback=True)` and direct `ask_question` when `caller_id` is present.
  3. `test_coding_orchestrator_anti_direct_coding_guardrails`:
     - Asserts explicit negative constraints in `agents/coding/agent.md` and `skills/coding-orchestrator/SKILL.md` prohibiting direct editing of source code, configuration files, Makefiles, build scripts, or test suites.
     - Asserts tool restriction clauses for `write_file` (restricted to approval gates in standalone mode) and `run_command` (inspection only).
  4. `test_coding_orchestrator_pre_clarified_fast_path`:
     - Asserts fast-path instructions exist in `agents/coding/agent.md` and `skills/coding-orchestrator/SKILL.md` to skip Phase 1 (`coding-clarify`) and route directly to Phase 2/3 when invoked with pre-clarified requirements or approved issue assessments.

---

## 5. Repository 2: `agentic` (Workspace Root Repository)

Branch: `feat/dual-mode-coding-orchestrator`

#### 5.1 Step 2.1: Git Branch Initialization
- **Action**: Check out branch `feat/dual-mode-coding-orchestrator` from `master` in workspace root.
- **Command**:
  ```bash
  git checkout -b feat/dual-mode-coding-orchestrator
  ```

#### 5.2 Step 2.2: Update Workspace Coordination Architecture ([`AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/AGENTS.md))
- **Objective**: Synchronize workspace-level agent coordination guidelines with Dual-Mode Orchestrator and multi-hop relay.
- **Modifications**:
  1. **Section 2.3 (Plugin Ecosystem Catalog)**:
     - Update `agentic-coding-plugin` summary to reflect Dual-Mode Orchestrator supporting Multi-Hop Relay and hard anti-direct-coding guardrails.
  2. **Section 3.1 (Dynamic Companion Detection & Adaptation)**:
     - Update companion interaction notes to specify standardized upstream invocation format for `@coding`.
  3. **Section 3.2.4 (Pure Subagents vs. Dual-Mode Subagents)**:
     - Update classification table: move `@coding` into the **Dual-Mode Orchestrators & Agents** tier.
     - Document multi-hop relay rules for `@coding` in subagent mode vs. standalone mode.
  4. **Section 3.3.4 (Pure Subagents vs. Dual-Mode Subagents Matrix)**:
     - Update artifact approval matrix: categorize `@coding` under Dual-Mode with multi-hop relay rules.
  5. **Standardized Upstream Invocation Template**:
     - Add explicit template for upstream orchestrators and subagents (`@coding-investigate`, `/project-grooming`, root agents) when invoking `@coding`:
       ```text
       Orchestrate the 7-stage software development lifecycle to implement the following feature / correction based on the approved assessment: <details/path>. Do not implement directly; delegate all phases through worker subagents.
       ```

---

## 6. Repository 3: `agentic-issue-investigation-plugin` (Sub-Repository)

Branch: `feat/dual-mode-coding-orchestrator`

#### 6.1 Step 3.1: Git Branch Initialization
- **Action**: Check out branch `feat/dual-mode-coding-orchestrator` from `master` in `agentic-issue-investigation-plugin/`.
- **Command**:
  ```bash
  git -C agentic-issue-investigation-plugin checkout -b feat/dual-mode-coding-orchestrator
  ```

#### 6.2 Step 3.2: Update Remediation Confirmation Gate ([`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/rules/AGENTS.md))
- **Objective**: Enforce the standardized upstream invocation template when `@coding-investigate` hands off remediation to `@coding`.
- **Modifications**:
  - Update Section 2 Remediation Confirmation Gate:
    ```markdown
    - **Remediation Confirmation Gate**:
      - If `agentic-coding-plugin` is installed: ask the user whether they want to invoke `@coding` to work on the resolution (using Question Relay Protocol via `send_message` in subagent mode, or `ask_question` in standalone mode). If Yes, invoke `@coding` using the standardized orchestration prompt:
        ```text
        Orchestrate the 7-stage software development lifecycle to implement the following feature / correction based on the approved assessment: <details/path>. Do not implement directly; delegate all phases through worker subagents.
        ```
    ```

---

## 7. Verification & Testing Strategy

### 7.1 Automated Unit Tests
- Execute pytest in `agentic-coding-plugin`:
  ```bash
  agentic-coding-plugin/.venv/bin/pytest agentic-coding-plugin/tests/ -v
  ```
- **Acceptance Criterion**: All existing tests (17 passed) plus all new unit tests pass with 100% success.

### 7.2 Workspace Git Status & Diff Inspection
- Run `git status` and `git diff` across all 3 repositories to ensure clean, scoped changes:
  ```bash
  git status
  git -C agentic-coding-plugin status
  git -C agentic-issue-investigation-plugin status
  ```

### 7.3 Cross-Repository Consistency Audit
- Assert that the Dual-Mode Orchestrator terminology, multi-hop relay schemas (`[ARTIFACT_APPROVAL_REQUEST]`, `[ARTIFACT_APPROVAL_RESPONSE]`, `[QUESTION_RELAY_REQUEST]`, `[QUESTION_RELAY_RESPONSE]`), anti-direct-coding negative constraints, and invocation templates match verbatim across:
  - `agentic-coding-plugin/agents/coding/agent.md`
  - `agentic-coding-plugin/skills/coding-orchestrator/SKILL.md`
  - `agentic-coding-plugin/rules/AGENTS.md`
  - `agentic-coding-plugin/agents/agent.md`
  - `agentic-coding-plugin/tests/test_plugin.py`
  - `agentic/AGENTS.md`
  - `agentic-issue-investigation-plugin/rules/AGENTS.md`

---

## 8. Risk Analysis & Contingency Handling

| Risk / Failure Mode | Likelihood | Impact | Mitigation Strategy |
| :--- | :---: | :---: | :--- |
| **Existing Unit Tests Regression** | Low | High | Run pytest baseline before changes and re-run iteratively after each file edit. Preserve all existing test fixtures and helper functions. |
| **Schema Incompatibility in Multi-Hop Envelopes** | Low | High | Maintain exact JSON schema contracts for `[ARTIFACT_APPROVAL_REQUEST]` and `[QUESTION_RELAY_REQUEST]` without mutating envelope keys or required fields. |
| **Subagent Timeout during Upstream Relay** | Very Low | Critical | Subagent Mode explicitly instructs `@coding` to never call blocking tools (`write_file(RequestFeedback=True)` or `ask_question`), yielding immediately to the root turn. |
| **Accidental Tool Deprecation** | Low | Medium | Keep `write_file` in `@coding` tools list for standalone mode approval gates, but constrain its runtime instructions strictly to `<appDataDir>/brain/<id>/*.md`. |

---

## 9. Traceability Matrix

| Specification Requirement | Target Repository | Affected File(s) | Verification Step |
| :--- | :--- | :--- | :--- |
| **Dual-Mode Orchestrator & Mode Detection** | `agentic-coding-plugin`<br>`agentic` | `agents/coding/agent.md`<br>`skills/coding-orchestrator/SKILL.md`<br>`rules/AGENTS.md`<br>`AGENTS.md` | `test_coding_orchestrator_dual_mode_classification` in `test_plugin.py` |
| **Multi-Hop Relay Protocol** | `agentic-coding-plugin`<br>`agentic` | `agents/coding/agent.md`<br>`skills/coding-orchestrator/SKILL.md`<br>`rules/AGENTS.md`<br>`AGENTS.md` | `test_coding_orchestrator_multi_hop_relay_contracts` in `test_plugin.py` |
| **Anti-Direct-Coding Guardrails & Tool Boundaries** | `agentic-coding-plugin` | `agents/coding/agent.md`<br>`skills/coding-orchestrator/SKILL.md`<br>`agents/agent.md` | `test_coding_orchestrator_anti_direct_coding_guardrails` in `test_plugin.py` |
| **Pre-Clarified Fast-Path** | `agentic-coding-plugin` | `agents/coding/agent.md`<br>`skills/coding-orchestrator/SKILL.md` | `test_coding_orchestrator_pre_clarified_fast_path` in `test_plugin.py` |
| **Standardized Upstream Invocation Template** | `agentic`<br>`agentic-issue-investigation-plugin` | `AGENTS.md`<br>`agentic-issue-investigation-plugin/rules/AGENTS.md` | Cross-file string audit & pytest assertion |
