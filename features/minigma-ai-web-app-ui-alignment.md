---
title: "Minigma AI Web App — UI Alignment & Screen Cleanup"
date: "2026-11-25"
phase: "Phase 7: Merge & Release Complete"
status: "Merged / Completed"
target_repository: "/Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app"
target_branch: "main"
pr: "https://github.com/gpfister/minigma-ai-web-app/pull/4"
commit: "49360d2"
adr: "[[architecture/adr-015-minigma-ai-web-app-spartan-ui-and-screen-cleanup|ADR-015]]"
architecture_plan: "[[architecture/minigma-ai-web-app-ui-alignment|Architecture Plan]]"
design_spec: "[[designs/minigma-ai-web-app-ui-alignment|Design Specification]]"
review_artifact: "/Users/gpfister/.gemini/antigravity/brain/b334efa9-f904-44c2-a445-bbfd30f309da/review-penpot-ui-alignment.md"
tags:
  - features
  - minigma-ai-web-app
  - spartan-ui
  - tailwind-v4
  - angular-19
  - merged
  - completed
---

# Minigma AI Web App — UI Alignment & Screen Cleanup

> [!NOTE]
> - **Status**: **Merged / Completed** — [PR #4](https://github.com/gpfister/minigma-ai-web-app/pull/4) squash-merged into `main` (`49360d2`); `.worktrees/penpot-ui-alignment` removed & pruned; `feature/penpot-ui-alignment` deleted locally & remotely; active local deployment on `http://app.localhost:8080/` (`minigma-ai-web-app:94f1f67-1795625897` in namespace `minigma-ai-web-app`).
> - **Architecture Plan**: [[architecture/minigma-ai-web-app-ui-alignment|Architecture Plan — Minigma AI Web App UI Alignment & Screen Cleanup]]
> - **ADR**: [[architecture/adr-015-minigma-ai-web-app-spartan-ui-and-screen-cleanup|ADR-015]]
> - **Design Spec**: [[designs/minigma-ai-web-app-ui-alignment|Design Spec — Minigma AI Web App UI Alignment & Screen Cleanup]]
> - **Project Note**: [[projects/minigma-ai-web-app|Minigma AI Web App]]
> - **Specification Artifact**: [`specification.md`](file:///Users/gpfister/.gemini/antigravity/brain/b334efa9-f904-44c2-a445-bbfd30f309da/specification.md)
> - **Implementation Plan Artifact**: [`plan.md`](file:///Users/gpfister/.gemini/antigravity/brain/b334efa9-f904-44c2-a445-bbfd30f309da/plan.md)
> - **QA Review Artifact**: [`review-penpot-ui-alignment.md`](file:///Users/gpfister/.gemini/antigravity/brain/b334efa9-f904-44c2-a445-bbfd30f309da/review-penpot-ui-alignment.md)

---

## 1. Summary

Aligns [`minigma-ai-web-app`](file:///Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app) with the approved **spartan/ui + Tailwind CSS 4** PenPot design system (`42288968-6e5e-808d-8008-fab5dcc739f5`) across **Late Sun Engineering (Light)** and **Twilight Engineering (Dark)** themes, removes all non-designed screens, and phases out `@angular/material`.

Implemented on `feature/penpot-ui-alignment` (`94f1f67`, 48 files changed: +1,251 / -1,645 lines) inside `.worktrees/penpot-ui-alignment`, verified via local Kubernetes deployment (`http://app.localhost:8080/`, image `minigma-ai-web-app:94f1f67-1795625897` in namespace `minigma-ai-web-app`), and squash-merged into `main` via [PR #4](https://github.com/gpfister/minigma-ai-web-app/pull/4) (`49360d2`).

---

## 2. Functional Requirements Summary

- **FR-1 (Route & Feature Cleanup)**: Remove `spec`, `code`, `ops`, `sandbox`, `user/account`, `admin/users`, and `adminGuard` from `src/app/app.routes.ts` and delete their feature directories. Retain only `'' -> 'chat'`, `'auth/login'`, `'auth/callback'`, `'onboarding'` (`authGuard`), `'chat'` (`authGuard`, `orgGuard`), and `'**' -> 'chat'`.
- **FR-2 (Design Tokens, Tailwind CSS 4 & `spartan/ui`)**: Configure Tailwind CSS 4 and `spartan/ui` (`HlmButton`, `HlmCard`, `HlmInput`, `HlmLabel`, `HlmSelect`, `HlmScrollArea`, popover/menu); define all Late Sun (Light) and Twilight (Dark, `html[data-theme="dark"]`) CSS custom properties, typography (`JetBrains Mono`, `Plus Jakarta Sans`), spacing, and radii in `src/styles.scss`; remove `@angular/material` and `mat-*` classes.
- **FR-3 (App Shell & `spartan-navbar`)**: Render `64px` `spartan-navbar` on `/chat` and `/onboarding` (theme-aware SVG logo + `Minigma AI`, `HlmSelect` Organisation Switcher on `/chat` only, `36×36px` Theme Toggle, User Profile Menu with `Logout`); hide `spartan-navbar` on `/auth/*` and render floating `36×36px` circular Theme Toggle.
- **FR-4 (Auth Screen `/auth/login`)**: Centered `460px` `HlmCard` supporting Login (`Email Address`) and Sign-Up (`First Name`, `Last Name`, `Email Address`) modes, mode-switch link, and `"Send Magic Link"` submit wired to `AuthService`.
- **FR-5 (Onboarding Screen `/onboarding`)**: Centered `460px` `HlmCard` with `"Create your organisation"`, `Organisation Name` field (`"e.g. Acme Corp"`), and `"Continue to Workspace"` wired to `OrgService`.
- **FR-6 (Chat Screen `/chat`)**: `900px` `chat-container` with `HlmScrollArea` (`MessageGroup / Agent`, `MessageGroup / User`, collapsible `AgentActivity`), `A2UI / ask_question` card, and `Composer` with `Agent Selector` (`Router Agent`) and right-arrow Send button.
- **FR-7 (Brand Assets & Favicons)**: Add `src/assets/logo-light.svg`, `src/assets/logo-dark.svg`, and `prefers-color-scheme` favicons in `src/index.html`.

---

## 3. Approved Unplanned Changes (`94f1f67`)

During implementation and QA review ([`review-penpot-ui-alignment.md`](file:///Users/gpfister/.gemini/antigravity/brain/b334efa9-f904-44c2-a445-bbfd30f309da/review-penpot-ui-alignment.md)), three unplanned changes were introduced and approved:

1. **`angular.json` Asset Output Config**: Added `"output": "assets"` to the `src/assets` asset entry so `logo-light.svg`, `logo-dark.svg`, and favicons emit to `dist/minigma-ai-web-app/browser/assets/` and are served at `/assets/*`.
2. **`OrgService.selectOrganisation(id: string)`**: Added `selectOrganisation(id: string)` in `src/app/core/services/org.service.ts` to update `activeOrganisationSignal` from `AppComponent`'s Organisation Switcher.
3. **`CallbackComponent` Alignment**: Migrated `src/app/features/auth/callback/callback.component.{ts,html}` from `@angular/material` (`MatCardModule`, `MatButtonModule`, `MatProgressSpinnerModule`, `MatIconModule`) to `HlmCardDirective` / `HlmButtonDirective` and replaced `/user/account` navigation/links with `/chat`.

---

## 4. Non-Functional Requirements & Acceptance Criteria

- **NFR-1**: `npm run build` passes within `angular.json` budgets (`initial` ≤ 1MB error / 500kB warning; `anyComponentStyle` ≤ 4kB error / 2kB warning). Verified: `Initial total: 481.49 kB` (0 errors).
- **NFR-2**: `ThemeService` synchronizes `html[data-theme="light" | "dark"]` and `localStorage['minigma-theme']`.
- **NFR-3**: Accessible labels and keyboard navigation via `@spartan-ng/brain`.
- **NFR-4**: Isolated Git worktree `.worktrees/penpot-ui-alignment` on `feature/penpot-ui-alignment` (removed & pruned post-merge; primary repo clean on `main` at `49360d2`).

### Acceptance Criteria Checklist
- [x] **AC-1**: `app.routes.ts` restricted to Auth, Onboarding, Chat; unused feature directories removed.
- [x] **AC-2**: `spartan/ui` + Tailwind CSS 4 adopted with zero `@angular/material` or `mat-*` usage (`rg` returns 0 matches).
- [x] **AC-3**: Light/Dark CSS tokens match `design-spec.md`.
- [x] **AC-4**: `spartan-navbar` and floating auth theme toggle behave per spec.
- [x] **AC-5**: `/auth/login` Login & Sign-Up modes match `460px` `HlmCard` spec (plus `CallbackComponent` aligned).
- [x] **AC-6**: `/onboarding` matches `460px` `HlmCard` spec.
- [x] **AC-7**: `/chat` matches `900px` `chat-container` spec.
- [x] **AC-8**: `logo-light.svg`, `logo-dark.svg`, and favicons wired in `src/index.html` and emitted to `/assets/*`.
- [x] **AC-9**: `npx tsc -p tsconfig.app.json --noEmit` and `npm run build` pass with zero errors (`npm test` not configured in `angular.json`).
