---
title: "Project Grooming Plugin & Skill (/project-grooming)"
date: "2026-11-01"
status: "Phase 5 Review Completed (Ready for Merge)"
phase: "Phase 5: Review / Pre-Merge (Passed)"
adr_id: "003"
audit_verdict: "PASS (117/117 Unit Tests Passing Across Plugins, 100% AC-1..AC-5 Met)"
tags:
  - feature
  - requirements
  - specification
  - plan
  - development
  - project-grooming
  - scaffolding
  - skills
  - plugins
  - review-completed
  - pre-merge-passed
affected_plugins:
  - agentic-project-grooming-plugin
  - agentic-triage-plugin
  - agentic-coding-plugin
  - agentic-document-plugin
  - agentic (root)
  - agentic-obsidian
---

# Feature: Project Grooming Plugin & Skill (`/project-grooming`)

> [!NOTE] Phase 5: Review Completed & Pre-Merge State
> - **Current Phase**: Phase 5: Review / Pre-Merge (Passed)
> - **Feature Status**: Code Audited, 100% Acceptance Criteria Met, 117/117 Unit Tests Passing Across All Plugins
> - **Architecture Decision**: Formalized in [[architecture/adr-003-project-grooming-plugin|ADR-003]]
> - **Target Plugin**: `agentic-project-grooming-plugin` (Standalone Git sub-repo, branch: `feature/project-grooming-plugin`, commit: [`2bbd11f`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/))
> - **Primary Skill**: `/project-grooming` ([`skills/project-grooming/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/skills/project-grooming/SKILL.md))
> - **Interactive Gate Artifact**: `Project_Request_Grooming.md` (`RequestFeedback=True`, sole tool call)
> - **Multi-Repo Plan**: [`plan.md`](file:///Users/gpfister/.gemini/antigravity/brain/401cb359-82fe-40de-a4e6-222c7a01c706/plan.md)
> - **Ecosystem Impact**: `agentic-triage-plugin` (branch: `feature/grooming-triage`, commit: [`c6fd560`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/)), root workspace (branch: `feature/add-project-grooming-plugin`, commit: [`d2e11c5`](file:///Users/gpfister/Development/Projects/gpfister/agentic/)), `agentic-coding-plugin`, `agentic-document-plugin`, and central knowledge vault.

---

## 1. Executive Summary & Objective

The **`agentic-project-grooming-plugin`** is a standalone Google Antigravity companion plugin that introduces structured, interactive project initialization, scope boundary elicitation, and requirements grooming to the `agentic` workspace.

Rather than relying on a separate subagent, the plugin provides the **`/project-grooming`** skill ([`skills/project-grooming/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/skills/project-grooming/SKILL.md)), running directly in the main agent context. Inspired by Matt Pocock's `/grill-me` interrogation pattern, the skill conducts single-question-at-a-time inquiries (`ask_question`) across seven core architectural dimensions before any repository scaffolding or code generation occurs.

Once interrogation converges, the skill authors a structured, interactive approval artifact (`Project_Request_Grooming.md`) rendered with Antigravity feedback buttons (`RequestFeedback=True`). Upon user approval, the skill orchestrates multi-repository creation, documentation vault setup, and workspace integration, delegating to `@coding` and `@coding-document` when companion plugins are present.

---

## 2. Specification: The 7 Core Requirements Categories

During the grill-me interrogation phase, `/project-grooming` systematically gathers requirements across seven essential dimensions:

```mermaid
mindmap
  root((Project Grooming<br/>7 Core Categories))
    1. Project Identification
      Name & Slug
      Short Pitch & Elevator Summary
      Target Domain & Scope Bounds
    2. Goals & Target Audience
      Primary Business Objectives
      Target User Personas
      Key Use Cases & Workflows
      Explicit Non-Goals
    3. Restrictions & Standards
      Language & Runtime Constraints
      Architectural Paradigms
      Regulatory / Security Compliance
      Quality & Testing Standards
    4. Repository Topology
      Single-Repo vs Multi-Repo
      Sub-Packages / Service Layout
      Directory Structure
    5. Licensing
      SPDX License Identifier
      Copyright Attribution
      Third-Party Dependencies Policy
    6. Documentation Requirements
      Obsidian Vault Structure
      ADR Conventions
      Plugin / Component Specifications
    7. Workspace Tooling
      Makefile Targets
      CI / CD Automation
      Linter & Formatter Toolchains
      Git Tracking & Branch Conventions
```

