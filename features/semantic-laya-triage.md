---
title: "Semantic Laya Triage & Parallel choice + noul Decomposition"
date: "2026-11-26"
status: "Merged / Completed"
phase: "Phase 7: Merge & Release Complete"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/d960a873-bd14-41ff-926f-3b099ca59e43/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/d960a873-bd14-41ff-926f-3b099ca59e43/plan.md"
review_path: "/Users/gpfister/.gemini/antigravity/brain/d960a873-bd14-41ff-926f-3b099ca59e43/review-semantic-laya-triage.md"
adrs:
  - "[[architecture/adr-017-semantic-laya-triage-and-parallel-choice-noul-decomposition|ADR-017: Semantic Laya Triage & Parallel choice + noul Decomposition]]"
tags:
  - feature
  - prompt-triage
  - laya-router
  - system-1
  - specification
  - approved-spec
  - implementation-plan
  - approved-plan
  - phase-1-completed
  - phase-2-completed
  - phase-3-completed
  - phase-4-completed
  - phase-5-completed
  - phase-6-completed
  - phase-7-completed
  - review-passed
  - merged
  - completed
  - released
affected_plugins:
  - agentic-prompt-triage-plugin
aliases:
  - semantic-laya-triage
---

# Feature: Semantic Laya Triage & Parallel `choice` + `noul` Decomposition

> [!NOTE] Lifecycle Status: Merged / Completed (`PHASE 7 COMPLETE`)
> - **Architecture Decision Record**: [[architecture/adr-017-semantic-laya-triage-and-parallel-choice-noul-decomposition|ADR-017: Semantic Laya Triage & Parallel choice + noul Decomposition]]
> - **Architecture Plan Note**: [[architecture/semantic-laya-triage|Semantic Laya Triage & Parallel choice + noul Decomposition Architecture Plan]]
> - **Affected Repository**: [`agentic-prompt-triage-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin) (`main` at `ff5a3ae`, [PR #2](https://github.com/gpfister/agentic-prompt-triage-plugin/pull/2) `MERGED`)
> - **Post-Merge Cleanup**: Worktree `.worktrees/semantic-laya-triage` removed & pruned; `feature/semantic-laya-triage` deleted locally & remotely
> - **Current Phase**: Phase 7: Merge & Release Complete (`Merged / Completed`)
> - **Verification**: `101/101` tests passing (`uv run pytest -v`), CLI smoke test passed, empirical Laya evaluation selected `default` (`Router(preload=False)`) over `typed-decisions`, QA review **PASS**
> - **Approved Spec**: [`specification.md`](file:///Users/gpfister/.gemini/antigravity/brain/d960a873-bd14-41ff-926f-3b099ca59e43/specification.md)
> - **Approved Plan**: [`plan.md`](file:///Users/gpfister/.gemini/antigravity/brain/d960a873-bd14-41ff-926f-3b099ca59e43/plan.md)
> - **QA Review Report**: [`review-semantic-laya-triage.md`](file:///Users/gpfister/.gemini/antigravity/brain/d960a873-bd14-41ff-926f-3b099ca59e43/review-semantic-laya-triage.md)

---

## 1. Feature Summary & Motivation

Refactors [`hooks/prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/hooks/prompt_triage.py) in `agentic-prompt-triage-plugin` to improve Laya Router System 1 zero-shot / non-autoregressive prompt classification accuracy, calibration, and observability without adding sequential latency.

---

## 2. Functional Requirements Summary

| Requirement | Status | Summary |
| :--- | :--- | :--- |
| **FR-1: Semantic Criteria Keys** | Merged & Verified | Replaced `response 1`..`response 9` with `inquiry`, `coding`, `monitoring`, `investigation`, `research`, `documentation`, `grooming`, `design`; removed `mixed` from `spec.responses` while preserving `response_to_intent["mixed"]` and `response_to_directive["mixed"]`. |
| **FR-2: Parallel `choice` + `noul`** | Merged & Verified | Executes `"about"` (`choice`), `"requires_changes"` (`noul`), and `"is_mixed"` (`noul`) in one `predict` pass; gates `MUTATING_AGENTS` (`coding`, `documentation`, `grooming`, `design`) & `inquiry`, while exempting `NON_MUTATING_AGENTS` (`monitoring`, `investigation`, `research`). |
| **FR-3: Anchored Instructions** | Merged & Verified | Set `TRIAGE_QUESTION = "Which single agent or action should handle `prompt`?"` and anchored `REQUIRES_CHANGES_QUESTION` and `IS_MIXED_QUESTION` to `` `prompt` ``. |
| **FR-4: `usage` & Calibrated Confidence** | Merged & Verified | `_inspect_usage` warns on `truncated` / `truncated_questions` and `options` collisions; prefers `ans.get("confidence")`; empirical evaluation selected `default` (`Router(preload=False)`) over `typed-decisions`. |
| **FR-5: Unit & Empirical Tests** | Merged & Verified | Updated `tests/test_prompt_triage.py` (`89/89` passed; `101/101` total with `tests/test_plugin.py`) and added `scripts/eval_laya_triage.py` across 10 prompt categories. |
| **FR-6: Documentation Consistency** | Merged & Verified | Updated `README.md` and `rules/AGENTS.md` to reflect semantic keys, `Router(preload=False)` default, parallel `choice` + `noul` decomposition, and `agentic-design-plugin` (`@coding-design`). |

---

## 3. Non-Functional Invariants

- **Single Forward Pass**: All three triage questions execute in one `self._router.predict({"prompt": text}, questions)` call.
- **Fail-Safe Fallback**: Any Laya initialization or inference failure falls back cleanly to `_heuristic_system1_triage` (returning semantic keys, defaulting to `"inquiry"`).
- **Backward Compatibility**: `TriageResult` and `HookContext` state schemas remain intact (`CONFIDENCE_THRESHOLD = 0.80`), with `response_choice` holding semantic keys (`inquiry`, `coding`, ..., `mixed`) and `inquiry` preserving `intent = "question"`.

---

## 4. Implementation & Verification Summary

- **Merged Commit (`ff5a3ae`, [PR #2](https://github.com/gpfister/agentic-prompt-triage-plugin/pull/2), 5 files, `846 insertions(+)`, `286 deletions(-)`)**:
  - [`hooks/prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/hooks/prompt_triage.py)
  - [`tests/test_prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/tests/test_prompt_triage.py)
  - [`scripts/eval_laya_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/scripts/eval_laya_triage.py)
  - [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/README.md)
  - [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/rules/AGENTS.md)
- **Empirical Model Selection**: `scripts/eval_laya_triage.py` benchmarked `default` (`Router(preload=False)`) vs `typed-decisions` across 10 categories; `default` outperformed `typed-decisions` and is configured in `LayaTriageEngine`.
- **Verification**: `uv run pytest -v` -> `101 passed in 3.96s` (`12` plugin tests, `89` prompt triage tests); CLI smoke test exited `0` (`{"injectSteps": []}`); QA review **PASS** with zero unplanned changes; post-merge worktree `.worktrees/semantic-laya-triage` and branch `feature/semantic-laya-triage` cleaned up and pruned.
