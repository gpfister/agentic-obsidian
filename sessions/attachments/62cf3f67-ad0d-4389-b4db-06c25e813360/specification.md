# Technical Specification: Plan-vs-Changes Audit & Unplanned Changes Validation in `@coding-review`

## 1. Summary & Problem Motivation

Currently, Phase 6 (`@coding-review`) in `agentic-coding-plugin` audits `DevelopReport`s, `DeployReport`s, specification compliance, cross-repo contracts, tests, and linters, and then proceeds directly to the Merge Confirmation gate. It does not systematically compare actual repository git changes (`git diff`, `git status`) against the approved `plan.md`, nor does it produce a persistent review artifact or prompt the user when unplanned changes are introduced during implementation or deployment fixes.

This enhancement updates `@coding-review` and the `@coding` orchestrator to:
1. Audit all actual git changes across participating repositories against the initial approved `plan.md`.
2. Generate a persistent, non-blocking review artifact (`review-<slug>.md`) on every review execution.
3. Introduce an interactive **Unplanned Changes Validation Gate** (via the Question Relay Protocol) whenever changes outside `plan.md` are detected, allowing the user to accept them, revert them, or provide custom revision instructions before reaching the separate **Merge Confirmation Gate**.

---

## 2. Functional Requirements

### FR-1: Plan-vs-Changes Audit
- **FR-1.1 Orchestrator Context Handoff**: When `@coding` invokes `@coding-review` in Phase 6, it **MUST** provide the path to the approved `plan.md` alongside `specification.md`, `DevelopReport`s, and `DeployReport`s.
- **FR-1.2 Git Change Inspection**: `@coding-review` **MUST** inspect actual repository changes across all participating repositories (e.g., via `git status` and `git diff` against the base branch / merge base) in addition to reviewing `SpecificationDocument`, `DevelopReport`s, `DeployReport`s, cross-repo contracts, unit tests, and linters.
- **FR-1.3 Unplanned Change Detection**: `@coding-review` **MUST** compare the observed file modifications, additions, deletions, and behavioral changes against the scope, affected files, and tasks defined in `plan.md`. Any change not covered by `plan.md` **MUST** be classified as an *unplanned change* and documented with a concise justification (e.g., discovered dependency, bugfix during deployment, refactor, or unexplained drift).

### FR-2: Review Artifact (`review-<slug>.md`)
- **FR-2.1 Filename & Slug Derivation**: `@coding-review` **MUST** generate a single combined artifact per review named `review-<slug>.md`, where `<slug>` is derived in `kebab-case` from the feature, task, or branch name (e.g., `review-user-auth.md`).
- **FR-2.2 Artifact Location**: The artifact **MUST** be written to `<appDataDir>/brain/<conversation-id>/review-<slug>.md`.
- **FR-2.3 Generation Frequency**: `review-<slug>.md` **MUST** be produced on **every** review run. When no unplanned changes are detected, the Plan vs. Actual Changes Audit section **MUST** explicitly report `"None"`.
- **FR-2.4 No Approval Gate Required**: `@coding-review` **MUST** write `review-<slug>.md` directly using `write_file` **without** `RequestFeedback=True` (omitting `ArtifactMetadata` or setting `RequestFeedback=False`) and **without** dispatching `[ARTIFACT_APPROVAL_REQUEST]`.
- **FR-2.5 Required Artifact Contents**: `review-<slug>.md` **MUST** include:
  1. **Plan vs. Actual Changes Audit**: Summary of planned vs. actual changes across repositories, explicitly listing all unplanned changes with justification for each, or `"None"` if none exist.
  2. **Specification Compliance Check**: Verification against `specification.md` functional and non-functional requirements.
  3. **Cross-Repo Interface & Schema Compatibility**: Verification of shared contracts, APIs, and schemas across participating repositories.
  4. **Unit Tests & Linter Verification Results**: Observed test suite and linter execution outcomes.
  5. **Local Deployment Health Status**: Summary of `DeployReport`s and service/container health (if applicable).
- **FR-2.6 Tooling Permission**: `write_file` **MUST** be added to `coding-review`'s allowed tools, strictly scoped to writing the review artifact in `<appDataDir>/brain/<conversation-id>/`. `@coding-review` **MUST** preserve read-only integrity on all repository source files, configurations, and tests.

### FR-3: Unplanned Changes Validation Gate
- **FR-3.1 Trigger Condition**: The Unplanned Changes Validation Gate **MUST** be triggered **only** when one or more unplanned changes (changes not present in the initial `plan.md`) are detected. If no unplanned changes exist, this gate is skipped.
- **FR-3.2 Question & Options**:
  - **Question**: `"Do you approve the changes ?"`
  - **Options**:
    1. `(Recommended) Yes`
    2. `No, revert all these changes`
    *(Custom write-in text is supported natively by `ask_question` / Question Relay).*
  - **Context**: Include a concise summary of the detected unplanned changes and a reference to `review-<slug>.md`.
- **FR-3.3 Relay Protocol**:
  - **Subagent Mode**: `@coding-review` **MUST** dispatch `[QUESTION_RELAY_REQUEST]` to `@coding` (which forwards upward if `@coding` itself has a `caller_id`, or prompts via `ask_question` in Standalone Mode) and await `[QUESTION_RELAY_RESPONSE]`.
  - **Standalone Fallback Mode**: If `@coding-review` is invoked directly without a caller orchestrator, it **MUST** invoke `ask_question` directly.
