---
title: "Minigma AI Web App — UI Alignment & Screen Cleanup"
date: "2026-11-25"
status: "Approved"
design_system: "spartan/ui + Tailwind CSS 4"
themes:
  - "Late Sun Engineering (Light)"
  - "Twilight Engineering (Dark)"
penpot_file_name: "minigma-ai-web-app"
penpot_file_id: "42288968-6e5e-808d-8008-fab5dcc739f5"
target_repository: "/Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app"
source_artifact: "/Users/gpfister/.gemini/antigravity/brain/40cd4ed6-0f8b-410e-8e0b-3ba0c88d1868/design-spec.md"
screens:
  - "Auth"
  - "Onboarding"
  - "Chat"
tags:
  - designs
  - penpot
  - minigma-ai
  - minigma-ai-web-app
  - spartan-ui
  - tailwind-v4
---

# Minigma AI Web App — UI Alignment & Screen Cleanup

> [!NOTE]
> - **Index**: [[designs/index|Design Specifications Index]]
> - **Target Application Repository**: [`minigma-ai-web-app`](file:///Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app) (`/Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app`)
> - **Source Artifact**: [`design-spec.md`](file:///Users/gpfister/.gemini/antigravity/brain/40cd4ed6-0f8b-410e-8e0b-3ba0c88d1868/design-spec.md)
> - **Source of Truth**: PenPot file `minigma-ai-web-app` (`42288968-6e5e-808d-8008-fab5dcc739f5`)

---

## 1. Overview & PenPot References

This specification aligns [`minigma-ai-web-app`](file:///Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app) with the **spartan/ui + Tailwind CSS 4** PenPot design system across both **Late Sun Engineering (Light)** and **Twilight Engineering (Dark)** themes, and removes all non-designed screens (`spec`, `code`, `ops`, `sandbox`, `user/account`, `admin/users`) so that only **Auth** (`auth/login` with Login/Sign-Up modes + `auth/callback`), **Onboarding** (`onboarding`), and **Chat** (`chat`) remain.

### PenPot File, Page & Artboard IDs

- **PenPot File**: `minigma-ai-web-app` (`42288968-6e5e-808d-8008-fab5dcc739f5`)

| Page | Page ID | Artboard / Frame | Artboard ID | Dimensions |
| :--- | :--- | :--- | :--- | :--- |
| `Design System & Tokens` | `42288968-6e5e-808d-8008-fab5dcc739f6` | `Color Palette — Late Sun Engineering (Light)` | `b16445c8-998f-80d4-8008-fb616f9c2dd8` | `1440×1140` |
| `Design System & Tokens` | `42288968-6e5e-808d-8008-fab5dcc739f6` | `Color Palette — Twilight Engineering (Dark)` | `b16445c8-998f-80d4-8008-fb6173a5e3a3` | `1440×1140` |
| `Design System & Tokens` | `42288968-6e5e-808d-8008-fab5dcc739f6` | `Typography, Spacing & Radii` | `b16445c8-998f-80d4-8008-fb61790a376a` | `1200×1020` |
| `App Shell & Components` | `b16445c8-998f-80d4-8008-fb61c9ed21a5` | `Component Library — spartan / ui + Tailwind CSS 4 (Light)` | `2e54129a-fea9-8080-8009-000551769d62` | `1520×1007` |
| `Screens — Light (Late Sun Engineering)` | `b16445c8-998f-80d4-8008-fb624216febc` | `Auth — Login` | `2e54129a-fea9-8080-8009-000431392eaa` | `1440×900` |
| `Screens — Light (Late Sun Engineering)` | `b16445c8-998f-80d4-8008-fb624216febc` | `Auth — Sign-Up` | `2e54129a-fea9-8080-8009-0004319e7586` | `1440×900` |
| `Screens — Light (Late Sun Engineering)` | `b16445c8-998f-80d4-8008-fb624216febc` | `Onboarding — Create Organisation` | `2e54129a-fea9-8080-8009-0004322f523d` | `1440×900` |
| `Screens — Light (Late Sun Engineering)` | `b16445c8-998f-80d4-8008-fb624216febc` | `Chat — Router Agent` | `2e54129a-fea9-8080-8009-0004328d58d4` | `1440×900` |
| `Screens — Dark (Twilight Engineering)` | `b16445c8-998f-80d4-8008-fb629034344e` | `Auth — Login (Dark)` | `2e54129a-fea9-8080-8009-000541ce69b5` | `1440×900` |
| `Screens — Dark (Twilight Engineering)` | `b16445c8-998f-80d4-8008-fb629034344e` | `Auth — Sign-Up (Dark)` | `2e54129a-fea9-8080-8009-00054211bca9` | `1440×900` |
| `Screens — Dark (Twilight Engineering)` | `b16445c8-998f-80d4-8008-fb629034344e` | `Onboarding — Create Organisation (Dark)` | `2e54129a-fea9-8080-8009-0005427178cf` | `1440×900` |
| `Screens — Dark (Twilight Engineering)` | `b16445c8-998f-80d4-8008-fb629034344e` | `Chat — Router Agent (Dark)` | `2e54129a-fea9-8080-8009-000542d3960d` | `1440×900` |
| `Logos — Light (Late Sun Engineering)` | `2e54129a-fea9-8080-8009-001e869cbc24` | `logo-light` | `2e54129a-fea9-8080-8009-001eb2073ccb` | `512×512` |
| `Logo — Dark (Twilight Engineering)` | `2e54129a-fea9-8080-8009-001ebefd1f02` | `logo-dark` | `2e54129a-fea9-8080-8009-001ee61a187d` | `512×512` |

---

## 2. Design Tokens

### 2.1 Color Palettes (Light & Dark)

| Token | Light (`Late Sun Engineering`) | Dark (`Twilight Engineering`, `html[data-theme="dark"]`) |
| :--- | :--- | :--- |
| `--color-primary` | `#A13A00` | `#FFB597` |
| `--color-on-primary` | `#FFFFFF` | `#591D00` |
| `--color-primary-container` | `#C64F13` | `#E36F3A` |
| `--color-on-primary-container` | `#FFFBFF` | `#4E1800` |
| `--color-secondary` | `#835500` | `#B0C7F4` |
| `--color-on-secondary` | `#FFFFFF` | `#183055` |
| `--color-secondary-container` | `#FEB23E` | `#32496F` |
| `--color-on-secondary-container` | `#6E4700` | `#A2B9E5` |
| `--color-tertiary` | `#375AA1` | `#FFB95F` |
| `--color-on-tertiary` | `#FFFFFF` | `#472A00` |
| `--color-tertiary-container` | `#5273BB` | `#CA8100` |
| `--color-on-tertiary-container` | `#FEFCFF` | `#3E2400` |
| `--color-surface` | `#FFF8F2` | `#10141A` |
| `--color-surface-container-lowest` | `#FFFFFF` | `#0A0E14` |
| `--color-surface-container-low` | `#FCF2E6` | `#181C22` |
| `--color-surface-container` | `#F6ECE1` | `#1C2026` |
| `--color-surface-container-high` | `#F0E7DB` | `#262A31` |
| `--color-surface-container-highest` | `#EAE1D5` | `#31353C` |
| `--color-surface-dim` | `#E2D9CD` | `#10141A` |
| `--color-surface-bright` | `#FFF8F2` | `#353940` |
| `--color-on-surface` | `#1F1B14` | `#DFE2EB` |
| `--color-on-surface-variant` | `#584239` | `#DEC0B5` |
| `--color-outline` | `#8C7167` | `#A58B81` |
| `--color-outline-variant` | `#DFC0B4` | `#57423A` |
| `--color-error` | `#BA1A1A` | `#FFB4AB` |
| `--color-on-error` | `#FFFFFF` | `#690005` |
| `--color-error-container` | `#FFDAD6` | `#93000A` |
| `--color-on-error-container` | `#93000A` | `#FFDAD6` |
| `--color-inverse-surface` | `#343028` | `#DFE2EB` |
| `--color-inverse-on-surface` | `#F9EFE3` | `#2D3137` |
| `--color-inverse-primary` | `#FFB598` | `#A23F0A` |
| `--color-surface-tint` | `#A53C00` | `#FFB597` |

### 2.2 Typography Scale

- **Headings / Brand / Code**: `JetBrains Mono` (`--font-mono`)
- **Body / Controls / Labels**: `Plus Jakarta Sans` (`--font-sans`)

| Role | Font Family | Size | Weight |
| :--- | :--- | :--- | :--- |
| H1 / Page Title | `JetBrains Mono` | `28px` | `700` |
| H2 / Section Title | `JetBrains Mono` | `22px` | `600` |
| H3 / Card Title | `JetBrains Mono` | `18px`–`20px` | `600` |
| Brand / Header Logo | `JetBrains Mono` | `18px`–`20px` (Auth: `22px`) | `600` |
| Body / Regular | `Plus Jakarta Sans` | `16px` | `400` |
| Body / Small / Controls | `Plus Jakarta Sans` | `14px` | `400` / `500` |
| Code / Inline / Activity | `JetBrains Mono` | `12px`–`14px` | `400` / `500` / `600` |
| Badges / Timestamps | `JetBrains Mono` | `11px` | `400` / `500` |

### 2.3 Spacing & Border Radii

- **Spacing**: `--space-xs: 0.25rem (4px)`, `--space-sm: 0.5rem (8px)`, `--space-md: 1rem (16px)`, `--space-lg / --gutter: 1.5rem (24px)`, `--margin: 2rem (32px)`, `--space-xl: 2.5rem (40px)`
- **Radii**: `--radius-sm: 4px`, `--radius-default: 8px`, `--radius-md: 12px`, `--radius-lg: 16px`, `--radius-xl: 24px`, `--radius-full: 9999px`

---

## 3. Component Hierarchy & Screen List

Target screens retained: **Auth**, **Onboarding**, and **Chat**.

### 3.1 App Shell & `spartan-navbar`
- **Height**: `64px`, full width, background `--color-surface-container-low` (`#FCF2E6` light / `#181C22` dark), bottom `1px` border `--color-outline-variant`.
- **Left (`brand`)**: Horizontal flex (`gap: 10px`) with theme-aware Minigma AI SVG logo (`logo-light` / `logo-dark`) + `Minigma AI` (`JetBrains Mono 18px/600`).
- **Right (`navbar-actions`)**: Horizontal flex (`gap: 10px`):
  - **Multi-org Chat**: `HlmSelect / Organisation Switcher` (building icon + org name + chevron-down) opening popover list of user organisations.
  - **Theme Toggle (`HlmButton` ghost)**: `36×36px` icon button toggling Light/Dark theme (`moon` / `sun`).
  - **User Profile Menu Trigger (`HlmButton` ghost)**: User icon opening popover with **Logout** action (no Account/Admin links).
  - **Onboarding**: Navbar omits Organisation Switcher (single/no-org state) and keeps Brand + Theme Toggle + User Profile Menu (Logout).

### 3.2 Screen 1 & 2: `Auth` (`Auth — Login` & `Auth — Sign-Up`, `/auth/login`, `/auth/callback`)
- **Canvas**: Full viewport (`1440×900` reference), centered column flex, background `--color-surface` (`#FFF8F2` / `#10141A`).
- **Floating Theme Toggle**: `36×36px` circular (`radius: 9999px`) button, `--color-surface-container-lowest` fill, `1px` `--color-outline-variant` stroke, top-right floating.
- **`HlmCard / Auth Card`**: `460px` width, `radius: 12px`, `--color-surface-container-lowest` fill, `1px` `--color-outline-variant` stroke, vertical flex (`gap: 20px`):
  - `brand`: `logo-light` / `logo-dark` + `Minigma AI` (`JetBrains Mono 22px/600`, `--color-primary`).
  - `auth-form` (`gap: 16px`):
    - **Login mode**: `HlmField / Email Address` (`HlmLabel` + `HlmInput`).
    - **Sign-Up mode**: `First Name`, `Last Name`, and `Email Address` fields.
  - `mode-switch-link` (`gap: 6px`): `"Don't have an account?"` + `"Sign-Up"` (or `"Already have an account?"` + `"Login"`) in `--color-primary` `600`.
  - `HlmButton` primary submit: `"Send Magic Link"` (or Sign-Up submit).

### 3.3 Screen 3: `Onboarding` (`Onboarding — Create Organisation`, `/onboarding`)
- **Shell**: `spartan-navbar` (without Organisation Switcher) + centered `HlmCard`.
- **Card**: `HlmCardTitle` `"Create your organisation"` (`JetBrains Mono 20px/600`), `HlmCardDescription` `"Set up your workspace to start collaborating in Minigma AI."`, `HlmField / Organisation Name` (`placeholder: "e.g. Acme Corp"`, hint `"You can rename your organisation later in settings."`), primary `HlmButton` `"Continue to Workspace"`.

### 3.4 Screen 4: `Chat` (`Chat — Router Agent`, `/chat`)
- **Container**: `minigma-content` (`padding: 24px`, centered, `--color-surface-container-lowest` or `--color-surface`) wrapping `chat-container` (`max-width: 900px`, vertical flex `justify-between`, `gap: 16px`).
- **`HlmScrollArea / Messages`** (`gap: 14px`):
  - `MessageGroup / Agent` (`align: start`, `gap: 4px`): `sender-label` (`Router Agent`, `JetBrains Mono 12px/600`) -> `MessageBubble / Agent` (`radius: 10px`, `--color-surface-container`, `1px` `--color-outline-variant` stroke) -> `agent-stats-timestamp` (`JetBrains Mono 11px/400`, e.g., `0.8s · 72.4 tok/s · 14:21`).
  - `MessageGroup / User` (`align: end`, `gap: 4px`): `sender-label` (`You`) -> `MessageBubble / User` (`radius: 10px`, `--color-primary-container` `#C64F13` / `#E36F3A`, `--color-on-primary-container`) -> `user-timestamp` (`JetBrains Mono 11px/400`).
  - `AgentActivity / Collapsible`: Compact monospace (`JetBrains Mono 12px/400`) trigger (`Thinking (1.2s)`, `Calling ask_question`) with expandable `activity-details` JSON block.
- **`chat-bottom-dock`** (`gap: 12px`):
  - **`A2UI / ask_question`** (when active): `radius: 12px`, `padding: 16px`, `--color-surface`, `1px` `--color-outline-variant` stroke, `gap: 10px`: header with `question-text` + `a2ui-badge` (`ask_question`, `JetBrains Mono 11px/500`), and radio-style `ask-options` list (including `Other (...)`).
  - **`Composer / Rounded Square Box`**: `radius: 14px`, `min-height: 96px`, `padding: 14px 16px`, `--color-surface`, `1px` `--color-outline-variant` stroke, vertical flex `justify-between` (`gap: 12px`):
    - Textarea/input (`placeholder: "Ask or instruct Minigma agents..."`)
    - `composer-toolbar`: Left `HlmSelect / Agent Selector` (`Router Agent` + chevron), Right `HlmButton / Send (Right Arrow)` (`arrow-right`).

### 3.5 Brand Assets (`logo-light` & `logo-dark` + Favicons)
- Exported from PenPot `512×512` `logo-light` (`2e54129a-fea9-8080-8009-001eb2073ccb`) and `logo-dark` (`2e54129a-fea9-8080-8009-001ee61a187d`) robot mascot SVGs.
- Favicons (`32×32` Light/Dark, `16×16` Light, `64×64` Apple Touch) defined in `App Shell & Components` (`favicons-row`).

---

## 4. Implementation Guidance (for `@coding-develop`)

> [!IMPORTANT]
> Apply all changes in [`/Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app`](file:///Users/gpfister/Development/Projects/gpfister/minigma-ai/minigma-ai-web-app) using an isolated worktree.

1. **Remove Unnecessary Screens & Routes**:
   - In `src/app/app.routes.ts`, remove routes and imports for `spec`, `code`, `ops`, `sandbox`, `user/account`, and `admin/users`.
   - Keep only `'' -> 'chat'`, `auth/login`, `auth/callback`, `onboarding`, `chat`, and `'**' -> 'chat'`.
   - Remove the corresponding feature directories (`src/app/features/spec`, `code`, `ops`, `sandbox`, `user`, `admin`) and `adminGuard` if unused, ensuring no broken imports or dead navigation links remain.
2. **Adopt `spartan/ui` (`@spartan-ng/brain` / `@spartan-ng/ui-*`) + Tailwind CSS 4**:
   - Configure Tailwind CSS 4 and spartan/ui primitives (`HlmButton`, `HlmCard`, `HlmInput`, `HlmLabel`, `HlmSelect`, `HlmScrollArea`, popover/menu).
   - Migrate `src/styles.scss` (or `styles.css`) to define all `--color-*`, `--font-sans`, `--font-mono`, `--space-*`, and `--radius-*` tokens for both default Light (`Late Sun Engineering`) and `html[data-theme="dark"]` (`Twilight Engineering`).
   - Phase out `@angular/material` usage from `AppComponent`, `LoginComponent`, `OnboardingComponent`, and `ChatComponent` in favor of spartan/ui + Tailwind v4.
3. **Align `AppComponent` Shell (`spartan-navbar`)**:
   - Replace the multi-link nav (`Specs`, `Code`, `DevOps`, `Sandboxes`, `Account`, `Admin`) with the PenPot `spartan-navbar`: theme-aware Minigma AI SVG logo + `Minigma AI` brand, `Organisation Switcher` (`HlmSelect` on `/chat` when authenticated), `Theme Toggle` ghost button, and `User Profile Menu` popover containing `Logout`.
   - On `/auth/*`, hide `spartan-navbar` and render the floating `36×36px` circular theme toggle.
4. **Align Auth, Onboarding & Chat Components**:
   - Update `LoginComponent` (Login & Sign-Up modes) and `OnboardingComponent` to match the centered `460px` `HlmCard` layout and spacing while preserving existing `AuthService` and `OrgService` logic.
   - Update `ChatComponent` to match `Chat — Router Agent` (`900px` max-width container, `MessageGroup / Agent` & `User`, `AgentActivity / Collapsible`, `A2UI / ask_question` card, and `Composer` with `Agent Selector` + right-arrow Send button).
5. **Add SVG Logos & Favicons**:
   - Place `logo-light.svg` and `logo-dark.svg` in `src/assets/` and wire theme-aware favicons in `src/index.html`.
