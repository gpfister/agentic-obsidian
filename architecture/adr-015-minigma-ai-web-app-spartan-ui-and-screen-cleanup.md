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

## Context

Inspection of [`minigma-ai-web-app`](file:///Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app) (Angular `^19.0.0`, TypeScript `~5.6.0`, standalone components) identified four structural misalignments against the approved PenPot design specification ([[designs/minigma-ai-web-app-ui-alignment|Minigma AI Web App — UI Alignment & Screen Cleanup]], PenPot file `42288968-6e5e-808d-8008-fab5dcc739f5`):

1. **Legacy Component Library**: The application shell (`AppComponent`) and active screens (`LoginComponent`, `OnboardingComponent`, `ChatComponent`) depend on `@angular/material` (`MatToolbarModule`, `MatButtonModule`, `MatIconModule`, `--mat-sys-*` tokens, `mat-typography mat-app-background`, and Google Material Icons), which conflicts with the PenPot **spartan/ui + Tailwind CSS 4** component architecture.
2. **Unaligned Theme Tokens & Assets**: `src/styles.scss` relies on `@use '@angular/material' as mat;` rather than the dual-theme **Late Sun Engineering (Light)** and **Twilight Engineering (Dark)** CSS custom properties, `JetBrains Mono` / `Plus Jakarta Sans` typography scale, and theme-aware SVG logos/favicons (`src/assets/` is currently empty).
3. **Bloated Route Surface**: `src/app/app.routes.ts` exposes placeholder/legacy routes (`spec`, `code`, `ops`, `sandbox`, `user/account`, `admin/users` with `adminGuard`) and corresponding feature directories that are excluded from the approved product design.
4. **Worktree & Deployment Constraints**: The repository includes local Kubernetes/Terraform deployment tooling (`scripts/deployment_local.sh`, `Dockerfile`, `nginx.conf`, `terraform/main.tf`). Per [[architecture/adr-011-worktree-based-development-lifecycle|ADR-011]], primary `main` must remain clean while development and local deployment verification execute from an isolated worktree.

### Alternatives Considered

1. **Incremental Angular Material Theme Overrides**:
   - *Rejection Rationale*: Wrapping `@angular/material` M3 components to mimic `spartan/ui` introduces CSS specificity conflicts, retains unused Material bundle weight, and diverges from the PenPot component primitives (`HlmButton`, `HlmCard`, `HlmInput`, `HlmLabel`, `HlmSelect`, `HlmScrollArea`).
2. **Keep Legacy Routes Hidden Behind Feature Flags**:
   - *Rejection Rationale*: Leaving unmaintained `spec`, `code`, `ops`, `sandbox`, `user`, and `admin` feature modules in `src/app/features/` bloats bundle size, leaves dead navigation/guard code, and violates the approved screen cleanup specification.
3. **Full Migration to `spartan/ui` + Tailwind CSS 4, Dual-Theme Tokens, Route Reduction & Worktree Isolation (Selected)**:
   - *Rationale*: Establishes 1:1 parity with the PenPot design system, eliminates `@angular/material` overhead, reduces the active surface strictly to Auth, Onboarding, and Chat, and preserves `main` stability via `.worktrees/penpot-ui-alignment`.

---

## Decision

We adopt the following architectural decisions for `minigma-ai-web-app`:

1. **Phase Out `@angular/material` in Favor of `spartan/ui` + Tailwind CSS 4**:
   - Install Tailwind CSS 4 (`tailwindcss`, `@tailwindcss/postcss`, `postcss`) with `.postcssrc.json` and `@spartan-ng/brain` + Hlm UI primitives (`HlmButton`, `HlmCard`, `HlmInput`, `HlmLabel`, `HlmScrollArea`).
   - Remove `@angular/material` imports, `mat-*` elements/classes, and the Google Material Icons stylesheet from `AppComponent`, `LoginComponent`, `CallbackComponent`, `OnboardingComponent`, `ChatComponent`, `src/styles.scss`, and `src/index.html`.
2. **Dual-Theme Design Tokens (`Late Sun Engineering` & `Twilight Engineering`)**:
   - Configure `@import "tailwindcss";` and `@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));` in `src/styles.scss`.
   - Define all `--color-*`, `--font-mono` (`JetBrains Mono`), `--font-sans` (`Plus Jakarta Sans`), `--space-*`, and `--radius-*` custom properties for **Late Sun Engineering (Light)** on `:root, html, html[data-theme='light']` and **Twilight Engineering (Dark)** on `html[data-theme='dark']`.
   - Add theme-aware `src/assets/logo-light.svg`, `src/assets/logo-dark.svg`, and `prefers-color-scheme` favicons in `src/index.html`, with `"output": "assets"` in `angular.json`.
3. **Route Surface Reduction to Auth, Onboarding, and Chat**:
   - Restrict `src/app/app.routes.ts` strictly to `'' -> 'chat'`, `'auth/login'`, `'auth/callback'`, `'onboarding'` (`authGuard`), `'chat'` (`authGuard`, `orgGuard`), and `'**' -> 'chat'`.
   - Delete unused feature directories (`spec`, `code`, `ops`, `sandbox`, `user`, `admin`) and `src/app/core/auth/admin.guard.ts`.
   - Align `AppComponent` (`64px` `spartan-navbar` on `/chat` and `/onboarding` with `OrgService.selectOrganisation()`; floating `36×36px` circular theme toggle on `/auth/*`), `LoginComponent` (Login & Sign-Up modes), `CallbackComponent` (redirecting to `/chat`), `OnboardingComponent`, and `ChatComponent` (message groups, collapsible `AgentActivity`, `A2UI / ask_question` card, and `Composer`).
4. **Worktree Isolation & Worktree-Aware Local Deployment**:
   - Execute all implementation, build/test verification, and local deployment (`./scripts/deployment_local.sh` against Minikube profile `minigma-ai`) inside `.worktrees/penpot-ui-alignment` on `feature/penpot-ui-alignment`, keeping primary `main` untouched until `@coding-merge`.

---

## Consequences

- **Positive:**
  - Exact visual and token parity with PenPot (`42288968-6e5e-808d-8008-fab5dcc739f5`) across Light and Dark themes.
  - Smaller, cleaner codebase and reduced bundle footprint (`481.49 kB` initial total) by removing `@angular/material` and 6 unused feature modules.
  - Accessible, headless UI primitives via `@spartan-ng/brain` with full Tailwind CSS 4 utility composition.
  - Safe, isolated development and Minikube deployment verification (`http://app.localhost:8080/`, `minigma-ai-web-app:94f1f67-1795625897` in namespace `minigma-ai-web-app`) via `.worktrees/penpot-ui-alignment`.
- **Negative:**
  - Requires installing and configuring Tailwind CSS 4 PostCSS pipeline and `spartan/ui` Hlm primitives compatible with Angular 19.
  - Existing unit specs (`app.component.spec.ts`, `login.component.spec.ts`, `onboarding.component.spec.ts`) required updates to remove Material test harnesses and add `chat.component.spec.ts`; `npm test` is not yet configured in `angular.json`.
- **Risks & Mitigations:**
  - *Risk*: Bookmarked legacy URLs (`/spec`, `/code`, `/ops`, `/sandbox`, `/user/account`, `/admin/users`) after route removal.
  - *Mitigation*: Wildcard `'**'` route safely redirects to `/chat` (enforcing `authGuard` and `orgGuard`).
  - *Risk*: `angular.json` bundle budget overruns (`initial` ≤ 1MB error / 500kB warning, `anyComponentStyle` ≤ 4kB error / 2kB warning).
  - *Mitigation*: Removing `@angular/material` and unused feature modules offsets Tailwind/spartan additions; verified by `npm run build` (`481.49 kB`).

---

## Post-Merge Implementation & Release Addendum (`49360d2`)

- `angular.json` updated with `"output": "assets"` on `src/assets` so SVG logos and favicons resolve under `/assets/*`.
- `OrgService.selectOrganisation(id: string)` added in `src/app/core/services/org.service.ts` to support the `AppComponent` Organisation Switcher.
- `CallbackComponent` (`src/app/features/auth/callback/callback.component.{ts,html}`) migrated from `@angular/material` to `HlmCardDirective` / `HlmButtonDirective` with `/user/account` replaced by `/chat`.
- [PR #4](https://github.com/gpfister/minigma-ai-web-app/pull/4) squash-merged into `main` (`49360d2`); `.worktrees/penpot-ui-alignment` removed and pruned; local and remote `feature/penpot-ui-alignment` deleted; primary repository clean on `main`.
