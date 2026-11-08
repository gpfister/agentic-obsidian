# Specification: Complete Branch Cleanup & Remote Synchronization for `@coding-merge`

## 1. Problem Statement & Motivation

In the current `agentic-coding-plugin` workflow, Stage 7 (`@coding-merge`) handles branch integration across two distinct scenarios:
1. **Local Release Strategy**: Used when no remote origin exists. `@coding-merge` checks out the base branch (`git checkout <base>`) and merges the feature branch (`git merge <branch>`), but leaves the feature branch untouched (`cleanup: false` in `MergeReport`), accumulating stale local branches over time.
2. **Remote Release Strategy**: Used when a remote repository is configured. Following Pull Request creation and confirmation of merge (`gh pr view` reports `state == "MERGED"`), `@coding-merge` checks out the base branch (`git checkout <base>`), pulls remote changes (`git pull --ff-only origin <base>`), and deletes the local feature branch (`git branch -d <branch>`). However, it does **not** delete the remote feature branch on `origin` (`git push origin --delete <branch>`), nor does it prune obsolete remote tracking branches (`git fetch --prune origin`).

### Deficiencies Identified
- **Stale Local Feature Branches**: Local-only merges do not clean up feature branches, causing local branch proliferation.
- **Stale Remote Branches**: Remote repositories accumulate merged feature branches on GitHub/GitLab unless repository-level auto-delete is configured.
- **Dangling Remote-Tracking References**: Without `git fetch --prune`, local git configurations retain stale tracking references (`origin/<branch>`), leading downstream tools (e.g., `@coding-document`, `git status`, branch listings) to report outdated branch topologies.
- **Inconsistent Working Tree State**: Failure to guarantee switching back to the base branch (`main`/`master`) in all execution paths leaves developers or downstream subagents in a detached or feature branch state.

---

## 2. Goals & Scope

### Goals
- Ensure that `@coding-merge` switches back to the primary base branch (`main` or `master`) and stays on it after all merge operations.
- Enforce full branch cleanup (local branch deletion) for **Local Release Strategy**.
- Enforce full branch cleanup (remote branch deletion, local branch deletion, and remote tracking branch pruning) for **Remote Release Strategy**.
- Handle edge cases gracefully, such as remote branches already deleted by GitHub PR auto-merge settings, squash/rebase merges where `git branch -d` requires confirmation, and missing remote tracking branches.
- Update all agent definitions, orchestrator skills, plugin rules, and automated test suites in `agentic-coding-plugin`.

### Scope
- **Target Repository**: `agentic-coding-plugin`
- **Affected Documentation & Prompts**:
  - `agents/coding-merge/agent.md`
  - `agents/coding/agent.md`
  - `agents/agent.md`
  - `skills/coding-orchestrator/SKILL.md`
  - `rules/AGENTS.md` (cross-checked for merge-related guidelines)
  - `tests/test_plugin.py`

---

## 3. Functional Requirements

### FR-1: Local Release Strategy Branch Cleanup & Base Switch
When a repository has no remote configured:
1. **Switch to Base Branch**: Check out the base branch (`git checkout <base>`).
2. **Merge Feature Branch**: Execute local fast-forward or standard merge (`git merge <branch>`).
3. **Verify Working Tree**: Confirm checkout succeeded on `<base>` (`git branch --show-current`).
4. **Delete Local Feature Branch**: Safely delete the merged local feature branch (`git branch -d <branch>`).
5. **Output Structured Report**: Emit `MergeReport` with `cleanup: true` and local deletion recorded:
   ```json
   {
     "status": "merged",
     "strategy": "local",
     "base_branch": "<base>",
     "merged_branch": "<branch>",
     "cleanup": true,
     "deleted_branches": {
       "local": "<branch>"
     }
   }
   ```

### FR-2: Remote Release Strategy Branch Cleanup & Base Switch
When a repository has a remote configured (`origin`) and PR is confirmed merged (`gh pr view` reports `state == "MERGED"`):
1. **Switch to Base Branch**: Check out the base branch (`git checkout <base>`).
2. **Synchronize Base with Remote**: Fast-forward pull remote updates (`git pull --ff-only origin <base>`).
3. **Delete Remote Feature Branch**:
   - Check if remote branch exists: `git ls-remote --heads origin <branch>`.
   - If present, delete remote branch: `git push origin --delete <branch>`.
   - If already deleted (e.g. by GitHub auto-delete head branches option), handle idempotently without error.
4. **Delete Local Feature Branch**:
   - Safely delete local feature branch: `git branch -d <branch>`.
   - In squash-merge or rebase-merge PR scenarios where git cannot detect direct ancestry, fallback to `git branch -D <branch>` strictly after verifying that PR `state == "MERGED"` and the base branch has pulled the merged commit.
5. **Prune Remote-Tracking References**:
   - Execute `git fetch --prune origin` (or `git fetch --prune`) to remove stale `origin/<branch>` references from `.git/refs/remotes/origin/`.
