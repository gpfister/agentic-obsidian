---
title: "Dual-Mode Coding Orchestrator with Multi-Hop Relay and Anti-Direct-Coding Guardrails Architecture & Implementation Plan"
date: "2026-11-06"
status: "Released / Completed"
phase: "Phase 7: Merge & Release Complete"
specification_id: "SPEC-2026-DUAL-MODE-001"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/99b212d5-05a0-498b-921e-81cc1ab9c56b/specification.md"
vault_specification_path: "sessions/attachments/99b212d5-05a0-498b-921e-81cc1ab9c56b/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/ae4bcb14-d756-42f4-aeea-52cc919954cf/plan.md"
vault_plan_path: "sessions/attachments/ae4bcb14-d756-42f4-aeea-52cc919954cf/plan.md"
adr: "[[architecture/adr-008-dual-mode-coding-orchestrator|ADR-008: Dual-Mode Coding Orchestrator with Multi-Hop Relay and Anti-Direct-Coding Guardrails]]"
tags:
  - architecture
  - implementation-plan
  - dual-mode
  - multi-hop-relay
  - anti-direct-coding
  - guardrails
  - coding-orchestrator
  - timeout-mitigation
  - subagent-protocol
  - approved-plan
  - phase-3-completed
  - phase-4-completed
  - phase-5-completed
  - phase-6-completed
  - phase-7-completed
  - released
  - completed
  - post-merge-synchronized
affected_plugins:
  - agentic-coding-plugin
  - agentic (workspace root)
  - agentic-issue-investigation-plugin
branch: "master"
aliases:
  - dual-mode-coding-orchestrator
  - dual-mode-coding-orchestrator-plan
---

# Dual-Mode Coding Orchestrator with Multi-Hop Relay and Anti-Direct-Coding Guardrails Architecture & Implementation Plan

> [!NOTE] Implementation Milestone: Phase 7 (Merge & Release Complete)
> - **Specification ID**: `SPEC-2026-DUAL-MODE-001`
> - **Architecture Decision Record**: [[architecture/adr-008-dual-mode-coding-orchestrator|ADR-008: Dual-Mode Coding Orchestrator with Multi-Hop Relay and Anti-Direct-Coding Guardrails]]
> - **Feature Specification Note**: [[features/dual-mode-coding-orchestrator|Feature: Dual-Mode Coding Orchestrator with Multi-Hop Relay and Anti-Direct-Coding Guardrails]]
> - **Target Branch**: `master` (Merged and synchronized across all repositories)
> - **Current Phase**: Phase 7: Merge & Release Complete
> - **Merge Commits**:
>   - `agentic`: PR #2 merged into `master` ([`fa5164a`](file:///Users/gpfister/Development/Projects/gpfister/agentic/))
>   - `agentic-coding-plugin`: PR #2 merged into `master` ([`3e04ac1`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/)), **21/21 tests passed (100%)**
>   - `agentic-issue-investigation-plugin`: PR #1 merged into `master` ([`34629b3`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/)), **14/14 tests passed (100%)**
> - **Verification Verdict**: **PASS (35/35 tests passing across feature repos, 100% pass rate)**

---

## 1. Problem Formulation & Root Causes

Operational production traces revealed two severe vulnerabilities when [`@coding`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding/agent.md) is executed:

### 1.1 Root Cause A: 60-Second Subagent Timeout & Duplicate Gate Presentation
During multi-agent handoffs (e.g., when the session root agent spawns `@coding` to resolve an issue diagnosed by `@coding-investigate`), `@coding` was classified as a top-level orchestrator. Consequently:
- Upon receiving `[ARTIFACT_APPROVAL_REQUEST]` from `@coding-specify` or `@coding-plan`, `@coding` called `write_file(RequestFeedback=True)` in its background turn.
- Meanwhile, the session root agent also intercepted the subagent idle status and rendered `write_file(RequestFeedback=True)` at the root.
- This produced duplicate interactive modals in Antigravity's UI.
- Furthermore, `@coding` called `ask_question` directly during clarification, triggering the host runtime's rigid 60-second execution timeout when user review exceeded one minute.

### 1.2 Root Cause B: Workflow Bypass & Direct Code Mutation
When `@coding` received an imperative prompt with technical guidance (such as an approved remediation plan), the LLM took the path of least resistance:
- Bypassed all 7 lifecycle phases (`coding-clarify`, `coding-specify`, `coding-plan`, `coding-develop`, `coding-deploy`, `coding-review`, `coding-merge`).
- Used its declared `write_file` and `run_command` tools to directly edit application code files and run tests.
- Omitted clean code reviews, test gates, PR creation, git branch management, and documentation updates.

