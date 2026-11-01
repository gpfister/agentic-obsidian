---
title: "Project Grooming Plugin & Skill (/project-grooming)"
date: "2026-11-01"
status: "Phase 3 Plan Approved"
phase: "Phase 3: Planning"
adr_id: "003"
tags:
  - feature
  - requirements
  - specification
  - plan
  - project-grooming
  - scaffolding
  - skills
  - plugins
  - phase-3-approved
  - plan-approved
affected_plugins:
  - agentic-project-grooming-plugin
  - agentic-triage-plugin
  - agentic-coding-plugin
  - agentic-document-plugin
  - agentic (root)
  - agentic-obsidian
---

# Feature: Project Grooming Plugin & Skill (`/project-grooming`)

> [!NOTE] Phase 3: Planning Approved
> - **Current Phase**: Phase 3: Planning (Approved)
> - **Feature Status**: Plan Approved & Ready for Development (Phase 4)
> - **Architecture Decision**: Formalized in [[architecture/adr-003-project-grooming-plugin|ADR-003]]
> - **Target Plugin**: `agentic-project-grooming-plugin` (Standalone Git sub-repo, branch: `feature/project-grooming-plugin`)
> - **Primary Skill**: `/project-grooming` ([`skills/project-grooming/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/skills/project-grooming/SKILL.md))
> - **Interactive Gate Artifact**: `Project_Request_Grooming.md` (`RequestFeedback=True`, sole tool call)
> - **Multi-Repo Plan**: [`plan.md`](file:///Users/gpfister/.gemini/antigravity/brain/401cb359-82fe-40de-a4e6-222c7a01c706/plan.md)
> - **Ecosystem Impact**: `agentic-triage-plugin` (branch: `feature/grooming-triage`), root workspace (branch: `feature/add-project-grooming-plugin`), `agentic-coding-plugin`, `agentic-document-plugin`, and central knowledge vault.

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

## 12. Next Steps

With Phase 3 (Planning) approved and formalized:
- **Phase 4: Development (`@coding-develop`)**:
  - Fan out development per repository on isolated branches.
  - Implement files and tests across all 3 repositories.
- **Phase 5: Review (`@coding-review`)**:
  - Audit deliverables against specification and acceptance criteria.
  - Run verification test suites.
- **Phase 6: Merge & Release (`@coding-merge`)**:
  - Merge feature branches into `master` across all repositories.
  - Update vault git statuses and release changelogs.
