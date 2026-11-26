---
title: "Workspace Projects Index"
date: "2026-11-25"
tags:
  - projects
  - index
  - architecture
---

# Workspace Projects Index

Directory of standalone projects, packages, and federated services maintained within the `agentic` workspace.

| Project Note | Phase | Status | Summary |
| :--- | :--- | :--- | :--- |
| [[projects/minigma-ai-web-app\|minigma-ai-web-app]] | Phase 7: Merge & Release Complete | Merged / Completed | Angular 19 frontend web application for Minigma AI. Aligned with the PenPot **spartan/ui + Tailwind CSS 4** design system (`42288968-6e5e-808d-8008-fab5dcc739f5`) across **Late Sun Engineering (Light)** and **Twilight Engineering (Dark)** themes, with `@angular/material` phased out and routes reduced to **Auth**, **Onboarding**, and **Chat**. Squash-merged into `main` via [PR #4](https://github.com/gpfister/minigma-ai-web-app/pull/4) (`49360d2`), `.worktrees/penpot-ui-alignment` removed & pruned, `feature/penpot-ui-alignment` deleted locally & remotely, and active locally at `http://app.localhost:8080/` (`minigma-ai-web-app:94f1f67-1795625897` in namespace `minigma-ai-web-app`). |
| [[projects/agentic-project-grooming-plugin\|agentic-project-grooming-plugin]] | Phase 6: Merge & Release | Active / Released | Standalone Antigravity companion plugin providing the `/project-grooming` skill for upfront scoping, Matt Pocock style grilling across 7 dimensions, interactive approval gate, and cross-plugin delegation. Merged into `master` across all repositories (`2bbd11f`, `c6fd560`, `d2e11c5`), feature branches deleted, and all 117/117 unit tests passing. |