### 2.1 Category 1: Project Identification
- **Project Name & Slug**: Human-readable name and filesystem/package-safe kebab-case slug.
- **Short Pitch & Elevator Summary**: High-level description capturing problem and purpose in 1–2 sentences.
- **Target Domain & Boundaries**: Problem domain, operational context, and ecosystem boundaries.

### 2.2 Category 2: Main Goals & Target Audience
- **Primary Business & Technical Objectives**: Core deliverables and value propositions.
- **Target User Personas**: Intended end-users, developers, or system operators.
- **Key Use Cases**: Primary workflows, happy paths, and critical user journeys.
- **Explicit Non-Goals**: Out-of-scope features to guard against scope creep.

### 2.3 Category 3: Known Restrictions & Technical Standards
- **Language & Runtime Constraints**: Permitted languages, runtimes, package managers, and runtime constraints.
- **Architectural Paradigms**: Architectural patterns (e.g., Clean Architecture, Event-Driven, Microservices, Hexagonal).
- **Compliance & Security**: Regulatory guidelines, credential handling, sandboxing, or privacy constraints.
- **Quality & Testing Standards**: Mandatory test coverage, static analysis, linting rules, and CI gates.

### 2.4 Category 4: Repository Topology
- **Repository Strategy**: Single standalone repo vs multi-repo federation vs monorepo sub-packages.
- **Directory Layout**: Layout convention (e.g. `src/`, `tests/`, `docs/`, `scripts/`, `skills/`, `hooks/`).
- **Inter-Service Communication**: Protocol definitions (REST, gRPC, direct library calls, message brokers) if multi-component.

### 2.5 Category 5: Licensing & Governance
- **SPDX License Expression**: e.g. `Apache-2.0`, `MIT`, `GPL-3.0-only`, `Proprietary`.
- **Copyright Attribution**: Organization name, individual author, and copyright years.
- **Third-Party Dependency Policy**: Permitted open-source license compatibility rules.

### 2.6 Category 6: Documentation Requirements (Obsidian Vault)
- **Obsidian Vault Structure**: Integration with `<project>-obsidian/` or shared root vault (`agentic-obsidian/`).
- **Standard Notes**: Project overview (`projects/<project-name>.md`), feature specifications (`features/`), architecture overviews, and ADR conventions.
- **Templates & References**: Obsidian templates for sessions, ADRs, and component designs.

