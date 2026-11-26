# Technical Specification: Semantic Laya Triage & Parallel `choice` + `noul` Decomposition

## 1. Summary & Problem Motivation

[`agentic-prompt-triage-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin) currently routes incoming user prompts in [`hooks/prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/hooks/prompt_triage.py) using a single `choice` question (`TRIAGE_QUESTION = "What is this prompt about ?"`) over opaque numbered criteria keys (`response 1` .. `response 9`), including a bloated `mixed` catch-all option.

This design has several drawbacks for Laya Router System 1 zero-shot / non-autoregressive classification:
1. **Opaque keys (`response 1`..`response 9`)** provide no semantic anchor for the model and couple option ordering to plugin combinations.
2. **Passive, unanchored instruction** (`"What is this prompt about ?"`) does not reference the state key (`` `prompt` ``) or frame the decision around agent/action routing.
3. **Bloated `mixed` catch-all option** enumerates every active capability in a single long string, diluting probability mass and increasing token length/truncation risk.
4. **Single-question classification** cannot independently verify whether a prompt requests file/code/design/doc mutations (`requires_changes`) or combines multiple workflows (`is_mixed`).
5. **Uninspected `usage` and uncalibrated confidence**: `evaluate_system1_triage` does not inspect `res.get("usage", {})` for `truncated` or `options` collisions, and reads `probs.get(choice, ans.get("answer_confidence", 0.0))` before checking Laya's calibrated `confidence` field.

This specification defines the refactoring of `hooks/prompt_triage.py`, its test suite in `tests/test_prompt_triage.py`, an empirical Laya Router benchmark/evaluation suite, and documentation updates.

---

## 2. Functional Requirements

