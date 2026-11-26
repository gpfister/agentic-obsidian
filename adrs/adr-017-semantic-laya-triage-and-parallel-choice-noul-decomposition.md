---
adr_id: "017"
title: "Semantic Laya Triage & Parallel choice + noul Decomposition"
status: "accepted"
date: "2026-11-26"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/d960a873-bd14-41ff-926f-3b099ca59e43/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/d960a873-bd14-41ff-926f-3b099ca59e43/plan.md"
tags:
  - architecture-decision
  - prompt-triage
  - laya-router
  - system-1
  - semantic-classification
  - plan-approved
aliases:
  - ADR-017
  - adr-017-semantic-laya-triage-and-parallel-choice-noul-decomposition
---

# ADR-017: Semantic Laya Triage & Parallel `choice` + `noul` Decomposition

See canonical record: [[architecture/adr-017-semantic-laya-triage-and-parallel-choice-noul-decomposition|ADR-017]].

## Summary

- **Semantic `criteria` Keys**: Replaces opaque `response 1`..`response 9` keys in `build_dynamic_triage_spec` with semantic keys (`inquiry`, `coding`, `monitoring`, `investigation`, `research`, `documentation`, `grooming`, `design`) and removes `mixed` from `choice` criteria.
- **Single-Pass Parallel `choice` + `noul` Decomposition**: Evaluates `"about"` (`choice`), `"requires_changes"` (`noul`), and `"is_mixed"` (`noul`) in one `self._router.predict({"prompt": text}, questions)` call, gating mutating agents (`coding`, `documentation`, `grooming`, `design`) and `inquiry` while exempting non-mutating specialized agents (`monitoring`, `investigation`, `research`).
- **Anchored Instructions, `usage` Diagnostics & Calibrated Confidence**: Anchors `TRIAGE_QUESTION` and `noul` prompts to `` `prompt` ``, inspects `res.get("usage", {})` for `truncated`/`options`, prefers `ans.get("confidence")`, and benchmarks `default` vs `typed-decisions` via `scripts/eval_laya_triage.py`.
