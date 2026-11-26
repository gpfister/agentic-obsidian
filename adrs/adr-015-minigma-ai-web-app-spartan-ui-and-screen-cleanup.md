---
adr_id: "015"
title: "Minigma AI Web App: spartan/ui + Tailwind CSS 4 Migration, Dual-Theme Tokens, and Route Surface Reduction"
status: "accepted"
date: "2026-11-25"
specification_path: "/Users/gpfister/.gemini/antigravity/brain/b334efa9-f904-44c2-a445-bbfd30f309da/specification.md"
plan_path: "/Users/gpfister/.gemini/antigravity/brain/b334efa9-f904-44c2-a445-bbfd30f309da/plan.md"
design_spec_path: "/Users/gpfister/.gemini/antigravity/brain/eff05b10-f717-4f08-806d-bd059eb10d54/design-spec.md"
review_path: "/Users/gpfister/.gemini/antigravity/brain/b334efa9-f904-44c2-a445-bbfd30f309da/review-penpot-ui-alignment.md"
target_repository: "/Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app"
pr: "https://github.com/gpfister/minigma-ai-web-app/pull/4"
commit: "49360d2"
tags:
  - architecture-decision
  - minigma-ai-web-app
  - spartan-ui
  - tailwind-v4
  - angular-19
  - design-tokens
  - worktree-isolation
  - merged
  - completed
aliases:
  - ADR-015
  - adr-015-minigma-ai-web-app-spartan-ui-and-screen-cleanup
---

# ADR-015: Minigma AI Web App: spartan/ui + Tailwind CSS 4 Migration, Dual-Theme Tokens, and Route Surface Reduction

See canonical record: [[architecture/adr-015-minigma-ai-web-app-spartan-ui-and-screen-cleanup|ADR-015]].

## Summary

- **Phase Out `@angular/material`**: Migrate `AppComponent`, `LoginComponent`, `CallbackComponent`, `OnboardingComponent`, `ChatComponent`, `styles.scss`, and `index.html` to `spartan/ui` (`@spartan-ng/brain` + Hlm primitives) and Tailwind CSS 4.
- **Dual-Theme Tokens**: Adopt **Late Sun Engineering (Light)** and **Twilight Engineering (Dark, `html[data-theme="dark"]`)** color, typography (`JetBrains Mono`, `Plus Jakarta Sans`), spacing, and radii tokens, plus theme-aware SVG logos and favicons (`angular.json` `"output": "assets"`).
- **Route Surface Reduction**: Retain only `'' -> 'chat'`, `'auth/login'`, `'auth/callback'`, `'onboarding'`, `'chat'`, and `'**' -> 'chat'`; remove `spec`, `code`, `ops`, `sandbox`, `user`, `admin`, and `adminGuard`; add `OrgService.selectOrganisation(id)`.
- **Merge & Deployment Verification**: Squash-merged into `main` via [PR #4](https://github.com/gpfister/minigma-ai-web-app/pull/4) (`49360d2`); `.worktrees/penpot-ui-alignment` removed & pruned; `feature/penpot-ui-alignment` deleted locally & remotely; `npm run build` `481.49 kB`; active locally at `http://app.localhost:8080/` (`minigma-ai-web-app:94f1f67-1795625897` in namespace `minigma-ai-web-app`).