---

## 2. Architectural Design: Dual-Mode & Multi-Hop Relay

```mermaid
flowchart TD
    Start["Invocation of @coding"] --> ModeCheck{"caller_id present in context?<br/>(Check &lt;subagent_reminder&gt;)"}
    
    ModeCheck -->|Yes: Subagent Mode| SubagentMode["Operate in Subagent Mode<br/>(Child Orchestrator)"]
    ModeCheck -->|No: Standalone Mode| StandaloneMode["Operate in Standalone Mode<br/>(Root Session Orchestrator)"]
    
    subgraph SubagentModeExecution["Subagent Mode Protocol"]
        SubArtifact["Receive [ARTIFACT_APPROVAL_REQUEST]"] --> RelayArtifact["Forward envelope upward to caller_id via send_message<br/>DO NOT call write_file(RequestFeedback=True)<br/>Terminate turn immediately"]
        RelayArtifact --> AwaitArtifact["Await [ARTIFACT_APPROVAL_RESPONSE]<br/>(Reactive Wakeup)"]
        AwaitArtifact --> ForwardArtifactDown["Forward response to worker subagent<br/>Advance phase"]
        
        SubQuestion["Receive [QUESTION_RELAY_REQUEST]"] --> RelayQuestion["Forward envelope upward to caller_id via send_message<br/>DO NOT call ask_question<br/>Terminate turn immediately"]
        RelayQuestion --> AwaitQuestion["Await [QUESTION_RELAY_RESPONSE]<br/>(Reactive Wakeup)"]
        AwaitQuestion --> ForwardQuestionDown["Forward answer to worker subagent"]
    end
    
    subgraph StandaloneModeExecution["Standalone Mode Protocol"]
        StandArtifact["Receive [ARTIFACT_APPROVAL_REQUEST]"] --> RenderGate["Call write_file(RequestFeedback=True)<br/>Sole & Final Tool Call of Turn"]
        RenderGate --> StandAwaitGate["Await interactive User Click<br/>(No 60-second timeout)"]
        StandAwaitGate --> StandSendArtifactResp["Send [ARTIFACT_APPROVAL_RESPONSE] to worker subagent"]
        
        StandQuestion["Receive [QUESTION_RELAY_REQUEST]"] --> AskUser["Call ask_question directly<br/>(Root User Interaction)"]
        AskUser --> StandSendQuestionResp["Send [QUESTION_RELAY_RESPONSE] to worker subagent"]
    end
    
    SubagentMode --> SubagentModeExecution
    StandaloneMode --> StandaloneModeExecution
```

### 2.1 Multi-Hop Relay Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    actor User as Human User
    participant Root as Root Main Agent (Session Root)
    participant Coding as @coding Orchestrator (Subagent Mode)
    participant Worker as Worker Subagent (@coding-specify / @coding-plan)

    Note over Coding,Worker: Phase 2 / Phase 3 Lifecycle Execution
    Worker->>Worker: write_file(artifact_path, Content, RequestFeedback=False)
    Worker->>Coding: send_message([ARTIFACT_APPROVAL_REQUEST])
    Note over Worker: Enters IDLE state

    Note over Coding: Subagent Mode detected (caller_id present)<br/>MUST NOT call write_file(RequestFeedback=True)
    Coding->>Root: send_message([ARTIFACT_APPROVAL_REQUEST])
    Note over Coding: Enters IDLE state

    Note over Root: Root Turn: Presents Single Interactive Gate
    Root->>User: write_file(artifact_path, Content, RequestFeedback=True)<br/>[Sole & Final Tool Call of Turn]
    Note over User: User reviews document (No 60s timeout)
    User-->>Root: Approve / Reject Decision

    Root->>Coding: send_message([ARTIFACT_APPROVAL_RESPONSE])
    Note over Coding: Reactive wakeup, intercepts response
    Coding->>Worker: send_message([ARTIFACT_APPROVAL_RESPONSE])
    Note over Worker: Reactive wakeup, advances to next phase
