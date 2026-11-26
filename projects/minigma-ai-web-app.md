---
title: "Minigma AI Web App (`minigma-ai-web-app`)"
date: "2026-11-25"
phase: "Phase 7: Merge & Release Complete"
status: "Merged / Completed"
repository: "/Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app"
branch: "main"
commit: "49360d2"
pr: "https://github.com/gpfister/minigma-ai-web-app/pull/4"
tags:
  - projects
  - minigma-ai
  - minigma-ai-web-app
  - angular-19
  - spartan-ui
  - tailwind-v4
  - merged
  - completed
---

# Minigma AI Web App (`minigma-ai-web-app`)

> [!NOTE]
> - **Repository**: [`minigma-ai-web-app`](file:///Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app) (clean on `main` @ `49360d2`)
> - **Merged PR**: [PR #4](https://github.com/gpfister/minigma-ai-web-app/pull/4) (`feature/penpot-ui-alignment` -> `main`, squash-merged at `49360d2`; `.worktrees/penpot-ui-alignment` removed & pruned; `feature/penpot-ui-alignment` deleted locally & remotely)
> - **Feature Spec**: [[features/minigma-ai-web-app-ui-alignment|Minigma AI Web App — UI Alignment & Screen Cleanup]]
> - **Architecture Plan**: [[architecture/minigma-ai-web-app-ui-alignment|Architecture Plan]]
> - **Design Spec**: [[designs/minigma-ai-web-app-ui-alignment|Design Spec]]
> - **ADR**: [[architecture/adr-015-minigma-ai-web-app-spartan-ui-and-screen-cleanup|ADR-015]]

---

## 1. Overview & Tech Stack

[`minigma-ai-web-app`](file:///Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app) is the Angular `^19.0.0` (TypeScript `~5.6.0`) frontend web application for Minigma AI. Following [PR #4](https://github.com/gpfister/minigma-ai-web-app/pull/4) (`49360d2`), the application uses:

- **UI Primitives & Styling**: `spartan/ui` (`@spartan-ng/brain` + local Hlm primitives in `src/app/shared/ui/`) and Tailwind CSS 4 (`tailwindcss`, `@tailwindcss/postcss`, `postcss` via `.postcssrc.json`), with `@angular/material` phased out from all application source code.
- **Dual-Theme Design System**: **Late Sun Engineering (Light)** and **Twilight Engineering (Dark)** CSS custom properties (`--color-*`, `--font-mono`, `--font-sans`, `--space-*`, `--radius-*`) in `src/styles.scss`, synchronized by `ThemeService` (`html[data-theme="light" | "dark"]`, `localStorage['minigma-theme']`).
- **Typography & Icons**: `JetBrains Mono`, `Plus Jakarta Sans`, and FontAwesome (`@fortawesome/angular-fontawesome`).
- **Brand Assets**: `src/assets/logo-light.svg`, `src/assets/logo-dark.svg`, `favicon-light-32.svg`, `favicon-dark-32.svg`, `favicon-16.svg` emitted via `angular.json` (`"output": "assets"`).

---

## 2. Application Structure (`src/app/`)

- **Shell & Routing**:
  - `src/app/app.component.{ts,html,scss,spec.ts}`: `64px` `spartan-navbar` on `/chat` and `/onboarding` (with `HlmSelect` Organisation Switcher on `/chat`, Theme Toggle, and User Profile Menu with `Logout`), plus floating `36×36px` circular Theme Toggle on `/auth/*`.
  - `src/app/app.routes.ts`: Restricted strictly to `'' -> 'chat'`, `'auth/login'`, `'auth/callback'`, `'onboarding'` (`authGuard`), `'chat'` (`authGuard`, `orgGuard`), and `'**' -> 'chat'`.
- **Core Services & Guards (`src/app/core/`)**:
  - `auth/`: `AuthService`, `authGuard`, `authInterceptor`, `auth.models.ts`.
  - `org/`: `orgGuard` (`org.guard.ts`).
  - `services/`: `OrgService` (`org.service.ts`, including `selectOrganisation(id: string)`), `UserService` (`user.service.ts`).
  - `theme/`: `ThemeService` (`theme.service.ts`).
- **Features (`src/app/features/`)**:
  - `auth/login/`: `LoginComponent` (`460px` `HlmCard`, Login & Sign-Up modes).
  - `auth/callback/`: `CallbackComponent` (`HlmCardDirective` / `HlmButtonDirective`, redirects to `/chat`).
  - `onboarding/`: `OnboardingComponent` (`460px` `HlmCard` wired to `OrgService`).
  - `chat/`: `ChatComponent` (`900px` `chat-container`, `HlmScrollArea`, message groups, collapsible `AgentActivity`, `A2UI / ask_question` card, `Composer`).
- **Shared UI (`src/app/shared/ui/`)**:
  - `hlm-button.directive.ts`, `hlm-card.directives.ts`, `hlm-input.directive.ts`, `hlm-label.directive.ts`, `hlm-scroll-area.component.ts`, `hlm-utils.ts`, `index.ts`.

---

## 3. Local Kubernetes Deployment

- **Deployment Script**: [`scripts/deployment_local.sh`](file:///Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app/scripts/deployment_local.sh) (worktree-aware via `GIT_COMMON_DIR` for shared `terraform/terraform.tfstate`).
- **Container & Ingress**: [`Dockerfile`](file:///Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app/Dockerfile) (Node 22 + `nginx:alpine`), [`nginx.conf`](file:///Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app/nginx.conf), [`terraform/main.tf`](file:///Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app/terraform/main.tf) (Deployment, Service, and Traefik `IngressRoute` for `Host('app.localhost')`).
- **Active Local Deployment**: Running on `http://app.localhost:8080/` (`minigma-ai-web-app:94f1f67-1795625897` in namespace `minigma-ai-web-app`).
