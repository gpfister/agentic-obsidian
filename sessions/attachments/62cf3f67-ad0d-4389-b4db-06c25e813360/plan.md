# Implementation Plan: Plan-vs-Changes Audit & Unplanned Changes Validation in `@coding-review`

> [!NOTE]
> **Specification**: [specification.md](file:///Users/gpfister/.gemini/antigravity/brain/62cf3f67-ad0d-4389-b4db-06c25e813360/specification.md)  
> **Scope**: Single-repository update in `agentic-coding-plugin`

---

## Repository 1: `agentic-coding-plugin`

- **Path**: `/Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin`
- **Git Branch**: `feature/review-plan-audit`
- **Local Deployment**: Not Required (no Dockerfile, Docker Compose, Terraform, Kubernetes/Helm manifests, local deployment scripts, or Makefile deploy targets exist in this repository).

### Concrete Implementation Steps

#### Step 1: Update `@coding-review` Subagent Definition
- **File to Modify**: [agents/coding-review/agent.md](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-review/agent.md)
- **Changes**:
  1. **Tooling Permission (`FR-2.6`, `NFR-2`)**:
     - Add `write_file` to `tools:` in the YAML frontmatter.
     - Add an explicit **Artifact-Only Write Guardrail** restricting `write_file` strictly to `<appDataDir>/brain/<conversation-id>/review-<slug>.md` and forbidding any writes to repository source code, configurations, or tests.
  2. **Plan-vs-Changes Audit (`FR-1.2`, `FR-1.3`, `NFR-3`)**:
     - Instruct `@coding-review` to inspect actual repository changes across all participating repositories listed in `plan.md` (and any additional modified workspace repos) using `git status` and `git diff` against the base branch / merge base.
     - Compare observed file modifications, additions, deletions, and behavioral changes against the scope, affected files, and tasks in `plan.md`.
     - Classify any change not covered by `plan.md` as an *unplanned change* with a concise justification (e.g., discovered dependency, bugfix during deployment, refactor, or unexplained drift).
  3. **Review Artifact Generation (`FR-2.1`–`FR-2.5`, `NFR-1`, `NFR-4`)**:
     - Derive `<slug>` in `kebab-case` (`[a-z0-9-]+`) from the feature, task, or branch name, stripping prefixes (`feature/`, `task/`, `bugfix/`) and falling back to `review-changes.md` if no name can be derived.
     - Write `<appDataDir>/brain/<conversation-id>/review-<slug>.md` on **every** review run using `write_file` **without** `RequestFeedback=True` (omitting `ArtifactMetadata` or setting `RequestFeedback=False`) and **without** dispatching `[ARTIFACT_APPROVAL_REQUEST]`.
     - Require the 5 mandatory sections in `review-<slug>.md`:
       1. **Plan vs. Actual Changes Audit** (explicitly `"None"` when no unplanned changes exist)
       2. **Specification Compliance Check**
       3. **Cross-Repo Interface & Schema Compatibility**
       4. **Unit Tests & Linter Verification Results**
       5. **Local Deployment Health Status**
  4. **Unplanned Changes Validation Gate (`FR-3.1`–`FR-3.4`, `NFR-1`)**:
     - Trigger **only** when one or more unplanned changes are detected (skip if none).
     - Prompt via `[QUESTION_RELAY_REQUEST]` in Subagent Mode (or `ask_question` in Standalone Fallback Mode) with:
       - **Question**: `"Do you approve the changes ?"`
       - **Options**: `(Recommended) Yes` and `No, revert all these changes` (native custom write-in text supported).
       - **Context**: Concise summary of detected unplanned changes and reference to `review-<slug>.md`.
     - Handle all 3 decision branches:
       - `Yes` / `(Recommended) Yes`: Accept unplanned changes and proceed to the separate **Merge Confirmation Gate** (if all other checks pass).
       - `No, revert all these changes`: Do **not** run `git checkout` or `git restore` directly; return a `ReviewReport` instructing `@coding` to delegate reverting all unplanned changes, re-running tests/linters (and `@coding-deploy` if applicable), and re-running `@coding-review`.
       - Custom write-in text: Return a `ReviewReport` containing the verbatim user revision instructions so `@coding` routes them to `@coding-develop`, re-runs verification, and re-runs `@coding-review`.
  5. **Separate Merge Confirmation Gate (`FR-4.1`)**:
     - Preserve Merge Confirmation as a separate final gate executed **only after** unplanned changes (if any) are approved and all quality, test, linter, and deployment checks pass.

#### Step 2: Update `@coding` Orchestrator Definition & Skill
- **Files to Modify**:
  - [agents/coding/agent.md](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding/agent.md)
  - [skills/coding-orchestrator/SKILL.md](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/skills/coding-orchestrator/SKILL.md)
- **Changes**:
  1. **Phase 6 Handoff (`FR-1.1`)**: Update the `invoke_subagent(TypeName="coding-review", ...)` prompt and Stage 6 instructions to explicitly pass the approved `plan.md` path alongside `specification.md`, `DevelopReport`s, and `DeployReport`s.
  2. **Rework Routing (`FR-3.4`)**: Document handling of `ReviewReport` outcomes when the user selects `No, revert all these changes` or provides custom write-in revision instructions: delegate to `@coding-develop` (and `@coding-deploy` if applicable), then re-run `@coding-review` (which overwrites/updates `review-<slug>.md`).
  3. **Flowchart & Gate Separation (`FR-4.1`)**: Update the Phase 6 Mermaid flowchart and stage descriptions to reflect `review-<slug>.md` generation, the conditional Unplanned Changes Validation Gate, and the separate Merge Confirmation Gate.

#### Step 3: Update Central Agent Catalog
- **File to Modify**: [agents/agent.md](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/agent.md)
- **Changes**:
  1. Update the `@coding-review` row in the Allowed Tools table to include `write_file` (strictly scoped to `review-<slug>.md` artifacts) and summarize Plan-vs-Changes auditing.
  2. Update `### Stage 6: Review (coding-review)` to document `plan.md` handoff, `review-<slug>.md` generation, the Unplanned Changes Validation Gate, and revert/revision rework routing via `@coding-develop`.

#### Step 4: Add Automated Verification Tests
- **File to Modify**: [tests/test_plugin.py](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/tests/test_plugin.py)
- **Changes**:
  1. Add tests verifying `coding-review/agent.md` includes `write_file` in `tools:`, enforces artifact-only write restrictions, documents `git status`/`git diff` comparison against `plan.md`, `review-<slug>.md` generation without `RequestFeedback=True`/`[ARTIFACT_APPROVAL_REQUEST]`, `"None"` reporting when no unplanned changes exist, all 5 required sections, and kebab-case slug normalization.
  2. Add tests verifying the Unplanned Changes Validation Gate (`"Do you approve the changes ?"`, `(Recommended) Yes`, `No, revert all these changes`, custom write-in handling, `[QUESTION_RELAY_REQUEST]`) and separation from the Merge Confirmation Gate.
  3. Add tests verifying `agents/coding/agent.md`, `skills/coding-orchestrator/SKILL.md`, and `agents/agent.md` document passing `plan.md` to `@coding-review` and routing revert/revision `ReviewReport`s to `@coding-develop`.

### Verification Commands
```bash
cd /Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin && uv run pytest tests/test_plugin.py
```