### 2.7 Category 7: Workspace Tooling & Automation
- **Build & Task Automation**: Root [`Makefile`](file:///Users/gpfister/Development/Projects/gpfister/agentic/Makefile) targets (`build`, `test`, `lint`, `sync`, `clean`).
- **Automation Scripts**: Helper scripts (`scripts/sync_sessions.py`, `scripts/worktree.py`, etc.).
- **Git & Worktree Conventions**: Branch topologies, worktree workflows, and remote tracking configurations.

---

## 3. Specification: Matt Pocock "Grill-Me" Interrogation Protocol

### 3.1 Conversational Mechanics
- **Single-Question Rule**: Only one question is asked per turn using the `ask_question` tool.
- **Multiple-Choice + Custom Input**: Provide 3–5 concrete, actionable options with a `(Recommended)` first choice, while Antigravity's UI automatically provides an open-ended "Other" text input option.
- **Adaptive Interrogation Flow**: Subsequent questions dynamically branch based on the user's previous answers.
- **Early Convergence Guard**: If the user's initial prompt already specifies certain categories (e.g. name, language, repo topology), the skill skips redundant questions and validates the assumptions.
- **No Premature Scaffolding**: No code files, directory structures, or configurations may be created during the interrogation loop.

### 3.2 Interrogation Sequence Flow

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant MainAgent as Active Agent (/project-grooming)
    participant UI as Antigravity UI

    User->>MainAgent: "I want to start a new project: Distributed Cache"
    MainAgent->>UI: ask_question("Category 1: Identification & Core Goals")
    UI-->>User: Present choices with (Recommended) option
    User->>MainAgent: Selects or types answer
    MainAgent->>UI: ask_question("Category 2: Repository Topology & Runtime")
    UI-->>User: Present choices
    User->>MainAgent: Selects or types answer
    Note over MainAgent: Continues single-question inquiries for categories 3-7...
    MainAgent->>UI: write_file("Project_Request_Grooming.md", RequestFeedback=True)
    Note over MainAgent: Sole & Final Tool Call of Turn!
    UI-->>User: Display interactive [Approve] / [Reject] buttons
```

---

## 4. Specification: Approval Gate Artifact (`Project_Request_Grooming.md`)

### 4.1 Artifact Location & Properties
- **Target Path**: `<appDataDir>/brain/<conversation-id>/Project_Request_Grooming.md`
- **Metadata**:
  ```python
  ArtifactMetadata = {
      "Summary": "Project Request Grooming specification and scaffolding plan for <project-name>",
      "UserFacing": True,
      "RequestFeedback": True,
  }
  ```
- **Sole and Final Tool Call Rule**:
  > [!IMPORTANT]
  > Calling `write_file` to author `Project_Request_Grooming.md` with `RequestFeedback=True` **must be the sole and final tool call of that interaction turn**. No secondary tool calls (e.g. `list_dir`, `run_command`, or further writes) may be batched or executed in the same turn. This guarantees that Antigravity's interactive approval gate UI renders properly.

### 4.2 Standard Artifact Schema
The artifact must follow this mandatory structure:
1. **Title & Executive Summary**: Project name, slug, overview, domain context.
2. **Goals, Personas & Scope Boundaries**: Business goals, target audience, explicit in-scope and non-goals.
3. **Known Restrictions & Technical Standards**: Languages, runtimes, frameworks, compliance, and coding rules.
4. **Repository Topology & Directory Layout**: Monorepo/multi-repo breakdown, directory tree mockups, component contracts.
5. **Licensing & Governance**: SPDX license, copyright holder, third-party dependency guidelines.
6. **Documentation & Obsidian Knowledge Base**: Vault directory, template schemas, ADR tracking.
7. **Workspace Tooling & Orchestration**: Makefile targets, build commands, test runner commands.
8. **Scaffolding Execution Plan**: Step-by-step actions to execute once approved.

### 4.3 Gate Decision Handling
- **`[GATE_APPROVED]`**: The user clicks "Approve" (or indicates approval). The agent proceeds immediately to Section 5 Scaffolding Delegation.
- **`[GATE_REJECTED]`**: The user clicks "Reject" or requests adjustments. The agent re-enters the single-question interrogation loop to clarify and revise the artifact.

---

## 5. Specification: Cross-Plugin Delegation & Fallback Protocols

```mermaid
flowchart TD
    GateApproved(["User Approves Project_Request_Grooming.md"]) --> CheckCoding{"agentic-coding-plugin<br/>Installed?"}
    
    %% Coding delegation
    CheckCoding -->|"Yes"| InvokeCoding["Invoke @coding<br/>(Full 6-phase lifecycle for repository & code scaffolding)"]
    CheckCoding -->|"No"| DirectScaffold["Execute Direct Scaffolding<br/>(Create dirs, git init, Makefile, boilerplate)"]
    
    %% Document delegation
    InvokeCoding --> CheckDoc{"agentic-document-plugin<br/>Installed?"}
    DirectScaffold --> CheckDoc
    
    CheckDoc -->|"Yes"| InvokeDoc["Invoke @coding-document<br/>(Scaffold &lt;project&gt;-obsidian vault & index specs)"]
    CheckDoc -->|"No"| DirectDoc["Create standard &lt;project&gt;-obsidian/<br/>and starter README.md directly"]
    
    InvokeDoc --> Complete(["Project Initialization Completed"])
    DirectDoc --> Complete
```

### 5.1 Delegation Matrix

| Companion Target | Condition | Protocol when Installed | Fallback when NOT Installed |
| :--- | :--- | :--- | :--- |
| **`agentic-triage-plugin`** | Prompt Triage | Intent classifier recognizes project initialization / grooming prompts and routes them dynamically to `/project-grooming <prompt>`. | User invokes `/project-grooming` skill directly via chat prompt. |
| **`agentic-coding-plugin`** | Scaffolding & Code | Invokes `@coding` to orchestrate repository creation, configuration scaffolding, and test verification across the 6-stage lifecycle (`@coding-clarify` through `@coding-merge`). | The skill directly generates directories, initializes git repositories, writes baseline `Makefile`, and scaffolds starter boilerplate. |
| **`agentic-document-plugin`** | Documentation Vault | Invokes `@coding-document` to initialize the project Obsidian vault (`<project>-obsidian/`), register templates, and index architecture notes. | The skill directly scaffolds a standard `<project>-obsidian/` directory structure with default templates and README. |

---

## 6. Specification: Dynamic Triage Routing (`agentic-triage-plugin`)

The dynamic triage system is extended to recognize project grooming intent:
- **Triage Hook**: `agentic-triage-plugin/hooks/dynamic_triage.py` (or router hook configuration).
- **Condition**: Only evaluated if `agentic-project-grooming-plugin` is installed in the workspace.
- **Routing Command**: `/project-grooming <user_prompt>`
- **Intent Triggers**: Keywords and phrasing such as *"create a new project"*, *"initialize a repository"*, *"groom project requirements"*, *"start a new service"*, *"scaffold application"*, *"architect a new project"*.

---

## 7. Specification: Workspace Integration Requirements

The plugin integration touches the root workspace configuration:
1. **Root [`Makefile`](file:///Users/gpfister/Development/Projects/gpfister/agentic/Makefile)**:
   - Add `agentic-project-grooming-plugin` to `SUB_REPOS` list.
   - Support `make clone`, `make pull`, `make push`, `make status`, `make diff` operations.
2. **Root [`.gitignore`](file:///Users/gpfister/Development/Projects/gpfister/agentic/.gitignore)**:
   - Add `agentic-project-grooming-plugin/` so the standalone repository is cleanly decoupled from root git tracking.
3. **Root [`AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/AGENTS.md)**:
   - Add `agentic-project-grooming-plugin` to Workspace Layout, Plugin Ecosystem Catalog, and Cross-Plugin Interoperability matrix.