### FR-1: Semantic `criteria` Keys & Action-Oriented Descriptions
In [`build_dynamic_triage_spec`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/hooks/prompt_triage.py#L144-L323):
1. Replace opaque numbered keys (`response 1` .. `response 9`) with semantic keys:
   - `inquiry` (always enabled)
   - `coding` (when `agentic-coding-plugin` is enabled)
   - `monitoring` (when `agentic-task-monitoring-plugin` or aliases are enabled)
   - `investigation` (when `agentic-issue-investigation-plugin` or alias is enabled)
   - `research` (when `agentic-topic-researching-plugin` or alias is enabled)
   - `documentation` (when `agentic-project-documenting-plugin` or aliases are enabled)
   - `grooming` (when `agentic-project-grooming-plugin` is enabled)
   - `design` (when `agentic-design-plugin` is enabled)
2. Replace verbose `"one or more ..."` descriptions with concise, mutually-exclusive, action-oriented descriptions that clearly separate read-only inquiries/reviews from mutating or specialized workflows:
   - `inquiry`: Read-only question, explanation, or code review making no file, code, design, or documentation changes.
   - `coding`: Implement, edit, refactor, fix, or build source code, configuration, or tests.
   - `monitoring`: Watch, tail, or monitor live application logs, containers, or serial console streams.
   - `investigation`: Diagnose bugs, trace errors, or investigate root causes without editing code.
   - `research`: Conduct web or literature research, compare technologies, or synthesize external information.
   - `documentation`: Author, update, or sync Obsidian vault notes, ADRs, architecture docs, or session records.
   - `grooming`: Groom project scope, audit workspace gaps, or scaffold and initialize a new repository.
   - `design`: Create, inspect, or update UI/UX wireframes, mockups, design tokens, or PenPot files.
3. **Remove `mixed` from `spec.responses`** (`choice` criteria). Mixed-intent detection is handled via the parallel `is_mixed` `noul` question (FR-2).
4. Maintain `response_to_intent` / `intent_to_response` mappings for all active semantic keys (with `inquiry` mapping to intent `"question"` or `"inquiry"` consistently across `TriageResult`, `PromptTriageHook`, and directives; preserving `"question"` as the `TriageResult.intent` for `inquiry` avoids breaking downstream hook consumers) and keep the `mixed` rejection directive accessible when `intent == "mixed"` is triggered.

### FR-2: Parallel `choice` + `noul` Decomposition in One Forward Pass
In [`LayaTriageEngine.evaluate_system1_triage`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/hooks/prompt_triage.py#L437-L454):
1. Submit all questions in a single `self._router.predict({"prompt": text}, questions)` forward pass:
   - `"about"` (`type: "choice"`):
     - `instructions`: `TRIAGE_QUESTION`
     - `criteria`: `self.triage_responses` (only active single-intent semantic keys; never `mixed`)
   - `"requires_changes"` (`type: "noul"`):
     - `instructions`: `"Does `prompt` ask to create, edit, scaffold, or modify files, code, designs, or documentation?"`
   - `"is_mixed"` (`type: "noul"`):
     - `instructions`: `"Does `prompt` combine two or more separate tasks for different workflows?"`
   - *(Note: If only `inquiry` is active — i.e., `len(self.triage_responses) == 1` — `is_mixed` is irrelevant/skipped or ignored.)*
2. Combine `choice` probabilities with `requires_changes` and `is_mixed` signals in Python:
   - **Mixed intent detection**:
     - If `is_mixed` resolves to `"yes"` with high confidence (e.g., `is_mixed_conf >= CONFIDENCE_THRESHOLD`, or `is_mixed == "yes"` combined with significant secondary probability mass on a second capability), return `("mixed", mixed_conf, probs)` so `triage()` emits the `mixed` rejection directive.
   - **Gating `inquiry` vs. mutating action agents** (`coding`, `documentation`, `grooming`, `design`):
     - Non-mutating specialized agents (`monitoring`, `investigation`, `research`) do **not** modify repository files and MUST NOT be penalized when `requires_changes == "no"`.
     - If `choice == "inquiry"` while `requires_changes == "yes"` with high confidence: treat as a conflicting signal (cap/penalize confidence below `CONFIDENCE_THRESHOLD` so `triage()` triggers `ask_question` disambiguation, or promote the highest-scoring mutating agent only if its probability is dominant).
     - If `choice` is a mutating agent (`coding`, `documentation`, `grooming`, `design`) while `requires_changes == "no"` with high confidence: cap/penalize confidence below `CONFIDENCE_THRESHOLD` so `triage()` triggers `ask_question` disambiguation rather than erroneously invoking a mutating workflow on a read-only question.
3. Update [`_heuristic_system1_triage`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/hooks/prompt_triage.py#L456-L649) to return semantic keys (`"inquiry"`, `"coding"`, `"monitoring"`, `"investigation"`, `"research"`, `"documentation"`, `"grooming"`, `"design"`, `"mixed"`) matching the new `spec.responses` keys (with `"inquiry"` as the ambiguous fallback key).

### FR-3: Anchor `instructions` to the State Key and Agent Routing
Update `TRIAGE_QUESTION` in [`hooks/prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/hooks/prompt_triage.py#L50) to an action-framed instruction explicitly referencing `` `prompt` ``:
```python
TRIAGE_QUESTION = "Which single agent or action should handle `prompt`?"
```
Ensure all `noul` question instructions also reference `` `prompt` `` to match the `{"prompt": text}` state dictionary passed to `self._router.predict`.

### FR-4: Inspect `usage` & Calibrated `confidence`
1. **Usage inspection**:
   - Inspect `res.get("usage", {})` returned by `self._router.predict({"prompt": text}, questions)`.
   - Check for `truncated` (boolean or truthy indicator) and `options` (e.g., collisions, dropped options, or mismatch against `self.triage_responses` keys).
   - Log structured warnings via `logger.warning(...)` if truncation or option collision is detected, and include `usage` diagnostics in telemetry/metadata when anomalous.
2. **Calibrated confidence extraction**:
   - Extract confidence using Laya's calibrated `confidence` field first:
     ```python
     conf = ans.get("confidence")
     if conf is None:
         conf = probs.get(choice, ans.get("answer_confidence", 0.0))
     ```
   - Apply the same calibrated `confidence` preference when reading `requires_changes` and `is_mixed` `noul` answers.
3. **Router model evaluation**:
   - Evaluate `Router(preload=False)` (default) vs `Router(model="typed-decisions", preload=False)` (or per-call `model` parameter if supported by the installed `laya>=0.4.15` API) on the empirical benchmark suite (FR-5), and configure `LayaTriageEngine` to use the better-performing option.

### FR-5: Empirical Laya Evaluation & Unit Tests
1. **Unit tests** in [`tests/test_prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/tests/test_prompt_triage.py):
   - Update all assertions expecting `response 1` .. `response 9` to expect semantic keys (`inquiry`, `coding`, `monitoring`, `investigation`, `research`, `documentation`, `grooming`, `design`).
   - Update `build_dynamic_triage_spec` tests to verify `mixed` is **absent** from `spec.responses`.
   - Update `TRIAGE_QUESTION` assertions to `"Which single agent or action should handle `prompt`?"`.
   - Add unit tests (using mocked `self._router.predict` responses) covering:
     - Parallel `choice` + `noul` (`requires_changes`, `is_mixed`) parsing in a single `predict` call.
     - `is_mixed == "yes"` triggering `mixed` rejection directive.
     - `inquiry` + `requires_changes == "yes"` gating (triggers disambiguation `< 0.80`).
     - Mutating agent (`coding`/`design`/`documentation`/`grooming`) + `requires_changes == "no"` gating (triggers disambiguation `< 0.80`).
     - Non-mutating agent (`monitoring`/`investigation`/`research`) + `requires_changes == "no"` **not** being penalized.
     - Calibrated `ans["confidence"]` preference over `probs[choice]` and `ans["answer_confidence"]`.
     - `res["usage"]` inspection logging warnings on `truncated` / `options` collisions.
2. **Empirical Laya evaluation & benchmark script**:
   - Add an empirical evaluation script/test (e.g., `scripts/eval_laya_triage.py` or `tests/test_laya_empirical.py`) exercising `LayaTriageEngine(use_router=True)` against a curated dataset of representative prompts covering:
     - `inquiry`, `coding`, `monitoring`, `investigation`, `research`, `documentation`, `grooming`, `design`, `mixed`, and ambiguous prompts.
   - Report per-category accuracy, confusion matrix / misclassifications, calibrated confidence distribution, `usage` (`truncated` / `options`) status, and `default` vs `typed-decisions` model comparison.

### FR-6: Documentation Consistency
Update [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/README.md) and [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/rules/AGENTS.md) if needed so they accurately reflect:
- All supported plugins including `agentic-design-plugin` (`@coding-design`).
- Semantic criteria keys and parallel `choice` + `noul` (`requires_changes`, `is_mixed`) decomposition with calibrated confidence.

---

## 3. Non-Functional Requirements

- **Latency**: All triage questions (`about`, `requires_changes`, `is_mixed`) MUST execute in a **single** `self._router.predict({"prompt": text}, questions)` forward pass (no sequential multi-turn Laya calls).
- **Resilience / Fail-Safe**:
  - `PromptTriageHook` and `main()` MUST never crash the parent Antigravity execution loop; any Laya initialization or inference failure falls back cleanly to `_heuristic_system1_triage`.
  - Keep `CONFIDENCE_THRESHOLD = 0.80` unchanged.
- **Backward Compatibility**:
  - `TriageResult` schema fields (`intent`, `response_choice`, `response_confidence`, `response_probabilities`, `requires_user_confirmation`, `inject_steps`, `installed_plugins`) remain intact, with `response_choice` now holding semantic keys (`inquiry`, `coding`, ..., `mixed`).
  - `HookContext` state keys (`triage_result`, `inject_steps`, `triage_intent`, `response_choice`, `response_confidence`, `requires_user_confirmation`) remain unchanged.

---

## 4. Affected Repositories

- [`agentic-prompt-triage-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin)
  - [`hooks/prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/hooks/prompt_triage.py)
  - [`tests/test_prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/tests/test_prompt_triage.py)
  - Empirical evaluation script / test (new file under `tests/` or `scripts/`)
  - [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/README.md)
  - [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/rules/AGENTS.md)

---

## 5. Edge Cases & Mitigations

| Edge Case | Mitigation |
| :--- | :--- |
| Only `inquiry` is active (no capability plugins installed) | Only `inquiry` is in `spec.responses`; `is_mixed` is omitted or ignored; `len(self.triage_responses) == 1` skips `ask_question` and routes directly to `inquiry`. |
| `laya` `predict` response omits `requires_changes` or `is_mixed` | Treat missing `noul` answers as neutral (no gating penalty, no mixed override) so `choice` proceeds normally. |
| `res["usage"]` reports `truncated=True` or `options` collision | Log a warning with `usage` details; keep descriptions concise to stay well within Laya's context/token budget. |
| `ans.get("confidence")` is `None` in older/mocked Laya responses | Fall back to `probs.get(choice, ans.get("answer_confidence", 0.0))`. |
| Read-only investigation/monitoring/research prompt has `requires_changes == "no"` | Exempt `monitoring`, `investigation`, and `research` from `requires_changes == "no"` gating so read-only operational/research tasks are not penalized. |
| `laya` package or model weights unavailable in offline CI | `use_router=False` unit tests use mocked `predict` and `_heuristic_system1_triage`; empirical `use_router=True` benchmark/tests can be run explicitly or skipped gracefully if Laya weights are unavailable. |

---

## 6. Verifiable Acceptance Criteria

1. `build_dynamic_triage_spec` produces semantic keys (`inquiry`, `coding`, `monitoring`, `investigation`, `research`, `documentation`, `grooming`, `design`) with concise action-oriented descriptions and **no** `mixed` key in `spec.responses`.
2. `TRIAGE_QUESTION` is `"Which single agent or action should handle `prompt`?"` (referencing `` `prompt` ``).
3. `LayaTriageEngine.evaluate_system1_triage` invokes `self._router.predict({"prompt": prompt}, questions)` **once** per triage with `"about"` (`choice`), `"requires_changes"` (`noul`), and `"is_mixed"` (`noul`).
4. `evaluate_system1_triage` inspects `res.get("usage", {})` for `truncated` and `options`, and prefers `ans.get("confidence")` over `probs.get(choice, ans.get("answer_confidence", 0.0))`.
5. Python post-processing correctly detects `mixed` via `is_mixed`, gates `inquiry` vs. mutating agents via `requires_changes`, and does not penalize `monitoring`, `investigation`, or `research` when `requires_changes == "no"`.
6. `uv run pytest` passes all updated and newly added unit tests in `tests/test_prompt_triage.py`.
7. Empirical evaluation using `LayaTriageEngine(use_router=True)` runs across all 10 prompt categories (`inquiry`, `coding`, `monitoring`, `investigation`, `research`, `documentation`, `grooming`, `design`, `mixed`, ambiguous), logs accuracy/calibration/usage metrics, and documents the `default` vs `typed-decisions` router evaluation.
8. `README.md` and `rules/AGENTS.md` are consistent with the updated plugin capabilities and semantic triage architecture.
