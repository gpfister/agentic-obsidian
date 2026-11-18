---
adr_id: "012"
title: "Exclusive Code-Change Authority for @coding-develop Across All Agentic Plugins"
status: "accepted"
date: "2026-11-18"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/5e833364-2100-4388-bb5e-9dc4468830ae/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/5e833364-2100-4388-bb5e-9dc4468830ae/plan.md"
tags:
  - architecture-decision
  - governance
  - security-boundary
  - coding-develop
  - anti-direct-coding
  - plan-approved
aliases:
  - ADR-012
  - adr-012-exclusive-code-change-authority
---

# ADR-012: Exclusive Code-Change Authority for `@coding-develop` Across All Agentic Plugins

## Context

Across the `agentic` plugin ecosystem, multiple agents either declared code-editing tools (`edit_file`) or contained fallback instructions permitting direct repository file writes:

1. **`@coding-deploy`** declared `edit_file` in its frontmatter, creating an avenue for deployment-time hotfixes to bypass `@coding-develop` and `@coding-review`.
2. **`@coding-document`** declared `edit_file` and had legacy permissions to edit `README.md` / `CONTRIBUTING.md` in code repositories directly.
3. **`/project-grooming`** (`agentic-project-grooming-plugin`) included a fallback path when `agentic-coding-plugin` was absent that instructed the agent to write repository scaffolding files directly via `write_file` / `run_command`.
4. **Companion plugins** (`@coding-investigate`, `@coding-monitor`, `@coding-research`) needed explicit, uniform restrictions on `write_file` and `run_command` to guarantee zero unreviewed mutations to repository source code, configurations, or test suites.

### Alternatives Considered

1. **Rely Solely on Prose Warnings While Leaving `edit_file` Declared**:
   - *Rejection Rationale*: Tool declarations in agent frontmatter grant runtime capability; prose-only warnings do not prevent accidental or LLM-driven direct edits.
2. **Allow `@coding-deploy` and `/project-grooming` Exceptions for Hotfixes and Scaffolding**:
   - *Rejection Rationale*: Bypasses worktree isolation ([[architecture/adr-011-worktree-based-development-lifecycle|ADR-011]]) and the `@coding-review` Plan-vs-Changes audit gate ([[architecture/adr-010-coding-review-plan-audit-and-unplanned-changes-gate|ADR-010]]).
3. **Strict Tool-Level & Rule-Level Enforcement Granting Code-Change Authority Exclusively to `@coding-develop` (Selected)**:
   - *Rationale*: Enforces single-point code mutation through `@coding-develop` inside isolated `.worktrees/<slug>` worktrees, guaranteeing every repository change passes through `@coding-review` and `@coding-merge`.

---

## Decision

We formalize **Exclusive Code-Change Authority** across all 6 plugin repositories and the root `agentic` workspace:

| Agent / Skill | `edit_file` | `write_file` Scope | `run_command` Scope |
| :--- | :--- | :--- | :--- |
| **`@coding-develop`** | **Allowed** (Sole agent) | Repository source, tests, configs, repo-local docs (inside `worktree_path` only) | Full build, test, linter, git worktree/commit commands (inside `worktree_path`) |
| **`@coding-merge`** | Prohibited | Prohibited | Git merge, PR, branch cleanup, `git worktree remove` / `prune` |
| **`@coding-deploy`** | **Removed** | Session artifacts (`deployment-issue*.md`) only | Read-only / operational deployment & health checks against `worktree_path` |
| **`@coding-review`** | Prohibited | Session artifacts (`review-<slug>.md`) only | Read-only git inspection (`status`, `diff`), tests & linters inside `worktree_path` |
| **`@coding`**, `@coding-clarify`, `@coding-specify`, `@coding-plan` | Prohibited | Session artifacts (`specification.md`, `plan.md`, etc.) under `<appDataDir>/brain/<conversation-id>/` | Read-only workspace inspection |
| **`@coding-document`** | **Removed** | `*-obsidian/` vault documentation & session artifacts only | Read-only git inspection & `scripts/sync_sessions.py` |
| **`@coding-investigate`** | Prohibited | `issue-assessment.md` only | Strictly read-only |
| **`@coding-monitor`** | Prohibited | Prohibited for repo code/config/tests | Strictly non-destructive read/tail observation |
| **`@coding-research`** | Prohibited | `research-report.md` only | Strictly read-only |
| **`/project-grooming`** (`@coding-groom`) | Prohibited | `Project_Request_Grooming.md` only (fallback outputs manual guidance/templates) | Strictly read-only |

### Key Architectural Rules

1. **Exclusive `edit_file` Declaration**: `edit_file` **MUST** be declared **only** in [`agentic-coding-plugin/agents/coding-develop/agent.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-plugin/agents/coding-develop/agent.md). It is removed from `@coding-deploy`, `@coding-document`, and any other agent definition.
2. **`write_file` Scoping**: On all agents other than `@coding-develop`, `write_file` is restricted strictly to session artifacts under `<appDataDir>/brain/<conversation-id>/*.md` (or `*-obsidian/` vault documentation for `@coding-document` only). Repo-local documentation edits (`README.md`, `CONTRIBUTING.md`) are delegated to `@coding-develop`.
3. **Non-Mutating `run_command`**: On all agents other than `@coding-develop` (and `@coding-merge` for git merge/branch/worktree lifecycle commands), `run_command` **MUST** be strictly read-only or operational and **MUST NEVER** mutate repository source code, configs, or tests (e.g., no `sed -i`, shell redirection `>`, `echo >>`, or auto-fix flags).
4. **`/project-grooming` Fallback**: When `agentic-coding-plugin` is not installed, `/project-grooming` provides manual scaffolding guidance (commands and file templates in chat or in `Project_Request_Grooming.md`) instead of writing repository files directly.

---

## Consequences

- **Positive:**
  - Eliminates unreviewed direct edits from deployment, documentation, grooming, investigation, monitoring, and research agents.
  - Guarantees 100% of repository code, configuration, test, and repo-local documentation changes flow through `@coding-develop` -> `@coding-review` -> `@coding-merge`.
  - Simplifies security auditing: grepping `edit_file` across `*/agents/` yields a single match (`coding-develop/agent.md`).
- **Negative:**
  - Repo-local documentation updates (`README.md`, `CONTRIBUTING.md`) must be routed through `@coding-develop` rather than edited directly by `@coding-document`.
- **Risks:**
  - *Risk*: Future plugins or agent definitions re-introducing `edit_file` or un-scoped `write_file`.
  - *Mitigation*: Automated verification check (`grep -rn "edit_file" */agents/`) and consistent `rules/AGENTS.md` guardrails across all 6 plugin repositories.
