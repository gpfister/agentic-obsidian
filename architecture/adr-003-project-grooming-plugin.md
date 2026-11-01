---
adr_id: "003"
title: "Project Grooming Architecture, Skill-Only Execution, Grilling Protocol, Interactive Approval Gate, and Cross-Plugin Delegation"
status: "accepted"
date: "2026-11-01"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/401cb359-82fe-40de-a4e6-222c7a01c706/plan.md"
specification_path: "/Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/features/project-grooming-plugin.md"
tags:
  - architecture-decision
  - project-grooming
  - skills
  - approval-gate
  - multi-repo
  - delegation
---

# ADR-003: Project Grooming Architecture, Skill-Only Execution, Grilling Protocol, Interactive Approval Gate, and Cross-Plugin Delegation

## Context
Across the `agentic` workspace and software development ecosystems, initiating a new project, service, or Antigravity plugin often suffers from premature scaffolding: code, directory hierarchies, and build files are generated before core architectural boundaries, licensing requirements, testing standards, and repository topologies are clarified. This causes subsequent scope drift, fragmented tooling, and costly rework.

While the workspace provides specialized subagents for development (`@coding`), root-cause investigation (`@coding-investigate`), telemetry surveillance (`@coding-monitor`), and topic research (`@coding-research`), it lacked a dedicated mechanism for upfront project requirements elicitation, architectural boundary grooming, and workspace initialization.

To solve this, we require a repeatable, structured framework that:
1. Conducts thorough architectural gap analysis and requirements gathering before repository creation.
2. Interrogates users systematically across 7 essential project dimensions without overwhelming them.
3. Enforces an explicit, user-facing interactive approval gate before any code or repository generation.
4. Seamlessly delegates code generation and documentation vault setup to existing specialized companions (`@coding` and `@coding-document`) while maintaining standalone fallback behavior when those plugins are not installed.

---

## Decision

We establish the following architectural decisions for project initialization and grooming across the workspace:

### 1. Skill-Only Execution Model in Main Agent Context
- The project grooming capability is implemented as a standalone Antigravity companion plugin, **`agentic-project-grooming-plugin`**, exposing the on-demand skill **`/project-grooming`** ([`skills/project-grooming/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/skills/project-grooming/SKILL.md)).
- **No Dedicated Subagent**: Unlike long-running workflows that delegate to autonomous subagents (e.g. `@coding` or `@coding-investigate`), project grooming operates directly within the primary agent session as a progressive skill.
- **Rationale**: An interactive interview protocol requires tight back-and-forth user interaction. Spawning an isolated subagent worker introduces unnecessary process boundaries, subagent context switching, and token amplification for simple question-and-answer turns.

### 2. Strict One-Question-at-a-Time "Grill-Me" Interrogation Protocol
- Inspired by Matt Pocock's `/grill-me` protocol, the skill executes a disciplined, sequential interrogation loop using `ask_question`.
- **Mandatory Invariants**:
  - **Single Question Rule**: Exactly one question is asked per interaction turn. Asking multiple questions or unformatted bulleted inquiries in chat text is strictly forbidden.
  - **Structured Choices with Open Input**: Questions must provide 3–5 concrete multiple-choice options with a recommended answer first (prefixed with `(Recommended)`), allowing Antigravity to render an open-ended "Other" text input for custom answers.
  - **The 7 Core Categories**: Interrogation systematically covers:
    1. *Project Identification* (Name, slug, pitch, problem domain)
    2. *Goals & Target Audience* (Core goals, user personas, key use cases, explicit non-goals)
    3. *Known Restrictions & Standards* (Languages, runtimes, frameworks, compliance, testing standards)
    4. *Repository Topology* (Monorepo vs multi-repo, directory layout, service contracts)
    5. *Licensing & Governance* (SPDX license expression, copyright holder, dependencies policy)
    6. *Documentation Requirements* (Obsidian vault setup, ADR conventions, component specs)
    7. *Workspace Tooling & Orchestration* (Makefile targets, Git worktrees, session sync, CI/CD linters)
  - **Early Convergence Guard**: If the user's initial prompt or repository inspection already clarifies certain categories, the skill skips redundant questions and validates assumptions.
  - **Zero Premature Scaffolding**: No directories, git branches, or boilerplate files may be created during the interrogation loop.

### 3. Interactive Approval Gate via `Project_Request_Grooming.md`
- Once interrogation converges, the agent synthesizes a comprehensive specification and scaffolding blueprint artifact named `Project_Request_Grooming.md`.
- **Artifact Configuration**:
  - Target path: `<appDataDir>/brain/<conversation-id>/Project_Request_Grooming.md`
  - Metadata: `RequestFeedback=True`, `UserFacing=True`.
- **Sole and Final Tool Call Rule**:
  > [!IMPORTANT]
  > Calling `write_file` with `Project_Request_Grooming.md` and `RequestFeedback=True` must be the sole and final tool call of that interaction turn. No secondary tool calls (e.g. `list_dir`, `run_command`, `git status`) may be batched or executed in the same turn, ensuring Antigravity renders the interactive **Approve** and **Reject** buttons cleanly.
- **Gate Handling**:
  - `[GATE_APPROVED]`: Execution proceeds directly to cross-plugin delegation and repository scaffolding.
  - `[GATE_REJECTED]`: Execution re-enters the interrogation loop to clarify scope and revise the blueprint.

### 4. Multi-Repo Scaffolding & Documentation Delegation with Graceful Fallback
- Scaffolding responsibilities are partitioned according to companion plugin availability:
  - **Coding Delegation**:
    - If `agentic-coding-plugin` is installed: Invoke `@coding` (`invoke_subagent(TypeName="coding", Role="Coding Workflow Orchestrator")`) to drive the 6-stage software development lifecycle (`@coding-clarify` through `@coding-merge`) for repository initialization, code generation, linting, and automated testing.
    - Fallback: If `agentic-coding-plugin` is NOT installed, the `/project-grooming` skill directly generates baseline directories, initializes Git repos, and writes starter boilerplate.
  - **Documentation Delegation**:
    - If `agentic-document-plugin` is installed: Invoke `@coding-document` (`invoke_subagent(TypeName="coding-document", Role="Documentation Specialist")`) to initialize the `<project>-obsidian/` knowledge base, configure templates, and record architecture overviews and ADRs.
    - Fallback: If `agentic-document-plugin` is NOT installed, directly scaffold `<project>-obsidian/` directory structure with standard templates and a starter README.
  - **Dynamic Prompt Triage**:
    - When `agentic-project-grooming-plugin` is installed, `agentic-triage-plugin` detects it in `PRIMARY_PLUGINS`, registers dynamic response candidate directives, and routes project initiation prompts to `/project-grooming <prompt>`.

### 5. Multi-Repository Work Breakdown & Git Branch Topology
Implementation spans three isolated git repositories with dedicated feature branches:
1. **`agentic-project-grooming-plugin`** (New Standalone Git Sub-repository):
   - Branch: `feature/project-grooming-plugin`
   - Files: [`plugin.json`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/plugin.json), [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/rules/AGENTS.md), [`skills/project-grooming/SKILL.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/skills/project-grooming/SKILL.md), [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/README.md), [`.gitignore`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/.gitignore), `LICENSE` (MIT).
2. **`agentic-triage-plugin`** (Dynamic Triage Sub-repository):
   - Branch: `feature/grooming-triage`
   - Files: [`hooks/prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/hooks/prompt_triage.py), [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/rules/AGENTS.md), [`tests/test_prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-triage-plugin/tests/test_prompt_triage.py).
3. **`agentic`** (Root Workspace Repository):
   - Branch: `feature/add-project-grooming-plugin`
   - Files: [`Makefile`](file:///Users/gpfister/Development/Projects/gpfister/agentic/Makefile) (add to `REPOS`), [`.gitignore`](file:///Users/gpfister/Development/Projects/gpfister/agentic/.gitignore), [`AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/AGENTS.md), [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/README.md).

---

## Consequences

### Positive
- **Architectural Clarity Upfront**: Projects are scaffolded with explicit boundaries, verified licensing, defined personas, and agreed-upon topologies before any source code is generated.
- **Lightweight User Experience**: Executing as a skill in the main agent avoids the latency, prompt overhead, and mental disconnect of switching to a subagent for basic clarifying questions.
- **Deterministic Quality Gates**: The mandatory interactive approval artifact (`Project_Request_Grooming.md`) provides human-in-the-loop sign-off before executing irreversible file creation.
- **Ecosystem Synergy**: Scaffolding delegates cleanly to `@coding` and `@coding-document`, reinforcing the multi-plugin federated architecture.
- **Resilience**: Complete fallback behaviors ensure standalone functionality if companion plugins are absent.

### Negative & Trade-offs
- Multiple sequential `ask_question` turns require patient user engagement (typically 3–7 conversational turns).
- Root workspace configurations must track an additional sub-repository.

### Risks & Mitigations
- *Risk*: User frustration with overly lengthy questioning.
  *Mitigation*: The skill performs upfront directory and prompt gap analysis, skipping already-specified categories and asking only high-signal differentiating questions.
- *Risk*: Premature execution if artifact tool call is batched with secondary commands.
  *Mitigation*: Strict single-tool-call rule enforced in `rules/AGENTS.md` and `SKILL.md`.

---

## Implementation & Verification Status

- **Phase 1: Clarification**: Approved (Requirements gathered across 7 core dimensions).
- **Phase 2: Specification**: Approved ([`features/project-grooming-plugin.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-obsidian/features/project-grooming-plugin.md)).
- **Phase 3: Planning**: Approved ([`plan.md`](file:///Users/gpfister/.gemini/antigravity/brain/401cb359-82fe-40de-a4e6-222c7a01c706/plan.md)).
- **Phase 4: Development**: Ready for execution on branches `feature/project-grooming-plugin`, `feature/grooming-triage`, and `feature/add-project-grooming-plugin`.

---

## References
- Approved Implementation Plan: [`plan.md`](file:///Users/gpfister/.gemini/antigravity/brain/401cb359-82fe-40de-a4e6-222c7a01c706/plan.md)
- Feature Specification & Tracking: [[features/project-grooming-plugin|Project Grooming Plugin & Skill (/project-grooming)]]
- Component Specification: [[plugins/agentic-project-grooming-plugin|agentic-project-grooming-plugin]]
- Architecture Overview: [[architecture/overview|Agentic Workspace Architecture Overview]]
- Plugin Standards: [[architecture/plugin-standards|Antigravity Plugin Standards]]