- **FR-3.4 Decision Branching**:
  - **`Yes` (or `(Recommended) Yes`)**: Accept the unplanned changes and proceed to the separate **Merge Confirmation Gate** (assuming all other quality and deployment checks pass).
  - **`No, revert all these changes`**: `@coding-review` returns a `ReviewReport` indicating the user rejected unplanned changes and requested a revert. `@coding` delegates to `@coding-develop` to revert all unplanned changes, re-run tests and linters (and `@coding-deploy` if applicable), and re-verify via `@coding-review`.
  - **Custom Write-In Text**: `@coding-review` returns a `ReviewReport` containing the user's custom revision instructions. `@coding` routes the instructions to `@coding-develop` to apply the requested adjustments, re-run verification, and re-run `@coding-review`.

### FR-4: Merge Confirmation Gate
- **FR-4.1 Separation of Gates**: The **Merge Confirmation Gate** **MUST** remain a separate final gate executed **only after** unplanned changes (if any) have been approved by the user and all other quality, test, linter, and deployment checks pass.

---

## 3. Non-Functional Requirements

- **NFR-1 Timeout Safety**: `@coding-review` **MUST NOT** call `ask_question` or `write_file(RequestFeedback=True)` directly when running as a subagent, preventing Antigravity's 60-second background subagent execution timeout.
- **NFR-2 Source Code Integrity**: Adding `write_file` to `coding-review` **MUST NOT** permit modifications to workspace repositories. Repository source code, configs, and tests remain strictly read-only for `@coding-review`; all reverts or code edits **MUST** be delegated to `@coding-develop`.
- **NFR-3 Multi-Repository Consistency**: In multi-repo tasks, `git diff` / `git status` auditing and unplanned change reporting **MUST** cover every repository listed in `plan.md` (and any additional workspace repos modified during the session).
- **NFR-4 Deterministic Slug Formatting**: `<slug>` in `review-<slug>.md` **MUST** be lowercase alphanumeric with hyphens (`[a-z0-9-]+`), stripping prefixes like `feature/`, `task/`, or `bugfix/`.

---

## 4. Affected Repositories & Components

- **Repository**: `agentic-coding-plugin` (`/Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin`)
- **Components**:
  - `@coding` orchestrator skill/agent (`skills/coding-orchestrator/SKILL.md` and related orchestrator definitions): Pass `plan.md` path to `@coding-review`; handle `ReviewReport` rework loops for reverting unplanned changes or applying custom write-in revisions via `@coding-develop`.
  - `@coding-review` skill/agent (`agents/coding-review.md` / `skills/coding-review/SKILL.md` or equivalent definition in `agentic-coding-plugin`): Add `write_file` tool permission (artifact-only), implement Plan-vs-Changes git audit, generate `review-<slug>.md`, implement Unplanned Changes Validation Gate via Question Relay, and preserve separate Merge Confirmation Gate.

---

## 5. Edge Cases & Mitigations

| Edge Case | Mitigation |
| :--- | :--- |
| **No unplanned changes detected** | `review-<slug>.md` is still generated with `"None"` under Plan vs. Actual Changes Audit; Unplanned Changes Validation Gate is skipped and workflow proceeds to Merge Confirmation Gate (if all checks pass). |
| **User selects `"No, revert all these changes"`** | `@coding-review` does **not** run `git checkout` or `git restore` itself. It reports the revert requirement in `ReviewReport` so `@coding` delegates the revert, test re-run, and re-deployment to `@coding-develop` / `@coding-deploy`, followed by a fresh `@coding-review` pass. |
| **User provides custom write-in feedback** | `@coding-review` captures the verbatim user text in `ReviewReport` and routes back through `@coding` -> `@coding-develop` -> `@coding-review`. |
| **Re-review after revert or revision** | A fresh `review-<slug>.md` (overwriting or updating the review artifact for that slug) is written reflecting the updated git diff and test results. |
| **Branch/task name contains slashes or special chars** | Strip branch prefixes (`feature/`, `bugfix/`, `task/`) and normalize to kebab-case (`[a-z0-9-]+`), falling back to `review-changes.md` if no name can be derived. |
| **Quality/test failures alongside unplanned changes** | Quality defects still fail the review audit; unplanned changes are documented in `review-<slug>.md`, and remediation is routed through `@coding` -> `@coding-develop` before Merge Confirmation. |

---

## 6. Verifiable Acceptance Criteria

1. **Orchestrator Handoff**: `coding-orchestrator/SKILL.md` (and any orchestrator agent definition) explicitly instructs `@coding` to pass `plan.md` alongside `specification.md`, `DevelopReport`s, and `DeployReport`s when invoking `@coding-review`, and handles `ReviewReport` revert/revision routing to `@coding-develop`.
2. **Tooling & Read-Only Guardrail**: `coding-review`'s definition includes `write_file` in its allowed tools, with explicit instructions restricting `write_file` to `<appDataDir>/brain/<conversation-id>/review-<slug>.md` and forbidding writes to repository files.
3. **Audit Logic**: `coding-review`'s instructions mandate comparing actual `git status` / `git diff` across all participating repositories against `plan.md`, identifying all unplanned changes with justifications.
4. **Artifact Generation**: `coding-review` always writes `review-<slug>.md` to `<appDataDir>/brain/<conversation-id>/review-<slug>.md` without `RequestFeedback=True` and without `[ARTIFACT_APPROVAL_REQUEST]`, reporting `"None"` when no unplanned changes exist.
5. **Unplanned Changes Gate**: When unplanned changes are detected, `coding-review` asks `"Do you approve the changes ?"` with options `(Recommended) Yes` and `No, revert all these changes` (supporting custom write-in text) via `[QUESTION_RELAY_REQUEST]` in subagent mode or `ask_question` in standalone mode, and branches correctly on all three outcomes.
6. **Separate Merge Gate**: Merge Confirmation remains a separate final gate executed only after unplanned changes (if any) are approved and all quality/deployment checks pass.