```

---

## 3. Subagent Classification Matrix

| Tier | Agents | Operating Protocol | Tooling & Action Permissions |
| :--- | :--- | :--- | :--- |
| **Pure Subagents** | `@coding-clarify`<br>`@coding-specify`<br>`@coding-plan`<br>`@coding-develop`<br>`@coding-deploy`<br>`@coding-review`<br>`@coding-merge` | Strictly run under `@coding`. **NEVER** call `ask_question` or `write_file(RequestFeedback=True)`. Always dispatch `[QUESTION_RELAY_REQUEST]` and `[ARTIFACT_APPROVAL_REQUEST]` to `caller_id`. | `send_message`, `write_file` (without `RequestFeedback=True`), specialized domain tools. |
| **Dual-Mode Orchestrators & Agents** | **`@coding`**<br>`@coding-investigate`<br>`/project-grooming` (`@coding-groom`)<br>`@coding-monitor`<br>`@coding-research` | **Subagent Mode (`caller_id` present)**:<br>• Relay `[ARTIFACT_APPROVAL_REQUEST]` up to `caller_id` via `send_message`.<br>• Relay `[QUESTION_RELAY_REQUEST]` up to `caller_id` via `send_message`.<br>• Forward responses down to requesting worker subagent.<br>**Standalone Mode (no `caller_id`)**:<br>• Call `write_file(RequestFeedback=True)` directly as sole and final tool call of turn.<br>• Call `ask_question` directly. | `send_message`, `invoke_subagent`, `ask_question`, `write_file` (strictly for gates in standalone mode), inspection tools (`git status`). |

---

## 4. Work Breakdown by Repository

### Repository 1: `agentic-coding-plugin` (Sub-Repository)
- **Branch**: `feat/dual-mode-coding-orchestrator`
- **Affected Files**:
  1. [`agents/coding/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding/agent.md): Add Dual-Mode mode detection, multi-hop relay rules, anti-direct-coding negative constraints, tool boundary rules, and pre-clarified fast-path instructions.
  2. [`skills/coding-orchestrator/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/coding-orchestrator/SKILL.md): Update skill instructions with multi-hop relay execution, tool constraints, and fast-path runbooks.
  3. [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/rules/AGENTS.md): Update Section 6 and Section 7 protocols to categorize `@coding` under Dual-Mode.
  4. [`agents/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/agent.md): Synchronize catalog tables and subagent privilege matrices.
  5. [`tests/test_plugin.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/tests/test_plugin.py): Add unit tests verifying Dual-Mode classification, multi-hop relay contracts, anti-direct-coding constraints, and pre-clarified fast-path.

### Repository 2: `agentic` (Workspace Root Repository)
- **Branch**: `feat/dual-mode-coding-orchestrator`
- **Affected Files**:
  1. [`AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/AGENTS.md): Update Section 2.3, Section 3.1, Section 3.2.4, Section 3.3.4, subagent categorization tables, and standardized upstream invocation template.

### Repository 3: `agentic-issue-investigation-plugin` (Sub-Repository)
- **Branch**: `feat/dual-mode-coding-orchestrator`
- **Affected Files**:
  1. [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/rules/AGENTS.md): Update Section 2 Remediation Confirmation Gate to format upstream handoffs using the standardized orchestration template.

---

## 5. Verification Strategy & Outcomes

1. **Automated Test Suite**:
   - `agentic-coding-plugin`: Executed `pytest agentic-coding-plugin/tests/test_plugin.py` -> **21/21 passed (100%)** (all 17 baseline tests + 4 new tests asserting dual-mode classification, multi-hop relay message handling, anti-direct-coding constraints, and pre-clarified fast-path).
   - `agentic-issue-investigation-plugin`: Executed `pytest agentic-issue-investigation-plugin/tests/test_plugin.py` -> **14/14 passed (100%)** (all 13 baseline tests + 1 new test asserting standardized orchestration prompt template).
2. **Quality Audit & Review (Phase 5)**:
   - Clean Code Reviewer audit performed and approved with zero architectural regressions.
   - Dual-Mode orchestrator protocol validated across all caller-subagent topologies.
3. **Pre-Merge Documentation Tidy-Up (Phase 6)**:
   - Synchronized all READMEs, feature notes, architecture plans, ADR-008, plugin specs, and git status tracking records across the workspace.
4. **Merge & Release Execution (Phase 7)**:
   - PR #2 in `agentic` merged into `master` (commit `fa5164a`).
   - PR #2 in `agentic-coding-plugin` merged into `master` (commit `3e04ac1`).
   - PR #1 in `agentic-issue-investigation-plugin` merged into `master` (commit `34629b3`).
   - Upstream changes pulled into local master branches; feature branches deleted; working trees clean. Workspace status, release changelog, and feature specs updated to Released / Completed.