6. **Verify Working Tree**: Confirm active branch is `<base>` (`git branch --show-current`).
7. **Output Structured Report**: Emit `MergeReport` with `cleanup: true`, remote/local deletion status, and prune confirmation:
   ```json
   {
     "status": "merged",
     "strategy": "remote_pr",
     "pr_number": "<pr_number>",
     "pr_url": "<pr_url>",
     "base_branch": "<base>",
     "merged_branch": "<branch>",
     "cleanup": true,
     "deleted_branches": {
       "local": "<branch>",
       "remote": "<branch>"
     },
     "pruned": true
   }
   ```

### FR-3: Update Agent Definitions & Catalogs
1. **`agents/coding-merge/agent.md`**:
   - Update Section 2 ("Local Release Strategy") to include local branch deletion (`git branch -d <branch>`) and updated `MergeReport` schema (`cleanup: true`).
   - Update Section 5 ("Post-Merge Base Synchronization & Local Cleanup" renamed to "Post-Merge Base Synchronization & Complete Cleanup") to include remote branch deletion (`git push origin --delete <branch>`), local branch deletion (`git branch -d <branch>`), and remote prune (`git fetch --prune origin`).
2. **`agents/coding/agent.md`**:
   - Update Stage 7 flow and phase descriptions to document post-merge base branch switch, local/remote branch deletion, and `git fetch --prune`.
3. **`agents/agent.md`**:
   - Update Stage 7 (`coding-merge`) summary in the multi-agent catalog to document full cleanup across local and remote release strategies.
4. **`skills/coding-orchestrator/SKILL.md`**:
   - Update Phase 7 (`coding-merge`) description in Section 5 to document switching to base branch, deleting local feature branch (local merge), and deleting remote branch + local branch + running `git fetch --prune` (remote PR merge).

### FR-4: Automated Verification in Test Suite
Update `tests/test_plugin.py`:
1. Update `test_coding_merge_pr_waiting_and_sync` (or add dedicated test `test_coding_merge_branch_cleanup_and_prune`):
   - Assert `git push origin --delete` (or remote branch deletion command).
   - Assert `git fetch --prune` in `agents/coding-merge/agent.md`, `skills/coding-orchestrator/SKILL.md`, and `agents/agent.md`.
   - Assert local merge strategy specifies `git branch -d` and `cleanup: true`.
   - Assert post-merge working directory returns to base branch (`git checkout <base>`).
2. Ensure 100% test pass with `pytest`.

---

## 4. Non-Functional Requirements & Safety Boundaries

### NFR-1: Safety First
- **No Premature Deletion**: Never delete local or remote feature branches before merge completion is explicitly confirmed (`git merge` returns 0 for local; `gh pr view` reports `state == "MERGED"` for remote).
- **Safe Branch Deletion**: Use `git branch -d` as the standard mechanism. Force deletion (`-D`) is only permissible for remote PR merges where PR status is confirmed `MERGED` and local ancestry check fails due to squash/rebase commit rewriting.

### NFR-2: Idempotency & Fault Tolerance
- If the remote feature branch has already been automatically deleted by GitHub upon merging, `git push origin --delete <branch>` must not fail the release process.
- `git fetch --prune` must handle offline or non-responsive remotes with clear error logging without corrupting local state.

### NFR-3: Multi-Repository Consistency
- In multi-repository workflows orchestrated by `@coding`, each affected repository must independently execute the complete cleanup routine so that all workspaces remain on their respective base branches in a pristine state.

---

## 5. Affected Files & Repositories

| Repository | File Path | Modification Summary |
| :--- | :--- | :--- |
| `agentic-coding-plugin` | `agents/coding-merge/agent.md` | Add local branch deletion to Local Release Strategy; add remote branch deletion and `git fetch --prune` to Remote Release Strategy; update `MergeReport` schema. |
| `agentic-coding-plugin` | `agents/coding/agent.md` | Update Stage 7 description and Mermaid diagrams for post-merge cleanup. |
| `agentic-coding-plugin` | `agents/agent.md` | Update Stage 7 summary in agent catalog for local/remote cleanup and prune. |
| `agentic-coding-plugin` | `skills/coding-orchestrator/SKILL.md` | Update Section 5 (Phase 7) cleanup and synchronization instructions. |
| `agentic-coding-plugin` | `tests/test_plugin.py` | Add assertions for `git push origin --delete`, `git fetch --prune`, local merge cleanup, and base checkout. |

---

## 6. Verifiable Acceptance Criteria

- [ ] `agents/coding-merge/agent.md` explicitly specifies checking out base branch, deleting local feature branch (`git branch -d <branch>`), and returning `cleanup: true` for the Local Release Strategy.
- [ ] `agents/coding-merge/agent.md` explicitly specifies checking out base branch, pulling with `--ff-only`, deleting remote feature branch (`git push origin --delete <branch>`), deleting local feature branch (`git branch -d <branch>`), running `git fetch --prune origin`, and returning `cleanup: true` with `pruned: true` for the Remote Release Strategy.
- [ ] `agents/coding/agent.md`, `agents/agent.md`, and `skills/coding-orchestrator/SKILL.md` are aligned with the new cleanup and prune procedures.
- [ ] `tests/test_plugin.py` contains explicit assertions verifying remote branch deletion, local branch deletion, remote pruning (`git fetch --prune`), and base branch switching across the updated documentation.
- [ ] All automated unit tests in `agentic-coding-plugin` pass cleanly (`pytest tests/test_plugin.py`).