4. **Root [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/README.md)**:
   - Add project grooming capability description and `/project-grooming` skill usage guidelines.

---

## 8. Specification: File & Repository Structure

The `agentic-project-grooming-plugin` repository layout:

```text
agentic-project-grooming-plugin/
├── plugin.json                              # Antigravity plugin manifest
├── rules/
│   └── AGENTS.md                            # Plugin-scoped rules (grill-me protocol, gate rules)
├── skills/
│   └── project-grooming/
│       ├── SKILL.md                         # Primary skill definition & runbook
│       ├── references/
│       │   ├── categories.md                # 7 core categories question bank & guidelines
│       │   └── artifact_template.md         # Template for Project_Request_Grooming.md
├── tests/
│   ├── test_skill_manifest.py               # Manifest and file structure validation
│   └── test_grooming_protocol.py            # Protocol and category coverage tests
├── README.md                                # Plugin documentation
└── LICENSE                                  # Workspace license (Apache-2.0)
```

---

## 9. Architecture Decision Record (ADR-003)

The architectural decisions governing this feature are formalized in [[architecture/adr-003-project-grooming-plugin|ADR-003]]:
1. **Skill-Only Execution Model**: Executes directly within the main agent context via progressive skill `/project-grooming` without spawning a separate subagent (`@coding-groom`), optimizing token efficiency and user interactivity.
2. **Matt Pocock "Grill-Me" Interrogation**: Strict one-question-at-a-time protocol using `ask_question` with multiple choices and freeform input, forbidding premature code or folder scaffolding.
3. **Interactive Approval Gate**: Mandatory `Project_Request_Grooming.md` artifact generated with `RequestFeedback=True` as the sole and final tool call of that interaction turn.
4. **Cross-Plugin Delegation & Fallback**: Scaffolding delegates to `@coding` and documentation vault setup delegates to `@coding-document` when installed, with deterministic self-contained fallbacks when absent.

