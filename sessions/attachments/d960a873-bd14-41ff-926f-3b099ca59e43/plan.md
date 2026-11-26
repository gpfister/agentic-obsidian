# Implementation Plan: Semantic Laya Triage & Parallel `choice` + `noul` Decomposition

**Approved Specification**: [`specification.md`](file:///Users/gpfister/.gemini/antigravity/brain/d960a873-bd14-41ff-926f-3b099ca59e43/specification.md)

---

## 1. Repository & Branch Breakdown

### Repository: `agentic-prompt-triage-plugin`
- **Repository Root**: [`/Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin)
- **Git Branch**: `feature/semantic-laya-triage` (isolated worktree at `.worktrees/semantic-laya-triage`)
- **Local Deployment**: Not Required
  > Inspected repository root: no `Dockerfile`, `docker-compose.yml`/`compose.yaml`, `./scripts/deploy*_local.sh`, `terraform/local`, `k8s/`/`helm/`/`deploy/`, or `Makefile` local deployment targets exist. `agentic-prompt-triage-plugin` is a Python lifecycle hook package managed via `uv` and `pytest`.

---

## 2. Work Items & Concrete Steps

### Step 1: Semantic Criteria Keys & Anchored `TRIAGE_QUESTION` (FR-1, FR-3)
- **Files to Modify**:
  - [`hooks/prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/hooks/prompt_triage.py)
- **Changes**:
  1. Update `TRIAGE_QUESTION` to:
     ```python
     TRIAGE_QUESTION = "Which single agent or action should handle `prompt`?"
     ```
  2. Refactor `build_dynamic_triage_spec(installed_plugins)` to use semantic criteria keys instead of `response 1`..`response 9`:
     - `inquiry`: `"Read-only question, explanation, or code review making no file, code, design, or documentation changes."` (always enabled; maps to intent `"question"`)
     - `coding`: `"Implement, edit, refactor, fix, or build source code, configuration, or tests."` (when `agentic-coding-plugin` is enabled)
     - `monitoring`: `"Watch, tail, or monitor live application logs, containers, or serial console streams."` (when `agentic-task-monitoring-plugin` or aliases are enabled)
     - `investigation`: `"Diagnose bugs, trace errors, or investigate root causes without editing code."` (when `agentic-issue-investigation-plugin` or alias is enabled)
     - `research`: `"Conduct web or literature research, compare technologies, or synthesize external information."` (when `agentic-topic-researching-plugin` or alias is enabled)
     - `documentation`: `"Author, update, or sync Obsidian vault notes, ADRs, architecture docs, or session records."` (when `agentic-project-documenting-plugin` or aliases are enabled)
     - `grooming`: `"Groom project scope, audit workspace gaps, or scaffold and initialize a new repository."` (when `agentic-project-grooming-plugin` is enabled)
     - `design`: `"Create, inspect, or update UI/UX wireframes, mockups, design tokens, or PenPot files."` (when `agentic-design-plugin` is enabled)
  3. Remove `mixed` from `triage_responses` (`spec.responses` / `choice` criteria). Retain `response_to_intent["mixed"] = "mixed"` and `response_to_directive["mixed"]` so `triage()` continues to emit the `mixed` rejection directive when `evaluate_system1_triage()` returns `"mixed"`.

---

### Step 2: Parallel `choice` + `noul` Triage, `usage` Inspection & Calibrated Confidence (FR-2, FR-4)
- **Files to Modify**:
  - [`hooks/prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/hooks/prompt_triage.py)
- **Changes**:
  1. Update `LayaTriageEngine.evaluate_system1_triage(self, prompt: str)` to submit a single `self._predict_with_laya(prompt, questions)` forward pass containing:
     - `"about"` (`type: "choice"`, `instructions: TRIAGE_QUESTION`, `criteria: self.triage_responses`)
     - `"requires_changes"` (`type: "noul"`, `instructions: "Does `prompt` ask to create, edit, scaffold, or modify files, code, designs, or documentation?"`)
     - `"is_mixed"` (`type: "noul"`, `instructions: "Does `prompt` combine two or more separate tasks for different workflows?"`) — omitted or ignored when `len(self.triage_responses) <= 1`.
  2. Inspect `res.get("usage", {})`:
     - Log structured `logger.warning(...)` if `usage.get("truncated")` is truthy or if `usage.get("options")` indicates collisions/mismatches against `self.triage_responses`.
  3. Extract calibrated confidence (`ans.get("confidence")`) first, falling back to `probs.get(choice, ans.get("answer_confidence", 0.0))` for `"about"` and `ans.get("confidence", ans.get("answer_confidence", 0.0))` for `noul` answers.
  4. Apply Python post-processing:
     - **Mixed intent**: When `len(self.triage_responses) > 1` and `is_mixed` resolves to `"yes"` with high confidence (`>= CONFIDENCE_THRESHOLD`, or `"yes"` with significant secondary capability probability mass), return `("mixed", mixed_conf, probs)`.
     - **Mutating-agent gating**: Define mutating keys `{"coding", "documentation", "grooming", "design"}` and exempt non-mutating keys `{"monitoring", "investigation", "research"}`.
       - If `choice == "inquiry"` and `requires_changes == "yes"` with high confidence: cap/penalize confidence below `CONFIDENCE_THRESHOLD` (triggering `ask_question` disambiguation) or promote a dominant mutating candidate if applicable.
       - If `choice` is a mutating key and `requires_changes == "no"` with high confidence: cap/penalize confidence below `CONFIDENCE_THRESHOLD` so `triage()` triggers `ask_question` disambiguation.
       - Never penalize `monitoring`, `investigation`, or `research` when `requires_changes == "no"`.
     - If `requires_changes` or `is_mixed` is absent in mocked/legacy responses, treat as neutral.
  5. Update `LayaTriageEngine._heuristic_system1_triage(self, prompt: str)` to return semantic keys (`"inquiry"`, `"coding"`, `"monitoring"`, `"investigation"`, `"research"`, `"documentation"`, `"grooming"`, `"design"`, `"mixed"`), defaulting ambiguous prompts to `"inquiry"`.
  6. Update `triage()` default fallback key from `"response 1"` to `"inquiry"`.

---

### Step 3: Update & Expand Unit Tests (FR-5.1)
- **Files to Modify**:
  - [`tests/test_prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/tests/test_prompt_triage.py)
- **Changes**:
  1. Update existing assertions expecting `response 1`..`response 9` to expect semantic keys (`inquiry`, `coding`, `monitoring`, `investigation`, `research`, `documentation`, `grooming`, `design`).
  2. Update `build_dynamic_triage_spec` tests to assert `"mixed"` is **not** in `triage_responses` while remaining in `response_to_intent` / `response_to_directive`.
  3. Update `TRIAGE_QUESTION` assertions to `"Which single agent or action should handle `prompt`?"`.
  4. Add unit tests (with mocked `self._router.predict`) covering:
     - Single `predict` call containing `"about"` (`choice`), `"requires_changes"` (`noul`), and `"is_mixed"` (`noul`).
     - `is_mixed == "yes"` triggering `"mixed"` rejection directive.
     - `inquiry` + `requires_changes == "yes"` gating (`conf < 0.80` -> `requires_user_confirmation=True`).
     - Mutating agents (`coding`, `design`, `documentation`, `grooming`) + `requires_changes == "no"` gating (`conf < 0.80` -> `requires_user_confirmation=True`).
     - Non-mutating agents (`monitoring`, `investigation`, `research`) + `requires_changes == "no"` **not** penalized.
     - Calibrated `ans["confidence"]` preferred over `probs[choice]` and `ans["answer_confidence"]`, with fallback when `confidence` is `None`.
     - `res["usage"]` inspection logging warnings on `truncated=True` or `options` collision.
     - Single-plugin (`inquiry`-only) edge case omitting/ignoring `is_mixed`.

---

### Step 4: Empirical Laya Evaluation & Router Model Selection (FR-4.3, FR-5.2)
- **Files to Create**:
  - `scripts/eval_laya_triage.py` (empirical evaluation script exercising `LayaTriageEngine(use_router=True)` across all 10 prompt categories: `inquiry`, `coding`, `monitoring`, `investigation`, `research`, `documentation`, `grooming`, `design`, `mixed`, and ambiguous prompts).
- **Changes**:
  1. Run `eval_laya_triage.py` comparing `Router(preload=False)` (`default`) vs `Router(model="typed-decisions", preload=False)` (or per-call `model` if supported by installed `laya>=0.4.15`).
  2. Capture per-category accuracy, confusion matrix / misclassifications, calibrated confidence distribution, and `usage` (`truncated` / `options`) status.
  3. Configure `LayaTriageEngine.__init__` in [`hooks/prompt_triage.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/hooks/prompt_triage.py) to use the better-performing model option.

---

### Step 5: Documentation Consistency (FR-6)
- **Files to Modify**:
  - [`README.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/README.md)
  - [`rules/AGENTS.md`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/rules/AGENTS.md)
- **Changes**:
  1. Ensure all supported plugins including `agentic-design-plugin` (`@coding-design`) are documented.
  2. Document the semantic criteria keys, parallel `choice` + `noul` (`requires_changes`, `is_mixed`) decomposition, `usage` diagnostics, and calibrated confidence behavior.

---

## 3. Verification Commands

Run all commands inside the worktree `.worktrees/semantic-laya-triage`:

```bash
# 1. Run full unit & plugin test suite
uv run pytest -v

# 2. Run empirical Laya Router benchmark (default vs typed-decisions)
uv run python scripts/eval_laya_triage.py

# 3. Smoke-test CLI hook entrypoint with sample JSON stdin
echo '{"prompt": "Explain how prompt_triage.py works", "invocationNum": 0}' | uv run python hooks/prompt_triage.py
```
