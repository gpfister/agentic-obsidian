---
title: "Workspace Release Changelog"
date: "2026-11-04"
status: "active"
tags:
  - changelog
  - releases
  - multi-repo
---

# Workspace Release Changelog

Chronological log of cross-repository releases, feature merges, and architectural upgrades across the `agentic` workspace.

---

## [2026-11-04] — `agentic-coding-tui`: Transparent Terminal Background

> [!NOTE] Release Summary: SPEC-TUI-TRANSPARENCY-001 Fast-Forward Merged to Master
> - **Repository**: [`agentic-coding-tui`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-tui/)
> - **Target Branch**: `master` (fast-forward merged from `feature/transparent-terminal-background`)
> - **Source Branch**: `feature/transparent-terminal-background` (deleted cleanly)
> - **Specification**: `SPEC-TUI-TRANSPARENCY-001`
> - **Architecture Decision**: [[architecture/adr-009-tui-transparent-pass-through-rendering|ADR-009: TUI Transparent Pass-Through Rendering Architecture]]
> - **Feature Note**: [[features/transparent-terminal-background|Transparent Terminal Background in agentic-coding-tui]]
> - **Verification**: 30/30 automated unit tests passing on `master` (100%), clean `ruff` linter/formatter.

### Commits Merged
- [`4dea5b5`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-tui/): `feat(ui): implement transparent terminal background (SPEC-TUI-TRANSPARENCY-001)`
- [`e9e187f`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-tui/): `docs(ui): document transparent terminal background architecture and invariants`

### Added & Changed
- **100% Pass-Through Canvas Transparency**:
  - Configured `Screen` root canvas and `AgenticCodeTUIApp` with `background: transparent;` in [`app.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-tui/src/agentic_coding_tui/app.py).
  - Applied `background: transparent;` across `TranscriptStream`, `Markdown`, `MarkdownFence`, `.user-message`, `.agent-message`, and `.thought-block` in [`transcript.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-tui/src/agentic_coding_tui/ui/transcript.py).
  - Made `PromptInput`, `CommandInput` (`#prompt-input`), and completion helpers completely transparent in [`prompt_input.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-tui/src/agentic_coding_tui/ui/prompt_input.py).
  - Enforced transparent background on `TaskSubagentFooter` and `#status-row` in [`status_bar.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-tui/src/agentic_coding_tui/ui/status_bar.py).
  - Configured `AskQuestionDialog` modal backdrop and `#dialog-box` with transparent canvas in [`dialogs.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-tui/src/agentic_coding_tui/ui/dialogs.py).
- **Truecolor Syntax Fidelity**:
  - Preserved 24-bit Truecolor typography (`ansi_color=False`) with zero solid background clipping boxes on Pygments syntax tokens.
- **Structural Demarcation**:
  - Replaced heavy container background boxes with crisp single-line dividers: `border-top: solid $primary;` on inputs and footers, `border: solid $accent;` on modal dialog borders.
  - Stripped decorative emojis across all status rows and headers for clean, minimalist typography.
- **Verification Suite**:
  - Expanded unit test coverage in [`tests/test_ui.py`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-coding-tui/tests/test_ui.py); verified 30/30 tests pass.

---

## [2026-11-02] — Multi-Repo Plugin Standardization & AC-8 Remediation

> [!NOTE] Release Summary: SPEC-2026-PLUGIN-RENAME-001 Merged across 5 Plugins
> - **Specification**: `SPEC-2026-PLUGIN-RENAME-001`
> - **Architecture Decision**: [[architecture/adr-004-standardize-plugin-repository-names|ADR-004: Standardize Plugin Repository Names]]
> - **Feature Note**: [[features/rename-plugin-repositories|Rename Plugin Repositories to Descriptive Action-Oriented Names]]
> - **Verification**: 135/135 automated unit tests passing across all plugins (100%).

### Merged Repositories & Commits
- [`agentic-prompt-triage-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-prompt-triage-plugin/): Commit `9c5c19b` (77/77 tests passed)
- [`agentic-project-documenting-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-documenting-plugin/): Commit `00b7d03` (10/10 tests passed)
- [`agentic-issue-investigation-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-issue-investigation-plugin/): Commit `8f46739` (7/7 tests passed)
- [`agentic-task-monitoring-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-task-monitoring-plugin/): Commit `76f7002` (8/8 tests passed)
- [`agentic-topic-researching-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-topic-researching-plugin/): Commit `ee5bfa0` (9/9 tests passed)

---

## [2026-11-01] — `agentic-project-grooming-plugin`: Interactive Scoping

> [!NOTE] Release Summary: Project Grooming Plugin & Skill (/project-grooming)
> - **Repository**: [`agentic-project-grooming-plugin`](file:///Users/gpfister/Development/Projects/gpfister/agentic/agentic-project-grooming-plugin/)
> - **Architecture Decision**: [[architecture/adr-003-project-grooming-plugin|ADR-003]]
> - **Feature Note**: [[features/project-grooming-plugin|Project Grooming Plugin]]
> - **Verification**: 8/8 automated unit tests passing (100%).