---

## 10. Multi-Repository Work Breakdown & Git Topology

Implementation is partitioned across three isolated Git repositories on dedicated feature branches:

```mermaid
flowchart TD
    subgraph Repo1["agentic-project-grooming-plugin<br/>(Branch: feature/project-grooming-plugin)"]
        F1["plugin.json"]
        F2["rules/AGENTS.md"]
        F3["skills/project-grooming/SKILL.md"]
        F4["README.md & LICENSE"]
        F5[".gitignore"]
    end

    subgraph Repo2["agentic-triage-plugin<br/>(Branch: feature/grooming-triage)"]
        T1["hooks/prompt_triage.py"]
        T2["rules/AGENTS.md"]
        T3["tests/test_prompt_triage.py"]
    end

    subgraph Repo3["agentic (Root Workspace)<br/>(Branch: feature/add-project-grooming-plugin)"]
        M1["Makefile (REPOS list)"]
        M2[".gitignore"]
        M3["AGENTS.md"]
        M4["README.md"]
    end

    Repo1 -->|Registered in| Repo3
    Repo2 -->|Dynamic routing to| Repo1
```

### 10.1 Repository 1: `agentic-project-grooming-plugin` (New Git Sub-repo)
- **Git Branch**: `feature/project-grooming-plugin`
- **Work Items**:
  1. Initialize standalone Git repository with `git init`.
  2. Create [`.gitignore`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/.gitignore) and `LICENSE` (MIT).
  3. Author [`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/plugin.json) manifest.
  4. Author [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/rules/AGENTS.md) enforcing grill-me rules, single-tool-call approval gates, and delegation.
  5. Implement [`skills/project-grooming/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/skills/project-grooming/SKILL.md) covering Gap Analysis, 7-Category Grilling, Gate Artifact Authoring, and Delegation.
  6. Author [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/README.md) documentation.

### 10.2 Repository 2: `agentic-triage-plugin`
- **Git Branch**: `feature/grooming-triage`
- **Work Items**:
  1. Modify [`hooks/prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/hooks/prompt_triage.py):
     - Add `agentic-project-grooming-plugin` to `PRIMARY_PLUGINS`.
     - Register `grooming` response candidates, directives (`/project-grooming <prompt>`), and intent mappings in `build_dynamic_triage_spec`.
     - Add grooming keyword heuristics and disambiguation in `_heuristic_system1_triage`.
  2. Update [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/rules/AGENTS.md) documenting grooming classification.
  3. Update [`tests/test_prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/tests/test_prompt_triage.py) adding unit tests for grooming intent and plugin presence/absence checks.

### 10.3 Repository 3: `agentic` (Root Workspace)
- **Git Branch**: `feature/add-project-grooming-plugin`
- **Work Items**:
  1. Add `agentic-project-grooming-plugin` to `REPOS` list in [`Makefile`](file:///Users/gpfister/Development/Projects/gpfister/agentic/Makefile).
  2. Add `/agentic-project-grooming-plugin/` to root [`.gitignore`](file:///Users/gpfister/Development/Projects/gpfister/agentic/.gitignore).
  3. Update [`AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/AGENTS.md) layout, catalog, and interoperability matrix.
  4. Update [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/README.md) summary table.

---

## 11. Verification & Acceptance Criteria

| ID | Repository | Criteria | Verification Command |
| :--- | :--- | :--- | :--- |
| **AC-1** | `agentic-project-grooming-plugin` | Standalone git repository initialized with clean history and valid `plugin.json` | `python3 -m json.tool plugin.json` |
| **AC-2** | `agentic-project-grooming-plugin` | Skill `/project-grooming` implements all 4 steps: gap analysis, 7-category user grilling via `ask_question`, `Project_Request_Grooming.md` artifact approval gate, and delegation to `@coding` and `@coding-document` | Inspection of [`skills/project-grooming/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/skills/project-grooming/SKILL.md) |
| **AC-3** | `agentic-triage-plugin` | Dynamic evaluation detects `agentic-project-grooming-plugin`, classifying prompts into `grooming` intent with directive `/project-grooming <prompt>` | `uv run pytest tests/test_prompt_triage.py` |
| **AC-4** | `agentic-triage-plugin` | Full test suite passes without regressions (100% pass) | `uv run pytest` |
| **AC-5** | `agentic` (Root) | `agentic-project-grooming-plugin` is registered in `Makefile`, ignored in `.gitignore`, and documented in `AGENTS.md` & `README.md` | `make status` |

---

## 12. Phase 4: Development Execution & Implementation Progress

Phase 4 (Development) fanned out implementation across three isolated feature branches in parallel, implementing the complete skill lifecycle, dynamic triage routing, and workspace integration.

### 12.1 Repository 1: `agentic-project-grooming-plugin`
- **Repository Path**: [`/Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/)
- **Feature Branch**: `feature/project-grooming-plugin`
- **HEAD Commit**: [`2bbd11f`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/) (`2bbd11f5f1383fa248c32243805a2f60df5bbe9d`)
- **Commit Message**: `feat: initial commit for agentic-project-grooming-plugin`
- **Development Status**: **Success (8/8 automated unit tests passing, 100%)**
- **Files Created**:
  1. [`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/plugin.json): Antigravity plugin manifest defining `agentic-project-grooming-plugin` (v0.1.0), registering the `/project-grooming` skill and declaring workspace companion dependencies.
  2. [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/rules/AGENTS.md): Plugin-scoped operational rules enforcing read-only pre-grooming analysis, the Matt Pocock single-question-at-a-time interrogation protocol (`ask_question`), sole-and-final-tool-call rule for `Project_Request_Grooming.md` (`RequestFeedback=True`), and cross-plugin delegation contracts.
  3. [`skills/project-grooming/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/skills/project-grooming/SKILL.md): Comprehensive 4-step progressive runbook covering non-destructive repository gap analysis, 7-category interactive user grilling, interactive approval artifact authoring, and dynamic delegation to `@coding` and `@coding-document`.
  4. [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/README.md): Plugin documentation outlining purpose, skill invocation, interrogation flow, and companion interoperability.
  5. [`LICENSE`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/LICENSE): Standard MIT License.
  6. [`.gitignore`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/.gitignore): Git ignore rules for Python caches, virtual environments, `.pytest_cache/`, and test artifacts.
  7. [`pyproject.toml`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/pyproject.toml): Project build manifest configuring pytest and project metadata.
  8. [`agents/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/agents/agent.md): Subagent definition reference documenting skill execution within the main agent context.
  9. [`tests/test_plugin.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/tests/test_plugin.py): Automated test suite asserting plugin manifest validity, rule formatting, skill structure, and category completeness.
- **Verification Command & Output**:
  ```bash
  $ cd agentic-project-grooming-plugin && uv run pytest
  ============================= test session starts ==============================
  platform darwin -- Python 3.13.0, pytest-9.2.0, pluggy-1.7.0
  rootdir: /Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin
  configfile: pyproject.toml
  collected 8 items

  tests/test_plugin.py ........                                            [100%]

  ============================== 8 passed in 0.01s ===============================
  ```

---

### 12.2 Repository 2: `agentic-triage-plugin`
- **Repository Path**: [`/Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/)
- **Feature Branch**: `feature/grooming-triage`
- **HEAD Commit**: [`c6fd560`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/) (`c6fd5604a0e167a8bc1bbd3b3f422cbfc786a5f5`)
- **Commit Message**: `feat(triage): add project grooming intent routing and candidate detection`
- **Development Status**: **Success (61/61 automated unit tests passing, 100%)**
- **Files Modified**:
  1. [`hooks/prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/hooks/prompt_triage.py):
     - Added `agentic-project-grooming-plugin` to `PRIMARY_PLUGINS`.
     - Added `grooming` intent candidate in `build_dynamic_triage_spec` returning `/project-grooming <prompt>`.
     - Extended `_heuristic_system1_triage` with keywords and patterns for project initiation, grooming, and requirements elicitation.
     - Added presence check logic to omit `grooming` response candidates when `agentic-project-grooming-plugin` is not installed.
  2. [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/rules/AGENTS.md): Updated prompt triage standards documentation with grooming classification and `/project-grooming` routing rule.
  3. [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/README.md): Documented project grooming intent classification support.
  4. [`tests/test_prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/tests/test_prompt_triage.py): Added test cases verifying intent detection for project initiation, keyword heuristics, dynamic candidate pruning when grooming plugin is missing, and directive formatting.
  5. [`tests/test_plugin.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/tests/test_plugin.py): Verified manifest integrity and hook registrations.
- **Verification Command & Output**:
  ```bash
  $ cd agentic-triage-plugin && uv run pytest
  ============================= test session starts ==============================
  platform darwin -- Python 3.13.0, pytest-9.2.0, pluggy-1.7.0
  rootdir: /Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin
  configfile: pyproject.toml
  plugins: anyio-4.15.1
  collected 61 items

  tests/test_plugin.py ............                                        [ 19%]
  tests/test_prompt_triage.py ............................................ [ 91%]
  .....                                                                    [100%]

  ============================== 61 passed in 5.46s ==============================
  ```

---

### 12.3 Repository 3: `agentic` (Root Workspace)
- **Repository Path**: [`/Users/gpfister/Development/Projects/gpfister/agentic`](file:///Users/gpfister/Development/Projects/gpfister/agentic/)
- **Feature Branch**: `feature/add-project-grooming-plugin`
- **HEAD Commit**: [`d2e11c5`](file:///Users/gpfister/Development/Projects/gpfister/agentic/) (`d2e11c51e75008e87029cff39bac2ba85cd017d6`)
- **Commit Message**: `feat(workspace): register agentic-project-grooming-plugin across root workspace`
- **Development Status**: **Success (`make status` tracks all 8 sub-repos cleanly)**
- **Files Modified**:
  1. [`Makefile`](file:///Users/gpfister/Development/Projects/gpfister/agentic/Makefile): Added `agentic-project-grooming-plugin` to `SUB_REPOS` array, enabling synchronized `make sync`, `make pull`, `make push`, `make status`, and `make diff`.
  2. [`.gitignore`](file:///Users/gpfister/Development/Projects/gpfister/agentic/.gitignore): Added `agentic-project-grooming-plugin/` ignore rule, ensuring the standalone git repository is decoupled from root tracking.
  3. [`AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/AGENTS.md): Registered `agentic-project-grooming-plugin` in Workspace Layout tree, Plugin Ecosystem Catalog, and Cross-Plugin Interoperability Matrix.
  4. [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/README.md): Documented project grooming capability and `/project-grooming` skill invocation in the root ecosystem directory.
- **Verification Command & Output**:
  ```bash
  $ make status
  === Repository Status Overview ===
    agentic (root)               branch: feature/add-project-grooming-plugin [clean]
    agentic-coding-plugin        branch: master         [clean] (ahead: 2, behind: 0)
    agentic-document-plugin      branch: master         [clean] (ahead: 2, behind: 0)
    agentic-investigate-plugin   branch: master         [clean] (ahead: 2, behind: 0)
    agentic-monitor-plugin       branch: master         [clean] (ahead: 2, behind: 0)
    agentic-obsidian             branch: master         [clean] (ahead: 9, behind: 0)
    agentic-project-grooming-plugin branch: feature/project-grooming-plugin [clean]
    agentic-research-plugin      branch: master         [clean] (ahead: 2, behind: 0)
    agentic-triage-plugin        branch: feature/grooming-triage [clean]
  ```

---

## 13. Phase 5: Review & Acceptance Verification

Phase 5 (`@coding-review`) conducted an exhaustive quality audit across all deliverables, validating functional specifications, clean code standards, automated test execution, and acceptance criteria.

### 13.1 Acceptance Criteria Audit Matrix

| ID | Repository | Criteria | Verification Command | Review Verdict |
| :--- | :--- | :--- | :--- | :--- |
| **AC-1** | `agentic-project-grooming-plugin` | Standalone git repository initialized with clean history and valid `plugin.json` | `python3 -m json.tool plugin.json` | **VERIFIED (PASS)**: Valid JSON, manifest conforms to Antigravity standard. |
| **AC-2** | `agentic-project-grooming-plugin` | Skill `/project-grooming` implements all 4 steps: gap analysis, 7-category user grilling via `ask_question`, `Project_Request_Grooming.md` artifact approval gate, and delegation to `@coding` and `@coding-document` | Inspection of [`skills/project-grooming/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/skills/project-grooming/SKILL.md) & [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/rules/AGENTS.md) | **VERIFIED (PASS)**: All 4 steps and 7 core categories fully specified, single-question and sole-tool-call gate enforced. |
| **AC-3** | `agentic-triage-plugin` | Dynamic evaluation detects `agentic-project-grooming-plugin`, classifying prompts into `grooming` intent with directive `/project-grooming <prompt>` | `uv run pytest tests/test_prompt_triage.py` | **VERIFIED (PASS)**: Dynamic registration in `PRIMARY_PLUGINS`, keyword heuristics, candidate pruning when missing. |
| **AC-4** | `agentic-triage-plugin` | Full test suite passes without regressions (100% pass) | `uv run pytest` | **VERIFIED (PASS)**: 61/61 unit tests passing cleanly. |
| **AC-5** | `agentic` (Root) | `agentic-project-grooming-plugin` is registered in `Makefile`, ignored in `.gitignore`, and documented in `AGENTS.md` & `README.md` | `make status` | **VERIFIED (PASS)**: Tracked in `SUB_REPOS`, ignored in root git, documented in workspace catalog. |

### 13.2 Automated Test Execution Summary
- **Total Ecosystem Unit Tests**: **117/117 passing (100% pass rate)**:
  - `agentic-project-grooming-plugin`: 8/8 passed in 0.01s.
  - `agentic-triage-plugin`: 61/61 passed in 5.46s.
  - `agentic-coding-plugin`: 16/16 passed in 0.03s.
  - `agentic-document-plugin`: 9/9 passed in 0.01s.
  - `agentic-investigate-plugin`: 7/7 passed in 0.01s.
  - `agentic-monitor-plugin`: 7/7 passed in 0.01s.
  - `agentic-research-plugin`: 9/9 passed in 0.01s.

### 13.3 Clean Code & Architecture Review
- **Single Responsibility Principle**: Distinct separation between dynamic triage detection (`agentic-triage-plugin`), interactive elicitation runbooks (`agentic-project-grooming-plugin`), and workspace orchestration (`Makefile`).
- **Fail-Safe & Graceful Fallback**: Dynamic candidate pruning operates without crashing when companion plugins are missing; fallback scaffolding works standalone.
- **Link Hygiene**: All file links utilize clickable markdown URLs (`file://`) and Obsidian wikilinks (`[[...]]`).

### 13.4 Merge Confirmation Gate
- User explicitly approved merge execution via the interactive Phase 5 confirmation gate.

---

## 14. Next Steps: Phase 6 (Merge & Release)

With Phase 5 (Review) completed and verified:
1. Fast-forward merge `feature/project-grooming-plugin` -> `master` in `agentic-project-grooming-plugin`.
2. Fast-forward merge `feature/grooming-triage` -> `master` in `agentic-triage-plugin`.
3. Fast-forward merge `feature/add-project-grooming-plugin` -> `master` in `agentic` (root).
4. Update `agentic-obsidian/git-status.md` and repository tracking post-merge.
