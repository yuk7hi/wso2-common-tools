# React Modular Template — Master Prompt (v4)

> **Version currency:** All library versions in this prompt were verified against the npm registry on 2026-07-08. The weekly `webapp-template-audit` skill re-verifies them.

> **For Claude Code:** This is the governing prompt for the entire project. Use plan mode for per-part planning, subagents (Agent tool) for implementation and delegated reviews, and the `/code-review` skill as the quality gate after each part.

**Goal:** Build a production-grade, modular React SPA template by first building a **full-featured demo app** (an inventory management system called "StockPilot"), then extracting reusable infrastructure into a configurable template via a strip-down script. Delivered as three artifacts: (1) the working demo, (2) the extractable template, and (3) a step-by-step learning guide that teaches someone to build the demo from scratch.

**Architecture:** React Router v8 framework mode with SPA config (`ssr: false`). The frontend is a pure API consumer — all backends are separate services (Java, Go, or Ballerina). During development, the backend is mocked via MSW (Mock Service Worker) driven by an OpenAPI 3.1 spec (3.1 chosen deliberately over 3.2 — codegen tooling support for 3.2 still lags). The same spec later generates the real backend — a sibling backend template + demo is planned in this repo to replace the MSW mock, which is why the frontend artifacts carry the `webapp-` prefix. Mobile support via responsive design + PWA, not React Native.

**Core principle:** The demo is the source of truth. Every standard, pattern, and convention lives in the demo first. The template is a byproduct of stripping demo-specific content. The review skill grows alongside the demo — each new pattern gets codified immediately.

---

## The Demo App: StockPilot (Inventory Management)

### Why inventory management?
It covers every pattern the template needs while staying realistic:
- **Auth:** Login + role-based access (Admin vs Viewer)
- **Dashboard:** Stat cards, low-stock alerts, recent activity, stock value chart
- **CRUD:** Items, categories, suppliers — full create/read/update/delete
- **Data tables:** Sortable, filterable, paginated, row actions
- **Forms:** Multi-field forms with validation, file upload (item images)
- **API patterns:** Pagination, filtering, search params, optimistic updates, error handling
- **State management & API:** Shared Redux store with RTK Query
- **Responsive:** Table → card layout on mobile, sidebar → hamburger
- **Theming:** Dark/light mode toggle
- **PWA:** Installable, offline shell caching

### Feature set
```
StockPilot — Inventory Management

- Login page (Asgardeo OIDC)
- Dashboard:
    • Total items count, total stock value
    • Low stock items (< threshold)
    • Recent stock adjustments (timeline)
    • Stock value over time (simple chart)
- Inventory:
    • Paginated table with search, category filter, column sorting
    • Item detail view (info, stock history, edit/delete actions)
    • Add/Edit item form (name, SKU, category, quantity, unit price,
      reorder threshold, supplier, description, image upload)
- Categories:
    • List + inline add/edit/delete
- Suppliers:
    • List + add/edit/delete (name, contact person, email, phone)
- Stock adjustments:
    • Add stock (with reason, quantity, date)
    • Remove stock (with reason, quantity, date)
    • Adjustment history per item
- Profile / user menu (name, role, logout)
- 404 page
- Dark/light theme toggle (persists in localStorage)
- Responsive across all pages
```

### User roles
- **Admin:** Full CRUD on all resources, stock adjustments, view dashboard
- **Viewer:** Read-only access to inventory, dashboard, categories, suppliers

---

## The Three Artifacts

### 1. The Demo (`/webapps/webapp-demo/`)
A fully working, runnable application. All features built, all standards followed. Serves as:
- The canonical reference for the review skill
- The source from which the template is extracted
- The subject of the learning guide

### 2. The Template (`/webapps/webapp-template/`)
Extracted from the demo via a strip-down script. What remains after removing:
- Demo-specific pages and route modules (inventory, categories, suppliers, dashboard)
- Demo-specific API handlers and mock data
- Demo-specific components (data tables, stat cards, etc.)
- Demo-specific state slices

What stays:
- Project scaffold (Vite, React Router, TypeScript, Biome)
- Auth infrastructure (Asgardeo provider, guards, hooks)
- API client pattern (Redux store with RTK Query)
- State management pattern (Redux slices per domain)
- Oxygen UI setup with theming
- Layout shell (sidebar, header, breadcrumbs, error boundaries)
- MSW infrastructure (for future mock needs)
- PWA setup, CI/CD, testing infrastructure
- Environment validation, error handling patterns

The strip-down script is the single source of truth for what's "template" vs "demo." Running it produces a clean starting point. A new project runs the generator CLI (Part 13) → gets a working skeleton.

### 3. The Learning Guide (`/webapps/docs/`)
A step-by-step guide to build **the demo** from scratch. Not the template — the full demo. Someone with basic React knowledge follows it and ends up with a working inventory management app. Each part of the guide corresponds to a part of the build and includes:
- Learning objectives
- Prerequisites
- Step-by-step instructions with copy-pasteable code
- Explanation of design decisions
- Common pitfalls and fixes
- Verification checkpoints ("at this point, `pnpm dev` should show X")

---

## The `webapp-template-review` Skill

A Claude Code skill at `.claude/skills/webapp-template-review/SKILL.md`, created at Part 0 (empty skeleton) and grown throughout: each part's Phase D (see Iterative Workflow) patches it with the patterns, standards, and conventions that part established. By Part 14 it is a comprehensive coding standard in executable form.

**Usage:** After any code change to the template or a project built from it, invoke `/webapp-template-review` to run a compliance check — it is the canonical reference for every pattern, the pre-commit verification, and the onboarding material.

**Structure (each section accumulates rules every part):**

```markdown
---
name: webapp-template-review
description: Coding-standard compliance review for the React template and projects built from it. Use when reviewing any code change, for pre-commit verification, or to onboard new team members — the skill IS the coding standard.
---

## Standards

### TypeScript
- strict mode, no `any`, named exports, curated public-API barrels only

### React & React Router
- hook rules, React Compiler memoization rules, component structure, file size limits, error boundaries, route config, layout composition

### Auth (Asgardeo)
- guard usage, token handling, session management

### API Layer
- RTK Query conventions, generated endpoints, error handling

### State Management
- slice-per-domain, selectors, persistence

### Styling & Theming
- Oxygen UI conventions, no direct `@mui/*` imports, dark mode

### Forms & Validation
- RHF + Zod patterns, server error mapping, form accessibility

### Testing
- test structure, render utilities, MSW handler patterns

### Security
- no secrets in client, CSP, input sanitization

### Accessibility
- ARIA labels, focus management, touch targets

### Performance
- loading states, no request waterfalls, bundle discipline
```

---

## Part Structure

### Part 0: OpenAPI Spec & Mock Backend
**What:** Define the full StockPilot API contract, the type-generation pipeline, and the MSW handler modules. The handlers are authored here as standalone modules — browser wiring and the `pnpm dev` verification land in Part 1, once the scaffold exists to run them.

**Deliverables:**
- `/webapps/spec/openapi.yaml` — complete API spec for all resources (OpenAPI 3.1)
- `/webapps/webapp-demo/app/mocks/` — MSW handler modules, typed against the spec via `openapi-msw` (browser wiring lands in Part 1); handlers hold in-memory state (mutations reflected in subsequent reads — Parts 7–9 gates depend on it) and are role-aware (Viewer mutations → 403, per Part 9)
- `/webapps/webapp-demo/app/lib/api-types.ts` — types generated via `openapi-typescript` (the spec is the single source of truth; Part 5 generates the RTK Query endpoints from the same spec)
- `/webapps/webapp-demo/app/lib/mock-data.ts` — realistic mock data factories
- `/webapps/docs/00-openapi-and-mocking.md` — "Define the API contract first"

**Quality gates:**
- Spec validates against OpenAPI 3.1 (`redocly lint` or equivalent)
- Generated types cover every endpoint and compile standalone (compiler errors on wrong request/response shapes)
- Every endpoint has a handler + mock-data factory, with simulated latency and error states via query params (`?simulateError=500`) — browser verification happens at Part 1's gate

**Review skill additions:** API contract patterns, mock data conventions, generated types usage

---

### Part 1: Project Scaffold
**What:** Initialize the demo project with Vite 8 (Rolldown bundler), React Router v8 (SPA mode, ESM-only), TypeScript 6.x, Biome 2.x, pnpm, and React Compiler enabled from day one. Wire up Part 0's MSW mock backend in the browser, plus the base test infrastructure so TDD (Phase B) works from the first part.

**Deliverables:**
- Working project skeleton: `pnpm install && pnpm dev` shows "Hello, StockPilot!"
- `vite.config.ts` with React Router plugin + React Compiler (`babel-plugin-react-compiler`)
- `react-router.config.ts` with `ssr: false`
- `tsconfig.json` with path aliases and ES2022 target (React Router v8 is ESM-only; TS 6 defaults: strict on, `esModuleInterop` always enabled, `classic` resolution removed)
- `.nvmrc` + `engines` field pinning Node 24 LTS (React Router v8 supports Active LTS only)
- `biome.json` with lint + format rules
- `app/root.tsx` — root layout
- `app/routes.ts` — single index route
- Environment variable setup (`VITE_API_BASE_URL`, validation at build time)
- MSW browser wiring (worker registration, dev-only start) — completes Part 0's mock backend
- Base test infrastructure: Vitest 4 + React Testing Library + MSW node server (`test/setup.ts`) with one smoke test — Part 11 builds the full utilities, coverage thresholds, and E2E on top
- `/webapps/docs/01-project-scaffold.md`

**Quality gates:**
- `tsc --noEmit` passes (strict mode)
- `biome check .` passes with zero errors
- `pnpm dev` serves the app with MSW intercepting — all Part 0 endpoints return realistic data in the browser
- `pnpm test` runs and passes the smoke test
- Project structure follows the agreed conventions

**Review skill additions:** Project structure conventions, Biome config standards, env validation pattern, React Compiler memoization rules (no hand-written `useMemo`/`useCallback`/`React.memo` without a documented justification — the compiler handles it)

---

### Part 2: Routing, Layouts & App Shell
**What:** Build the application shell — routing, responsive layouts, navigation. Note: Oxygen UI (integrated in Part 3) ships `AppShell`, `Sidebar`, `Header`, and `AppBreadcrumbs` — build this part's shell as thin structural placeholders and swap them onto the Oxygen components in Part 3, or run Parts 2 and 3 together.

**Deliverables:**
- Route config with nested layouts: root → auth layout / app layout
- `AppLayout`: sidebar + header + main content + breadcrumbs
- `AuthLayout`: centered card layout for login page
- `Sidebar`: nav items, collapse on mobile (≤ 768px), hamburger toggle
- `Header`: user menu placeholder, mobile menu toggle
- `Breadcrumbs`: auto-generated from route hierarchy
- Placeholder routes: `/login`, `/dashboard`, `/inventory`, `/categories`, `/suppliers`, `/profile`
- Route-level error boundaries (`errorElement` on root + layouts) — the "report error" fallback UI upgrade comes in Part 12
- `/webapps/docs/02-routing-and-layouts.md`

**Quality gates:**
- Navigate between all routes — layout persists, no remount
- Sidebar collapses to hamburger on mobile
- Breadcrumbs show correct path
- 404 page for unknown routes
- A route that throws renders its error boundary, not a blank screen
- Each route has a `<title>` via `<Meta>`
- `tsc --noEmit` passes, `biome check .` passes

**Review skill additions:** Route config conventions, layout composition pattern, responsive patterns, error boundary placement

---

### Part 3: UI Library — Oxygen UI (Decided)
**What:** Integrate WSO2's Oxygen UI design system (`@wso2/oxygen-ui` v0.12.0). Built on MUI v7 with React 19 and Emotion styling — the team's existing MUI knowledge transfers directly.

**No evaluation needed.** Oxygen UI is the mandated choice — WSO2's own design system, already used across WSO2 products, and aligned with Asgardeo auth (also WSO2). It is pre-1.0: pin the exact version; the weekly `webapp-template-audit` skill tracks its releases.

**Oxygen UI provides (verified against the published 0.12.0 package):**
- The full MUI v7 component library plus MUI X (`DataGrid`, `DatePickers` incl. `AdapterDateFns`, `TreeView`), all re-exported from its root entry point; Emotion, `date-fns`, and the Inter font ship inside it as regular dependencies
- App-level components: `AppShell`, `Sidebar`, `Header`, `AppBreadcrumbs`, `UserMenu`, `StatCard`, `ListingTable` (compound table with search, sort, pagination, selection, density), `Form`, `SearchBar`, `NotificationPanel`, `PageTitle`/`PageContent`, `ThemeSwitcher`/`ColorSchemeToggle`
- Built-in dark/light theming
- `@wso2/oxygen-ui-icons-react` — WSO2 product icon set
- AI skills for component reference and usage guidance
- Accessible by default (follows MUI's a11y standards)

**Dependencies:** install only `@wso2/oxygen-ui`, `@wso2/oxygen-ui-icons-react`, and React 19 + React DOM 19 (peer deps — Oxygen pins the React peer to an exact version, e.g. 19.2.3 for v0.12.0; a newer React patch needs a pnpm `peerDependencyRules` override). Never install `@mui/*` packages directly: npm's latest MUI is v9 while Oxygen bundles v7, so a direct install creates a second, incompatible MUI copy and breaks theming — import everything through `@wso2/oxygen-ui` (enforced by the review skill as a lint rule).

**Parts 2, 6, and 7 build on Oxygen's components rather than hand-rolling equivalents** — the layout shell on `AppShell`/`Sidebar`/`Header`/`AppBreadcrumbs` (Part 2), dashboard cards on `StatCard` (Part 6), the inventory table on `ListingTable` (Part 7).

**Deliverables:**
- Theme provider wrapping the app with Oxygen UI theme
- Design tokens: WSO2-branded colors, typography, spacing
- Dark/light mode toggle (system preference detection + localStorage persistence)
- No flash of wrong theme on page load
- First styled components: button variants, input fields, card, dialog
- `/webapps/docs/03-oxygen-ui-and-theming.md`

**Quality gates:**
- Toggle dark/light — all Oxygen UI components respond instantly
- Respects `prefers-color-scheme` on first load
- Theme persists across page refresh
- Components are accessible (Oxygen UI inherits MUI's a11y)
- `tsc --noEmit` passes with strict mode

**Review skill additions:** Oxygen UI theme structure, component usage conventions, dark mode pattern

---

### Part 4: Authentication — Asgardeo
**What:** Integrate Asgardeo OIDC auth using `@asgardeo/react` SDK v0.25.x, part of the Asgardeo JavaScript SDK Suite (do not use `@asgardeo/auth-react` — deprecated and no longer maintained). The SDK is pre-1.0 — pin the exact version.

**Deliverables:**
- `<AuthProvider>` wrapping the app with Asgardeo config (exact component name from `@asgardeo/react`)
- `useAuth()` hook wrapping the SDK's auth hook (API will be discovered during implementation)
- `AuthGuard` — protects routes, redirects to `/login` if unauthenticated
- `GuestGuard` — redirects to `/dashboard` if already authenticated
- Login page with Asgardeo redirect
- Logout: RP-initiated OIDC logout via the SDK (`end_session_endpoint` with `id_token_hint` + registered `post_logout_redirect_uri`) — clearing local state alone leaves the IdP session alive, and `trySignInSilently()` would sign the user straight back in on the next load
- OIDC hygiene: authorization code + PKCE (S256) only — implicit flow forbidden; redirect and post-logout URIs registered as exact matches (no wildcards)
- Role extraction from ID token (Admin vs Viewer)
- API request integration: a thin wrapper over the SDK's worker-proxied HTTP client, consumed by Part 5's custom RTK Query `baseQuery` — in webWorker mode the main thread never sees the token, so do NOT extract it for `prepareHeaders`-style header injection
- Token storage: `webWorker` mode — WSO2's recommended mechanism (tokens isolated in a worker thread, strongest XSS protection):
    • Caveat: worker storage clears on page reload, so the SDK must re-authenticate with the IdP on every reload
    • Mitigation (SDK-supported): silent re-authentication via `trySignInSilently()` / `prompt: "none"` against the active IdP session — no visible re-login, graceful fallback to the login page when no session exists — plus an app-level loading state to avoid flicker. See the [WSO2 silent sign-in guide](https://wso2.com/identity-platform/docs/complete-guides/javascript/manage-tokens-in-apps/#silent-sign-in); its snippets predate the current SDK, so confirm the equivalent API in `@asgardeo/react` during implementation
    • Document the security/UX tradeoff per the [WSO2 front-end security guide](https://wso2.com/identity-platform/docs/complete-guides/fesecurity/insecure-tokens/); httpOnly cookies are ruled out — they would require a BFF, contradicting the pure-SPA architecture
- `/webapps/docs/04-authentication.md`

**Quality gates:**
- Unauthenticated user → redirected to Asgardeo login → redirected back → authenticated
- Authenticated user visiting `/login` → redirected to `/dashboard`
- Logout ends the Asgardeo session (RP-initiated) and redirects to login
- After logout, a page reload lands on `/login` and does NOT silently re-authenticate
- Token refresh works transparently (SDK handles this)
- Page reload re-establishes the session silently via `trySignInSilently()` — the user sees a brief loading state, never a re-login
- Silent sign-in verified cross-browser — the flow typically relies on a hidden iframe to the IdP, which Safari/Firefox third-party cookie blocking can break; where it fails, the app must fall back to a full redirect re-auth, not an error
- No auth tokens in localStorage or client bundle source

**Review skill additions:** Auth pattern (guard + provider + hook), token handling rules, Asgardeo config conventions

---

### Part 5: State Management & API Layer — Redux Toolkit + RTK Query (Decided)
**What:** Set up Redux Toolkit as the single state management + API data fetching layer. RTK Query replaces TanStack Query — one store, one cache, one DevTools panel.

**Why Redux Toolkit over Context API:**
StockPilot and the team's real applications share patterns that benefit from Redux:
- **Reference data:** Employee lists, company metadata, dropdown options — consumed across deeply nested component trees. Redux selectors avoid prop drilling and unnecessary re-renders.
- **Paginated API state + UI state in one system:** RTK Query handles pagination, caching, and invalidation for API data while Redux slices manage auth, theme, and UI state — all in one DevTools session.
- **Predictable state changes:** Every state mutation is a dispatched action, logged and time-travel-debuggable. Critical for debugging stale dropdown data or cascading API calls.

**Store structure (slice-per-domain pattern):**

```
app/store/
├── store.ts              — configureStore with middleware
├── hooks.ts              — useAppSelector, useAppDispatch (typed)
├── slices/
│   ├── auth-slice.ts     — user, role, isAuthenticated
│   ├── theme-slice.ts    — dark/light mode, persisted to localStorage
│   └── ui-slice.ts       — sidebar open, active modal, toasts
└── api/
    ├── base-api.ts       — createApi with a custom baseQuery over the worker-proxied HTTP client (Part 4)
    ├── generated.ts      — endpoints generated from the OpenAPI spec via @rtk-query/codegen-openapi
    ├── items-api.ts      — enhanceEndpoints: cache tags, optimistic updates, pagination config
    ├── categories-api.ts — enhanceEndpoints: cache tags
    ├── suppliers-api.ts  — enhanceEndpoints: cache tags
    └── adjustments-api.ts— enhanceEndpoints: cache tags + history
```

**Dependencies to pull in:**
- `@reduxjs/toolkit` + `react-redux`
- `@rtk-query/codegen-openapi` (dev dep — generates the RTK Query endpoints from `/webapps/spec/openapi.yaml`)
- No persistence library (`redux-persist` is unmaintained) — persist theme + user preferences with RTK's `createListenerMiddleware` syncing selected slices to localStorage, plus a preloaded-state read at store creation

**Deliverables:**
- Store configuration with RTK Query middleware
- Typed hooks (`useAppSelector`, `useAppDispatch`)
- Auth slice: synced with Asgardeo auth state
- Theme slice: dark/light with localStorage persistence via listener middleware
- UI slice: sidebar, modals, toast notifications
- RTK Query endpoints generated from the OpenAPI spec via `@rtk-query/codegen-openapi`, enhanced with cache tags and optimistic updates (the spec stays the single source of truth — no hand-maintained endpoint definitions to drift)
- Auto-generated React hooks: `useGetItemsQuery`, `useCreateItemMutation`, etc.
- Custom `baseQuery` delegating requests to the Asgardeo worker-proxied HTTP client (Part 4) — the Bearer token is attached inside the worker and never enters main-thread code, the Redux store, or DevTools
- Redux DevTools disabled in production (`devTools: import.meta.env.DEV`)
- Response interceptor: 401 → logout, 403 → show forbidden, 500 → toast
- Optimistic update example on item toggle
- `/webapps/docs/05-state-management-and-api.md`

**Quality gates:**
- Redux DevTools show all actions and state changes in dev builds — and are disabled in production
- API calls include the Bearer token automatically (attached inside the worker)
- Auth state updates on login/logout
- Theme toggle updates store + persists to localStorage
- RTK Query hooks return typed data matching OpenAPI schemas
- 401 response triggers logout flow
- Paginated queries don't waterfall
- Cache invalidates after mutations
- Every query hook handles loading + error + success + empty states
- No unnecessary re-renders (React Compiler handles memoization — the profiler pass is a spot check only)

**Review skill additions:** Redux Toolkit conventions, slice-per-domain pattern, RTK Query patterns, selector patterns, persistence patterns

---

### Part 6: Demo Features — Dashboard
**What:** Build the StockPilot dashboard — the first real demo feature.

**Deliverables:**
- Stat cards: total items, total stock value, low stock count, active suppliers (Oxygen UI `StatCard`)
- Low stock alert table (items below reorder threshold)
- Recent stock adjustments timeline
- Stock value over time (simple area chart — `recharts` v3)
- All data fetched via the generated, typed RTK Query hooks
- Responsive: stat cards reflow to 2-column then single-column on mobile
- Loading skeleton states for each section
- Error state for each section with retry button
- `/webapps/docs/06-dashboard.md`

**Quality gates:**
- Dashboard loads with all sections populated
- Loading skeletons appear while data fetches
- Error state renders on API failure (test via `?simulateError=500`)
- Chart renders correctly with mock data
- Responsive across breakpoints
- Viewer role: dashboard loads but CRUD actions are hidden (ad-hoc role check for now — Part 9 codifies the role-based UI system)

**Review skill additions:** Dashboard patterns, data visualization conventions, loading/error state patterns

---

### Part 7: Demo Features — Inventory CRUD & Data Tables
**What:** Build the inventory management core — data table, item detail view, and CRUD forms.

**Deliverables:**
- Paginated data table component (reusable), built on Oxygen UI's `ListingTable` compound component (search, sort, pagination, selection, density built in):
    • Search input (debounced, synced with URL search params)
    • Category filter dropdown
    • Column sorting (click header to toggle asc/desc)
    • Pagination controls (page size selector, page numbers)
    • Row actions (view, edit, delete with confirmation dialog)
    • Empty state ("No items found. Create your first item.")
    • Loading state (skeleton rows)
- Item detail page: info card, stock history table, edit/delete buttons
- Add/Edit item form (inline or separate page):
    • Fields: name, SKU, category (select), quantity (number), unit price, reorder threshold, supplier (select), description (textarea), image (file upload preview)
    • Validation (Zod 4 schema via React Hook Form + `@hookform/resolvers`)
    • Success toast + redirect to list
- Responsive: table → card list on mobile (each card shows key fields, tap to view detail)
- `/webapps/docs/07-inventory-crud.md`

**Quality gates:**
- Table loads with paginated, sorted, filtered data
- Search updates URL params and filters results
- Click column header → sort asc → click again → sort desc → click again → remove sort
- Pagination: next/prev/page numbers work, page size changes work
- Create item → appears in table → detail view shows correct data
- Edit item → changes reflected immediately (optimistic or refetch)
- Delete item → confirmation dialog → item removed from table
- All table states work: loading, empty, error, data
- Mobile: cards show correctly, tap navigates to detail
- Forms validate: required fields show errors, numeric fields reject text
- Image upload preview works (even with mock — show placeholder)

**Review skill additions:** Data table conventions, form patterns, URL search param sync, optimistic update patterns, modal patterns

---

### Part 8: Demo Features — Categories & Suppliers
**What:** Build the supporting CRUD modules — simpler than inventory but with their own patterns.

**Deliverables:**
- Categories: list with inline add (input + button at top), inline edit (click to edit name), inline delete
- Suppliers: table (simpler than inventory — name, contact, email, phone), add/edit modal, delete confirmation
- Both integrated with the existing generated RTK Query endpoints
- Both responsive: inline edit works on mobile, modals are full-screen on mobile
- `/webapps/docs/08-categories-and-suppliers.md`

**Quality gates:**
- Categories: add, edit, delete all work inline without page navigation
- Suppliers: add via modal, appears in table, edit/delete work
- Both handle loading, empty, and error states
- Category delete with items assigned → show warning ("3 items use this category")

**Review skill additions:** Inline edit patterns, modal patterns, cascading delete handling

---

### Part 9: Stock Adjustments & Role-Based Access
**What:** Build the stock adjustment flow and implement role-based UI restrictions.

**Deliverables:**
- Stock adjustment form (from item detail or inventory list):
    • Add/remove toggle
    • Quantity, reason (select: received, returned, damaged, adjustment, other), date, notes
- Adjustment history per item (table on item detail page)
- Role-based UI:
    • Admin: all CRUD actions visible and functional
    • Viewer: view-only — edit/delete/add buttons hidden or disabled
    • Route-level protection: viewer trying to POST/PUT/DELETE gets 403
- `/webapps/docs/09-stock-adjustments-and-roles.md`

**Quality gates:**
- Add stock → quantity increases, adjustment appears in history
- Remove stock → quantity decreases (cannot go below 0, validated)
- History shows all adjustments in reverse chronological order
- Viewer logs in → no add/edit/delete buttons visible anywhere
- Viewer tries direct API call → 403 error shown gracefully

**Review skill additions:** Role-based access patterns, audit trail patterns, complex form patterns

---

### Part 10: Forms & Validation System
**What:** Extract reusable form patterns from the demo into a form system. Standardize validation, error display, and accessibility.
(Note: forms have been built throughout Parts 6-9. This part codifies and refactors them into reusable patterns.)

**Deliverables:**
- Form state: React Hook Form + `@hookform/resolvers` (Zod 4) — do not hand-roll dirty tracking, submission state, or focus-on-error
- Form field components: `FormInput`, `FormSelect`, `FormTextarea`, `FormDatePicker`, `FormFileUpload` — thin wrappers binding Oxygen UI inputs to React Hook Form
- `Form` wrapper component (provides the RHF context, handles submission state)
- Zod 4 schema library (`/webapps/webapp-demo/app/lib/schemas/`) — all validation schemas in one place (Zod 4 explicitly: its error APIs differ from v3, and guide code must be copy-pasteable)
- Server error mapping: API error response → form field errors
- Accessibility: labels, error messages linked via `aria-describedby`, focus management on error
- `/webapps/docs/10-forms-and-validation.md`

**Quality gates:**
- Tab through any form — focus order is logical
- Submit empty form — first error field receives focus, error announced by screen reader
- Server validation errors map to correct fields
- File upload validates size and type before submission
- All forms show loading state during submission, disable buttons to prevent double-submit

**Review skill additions:** Form architecture, validation patterns, accessibility checklist

---

### Part 11: Testing Infrastructure
**What:** Set up testing with Vitest 4 + React Testing Library + MSW, plus a small Playwright E2E smoke suite for the flows unit tests can't reach (auth redirects, role-based UI, PWA installability).

**Deliverables:**
- Extends the base Vitest + MSW setup from Part 1 with the full provider stack
- `renderWithProviders()` — wraps components with Router + Redux Store + Auth + Theme + MSW
- `createMockAuth()` — helper to set auth state per test (Admin, Viewer, unauthenticated)
- Example tests:
    • Component render test (dashboard stat cards)
    • User interaction test (login → dashboard redirect)
    • Hook test (`useItems` returns paginated data)
    • API integration test (create item → appears in list)
    • Auth guard test (protected route redirects unauthenticated)
    • Role-based access test (Viewer cannot see edit buttons)
- Explicit environment decision: Vitest browser mode (stable in Vitest 4, runs component tests in a real browser) vs jsdom — pick one and document why
- Playwright E2E smoke suite: Asgardeo login redirect round-trip, one CRUD happy path, viewer-role restrictions (the PWA manifest/service-worker check joins this suite in Part 12, when the PWA exists)
- Automated accessibility checks: `vitest-axe` on key components + `@axe-core/playwright` scans of every top-level page in the E2E suite
- Coverage thresholds: 80% branches, 80% functions, 80% lines (unit/component only — E2E excluded)
- `/webapps/docs/11-testing.md`

**Quality gates:**
- `pnpm test` runs all unit/component tests and passes
- `pnpm test:e2e` runs the Playwright smoke suite and passes
- axe reports zero violations on top-level pages and key components
- `pnpm test:coverage` meets thresholds
- No real network calls in tests (MSW intercepts everything)
- Tests don't depend on test order

**Review skill additions:** Test structure conventions, render utility patterns, mock patterns, automated a11y check conventions

---

### Part 12: CI/CD, PWA & Production Readiness
**What:** GitHub Actions pipeline, PWA setup, Choreo deployment, env validation, error monitoring placeholder. Deployment is handled by Choreo — merging a PR on the connected repo deploys automatically, so the CI pipeline's job is to gate what merges.

**Deliverables:**
- GitHub Actions, structured for the monorepo architecture:
    • Central reusable workflow (hosted once, versioned by tag) carrying the pipeline — lint → type-check → test → build → Lighthouse CI → bundle budgets — parameterized by `working-directory`
    • Per-monorepo umbrella workflow at the repo root `.github/workflows/` (GitHub only reads workflows there): always runs on PRs, auto-discovers template webapps via the generator-stamped `package.json` marker, runs the reusable workflow for changed apps only (`dorny/paths-filter`), and ends in a `summary` job with one stable check name that always reports — naive path-filtered required checks hang PRs that don't touch a webapp
    • Merge gating: an org-level ruleset requires the `summary` check, targeted by a custom repository property on opted-in repos — legacy webapps without the marker are untouched, new template webapps are discovered with zero workflow edits
    • Hardening: actions pinned to commit SHAs, top-level `permissions: contents: read` (elevated per job only as needed), `pnpm install --frozen-lockfile`
- `vite-plugin-pwa`: service worker precaching the app shell ONLY — API origin excluded from runtime caching (`NetworkOnly`), Cache Storage cleared on logout; install prompt
- CD is handled by Choreo: the repo connects as a web application component and deployment happens automatically on PR merge
- Environment validation at build time (missing `VITE_*` vars fail the build)
- Error boundary component with fallback UI and "report error" button (placeholder)
- Security headers, configured at the Cloudflare level for the app's domain (Transform Rules or a Worker — `<meta>` tags cannot deliver `frame-ancestors` or HSTS, so header-level config matters):
    • CSP: `default-src 'self'; script-src 'self'; connect-src 'self' <API origin> <Asgardeo origin>; frame-src <Asgardeo origin>; worker-src 'self' blob:; object-src 'none'; base-uri 'self'; frame-ancestors 'none'` — `frame-src` is required by silent sign-in and `worker-src blob:` by webWorker token storage; a naive strict CSP silently breaks both
    • `style-src`: Emotion injects inline styles — either accept `style-src 'unsafe-inline'` (styles only, scripts stay strict) and document the tradeoff, or inject a fresh per-request nonce via a Cloudflare Worker (HTMLRewriter on the HTML + matching CSP header, wired into Emotion's `CacheProvider`); a build-time nonce is a constant and adds nothing
    • Companions: `Strict-Transport-Security` (or Cloudflare's HSTS setting), `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin` (OIDC redirects carry `code`/`state` in URLs), `Permissions-Policy`
- Lighthouse CI (`@lhci/cli`) job asserting Performance ≥ 90, Accessibility ≥ 95, Best Practices ≥ 90 on the built app (Lighthouse removed its PWA category in v12 — installability is verified by the Playwright manifest/service-worker check instead)
- Bundle-size budgets enforced in CI (`size-limit` or equivalent), set per entry chunk — watch closely, since Oxygen UI bundles all of MUI v7 + MUI X
- `CONTRIBUTING.md`: how to run the gates locally, PR expectations, review process
- `/webapps/docs/12-cicd-and-production.md`

**Quality gates:**
- PR workflow: all checks must pass before merge
- Merged PR auto-deploys via Choreo; the deployed app responds with the full security header set (verify with `curl -I`)
- Lighthouse CI and bundle-budget jobs fail the pipeline on regression
- Missing env vars fail at build, not runtime
- PWA: install prompt appears in Chromium (iOS Safari has no prompt — Add to Home Screen only), offline shell works
- Playwright: PWA manifest + service worker registration verified; API responses never appear in Cache Storage
- Playwright: login and silent sign-in work with the production CSP enforced

**Review skill additions:** CI/CD conventions, Choreo deployment conventions, PWA checklist

---

### Part 13: Extraction Script & Generator CLI
**What:** Build the script that strips demo-specific code to produce the template, and the CLI that creates new projects from it. Both scripts are ESM (React Router v8 and the toolchain are ESM-only) — run via `tsx` or Node 24's native type stripping.

**Deliverables:**
- `scripts/extract-template.ts`:
    • Removes demo route modules and pages
    • Removes demo API handlers and mock data
    • Removes demo components (inventory table, dashboard widgets, etc.)
    • Keeps all infrastructure: auth, API client, state management, layouts, MSW setup, theme, testing, CI/CD
    • Produces `/webapps/webapp-template/` directory
    • Script is versioned alongside the demo — always in sync
- `scripts/create-app.ts` (generator CLI):
    • Prompts for the destination directory and project metadata (name, package name, Asgardeo org/client ID, API base URL)
    • Copies the template into the destination and applies the metadata substitutions
    • Stamps the template version marker into the generated `package.json` (e.g. `"wso2Template": "webapp@<version>"`) — the CI umbrella discovers apps by this marker (Part 12), and the audit skill uses it to find apps on outdated template versions
    • Refuses to overwrite a non-empty destination
    • Outputs a working project
- Template versioning: semver tags + `CHANGELOG.md` recording template-affecting changes per release (consumers upgrade by reviewing the changelog against their generated baseline)
- CI drift guard: a pipeline job (added once this part lands) that runs extraction → generator → install + build on the output, so demo changes that silently break the template fail the PR
- Verification: run extraction → run generator → the output installs, builds, and runs
- `/webapps/docs/13-extraction-and-generator.md`

**Quality gates:**
- Running extraction produces a clean template in `/webapps/webapp-template/`
- Generated project is structurally identical to the template apart from metadata substitutions
- Generated project passes the full gate set out of the box: `pnpm install`, `tsc --noEmit`, `biome check .`, `pnpm test`, `pnpm dev`
- Extraction script is deterministic (same demo → same template every time)
- The CI drift guard runs on every PR and fails when extraction or generation breaks

**Review skill additions:** Extraction/generator patterns (meta — about the template itself)

---

### Part 14: Documentation Site & Learning Guide
**What:** Assemble all per-part learning guides into a cohesive documentation site (VitePress or similar).

**Deliverables:**
- VitePress site at `/webapps/docs/` with all 15 guides (Parts 0–14)
- Getting started page: prerequisites, quick start
- Architecture overview: diagrams (Mermaid) showing component tree, data flow, route structure
- Architecture Decision Records (ADRs) for key choices
- "Common recipes" section: how to add a new page, how to add a new API endpoint, etc.
- Search functionality
- Docs deploy workflow: the site builds and publishes (GitHub Pages or similar) on merge to main
- `/webapps/docs/14-documentation-site.md` (meta-doc about how the docs site was built)

**Quality gates:**
- All guides are accessible from the sidebar
- Code blocks are syntax-highlighted and copy-pasteable
- Internal links between guides work
- Search returns relevant results
- Site builds and deploys (GitHub Pages or similar)

---

## Iterative Workflow Per Part

For **each** part, follow this exact cycle:

### Phase A: Plan
1. Enter plan mode
2. Write a detailed implementation plan to `_plans/YYYY-MM-DD_part-NN-slug.md` (working artifact — `_plans/` is non-versioned and removed once the project completes)
3. Plan must include: exact files, complete code stubs, test cases, verification steps
4. Plan must specify ALL three deliverables: demo code, review skill additions, doc section

### Phase B: Build (TDD)
1. Follow test-driven development where applicable
2. Implement following RED-GREEN-REFACTOR
3. Build demo code first, then documentation, then patch review skill
4. Commit after each logical chunk

### Phase C: Evaluate & Review (Two-Stage Gate)
1. Run `/code-review` on the part's diff
2. **Stage 1 — Standards Compliance Review** (run `/webapp-template-review`):
   - Does the code match the growing `webapp-template-review` skill?
   - Does it follow React 19 / React Router v8 best practices?
   - Are hooks used correctly (rules of hooks)?
   - Is TypeScript strict mode satisfied?
   - Are accessibility requirements met?
   - Is the layout responsive (mobile-first)?
3. **Stage 2 — Security Review** (run `/security-review`):
   - Static analysis for secrets, injection, XSS
   - Dependency audit (`pnpm audit`)
   - Token handling audit (no secrets in client)
   - CSP header readiness
4. Both stages must PASS before proceeding

### Phase D: Update Review Skill
1. Identify new patterns, standards, and conventions introduced in this part
2. Patch `webapp-template-review` skill with additions
3. Verify: run the skill against the demo code to confirm it catches the patterns

### Phase E: Push & Tag
1. Clean commit per part
2. Tag: `part-NN-complete`
3. Push to remote

### Phase F: Retrospective
1. Brief note: what went well, what was tricky, decisions to revisit
2. Save to `_plans/retrospectives/part-NN.md`
3. Promote decisions worth keeping into the ADRs (Part 14) — retrospectives live in the temporary `_plans/` and are discarded with it

---

## Quality Agent Skills

### Skill: `webapp-template-audit`
Created alongside Part 1 — the build spans weeks, and version pins need tracking from the start, not only post-build. Weekly scheduled agent (Claude Code routine / cron) that checks:
- Dependency freshness (major version bumps in core deps)
- React Router release notes for breaking changes (yearly major cadence — expect v9)
- Asgardeo SDK updates (`@asgardeo/react` — pre-1.0, watch for breaking API changes and the 1.0 release)
- Oxygen UI releases and its bundled dependency versions (it bundles MUI v7 while npm latest is v9 — confirm every release that all MUI imports still come through Oxygen's re-exports)
- TypeScript version currency — including TypeScript 7 (the Go-native compiler, already at RC): plan the migration when it goes stable
- React Compiler and Vite (Rolldown) release notes
- Any CVEs in dependencies
- Template adoption: scan for generated apps (via the `wso2Template` marker) running outdated template versions
- Report: ✅ current / ⚠️ minor updates / 🔴 major migration needed / 🚨 security patch

### Skill: `webapp-template-security-scan`
Created once Part 13 produces the template. Monthly deep scan:
- `pnpm audit` with zero-tolerance for HIGH/CRITICAL
- Static analysis for secrets, XSS vectors, unsafe DOM manipulation
- JWT token storage audit
- CSP configuration review
- Dependency confusion check
- Runs against both demo and template

---

## Constraints & Standards

### Code Quality
- TypeScript strict mode — no `any` without explicit justification
- `// @ts-ignore` is a failing review
- Biome for linting and formatting — zero warnings policy
- No commented-out code in committed files
- File size: components ≤ 300 lines, utilities ≤ 150 lines
- Named exports only (no default exports except route components)
- React Compiler is on — hand-written `useMemo`/`useCallback`/`React.memo` requires a documented justification

### Architecture
- Every module exposes a clean public API (a deliberate, hand-curated `index.ts` barrel — never whole-directory re-exports); within a module, use direct imports (broad barrels hurt Vite dev cold-start and tree-shaking, and invite circular imports)
- All MUI imports go through `@wso2/oxygen-ui` re-exports — `@mui/*` import paths are a failing review
- Modules must not import from sibling modules directly
- The demo is the source of truth — patterns are extracted, not invented in isolation
- API calls always go through RTK Query hooks — never raw `fetch` in components
- URL search params for filterable/sortable/paginated state

### Responsive Design
- Mobile-first CSS approach
- Sidebar collapses to hamburger on ≤ 768px
- Touch targets ≥ 24px (WCAG 2.2 AA minimum, SC 2.5.8); use 44–48px for primary touch controls per Apple HIG / Material guidance
- Horizontal scroll should never happen on mobile
- Test on viewport widths: 375px, 768px, 1024px, 1440px
- Data tables → card layout on mobile

### Language & Formatting
- English-only — no i18n framework (internal tooling; the retrofit cost is accepted in the unlikely event it's ever needed)
- Dates and times displayed in ISO 8601 (`YYYY-MM-DD`, `YYYY-MM-DD HH:mm`) regardless of the viewer's locale — shared screenshots stay unambiguous when troubleshooting across regions
- Timestamps stored and transferred as UTC ISO 8601; displayed in local time, with the UTC offset shown where ambiguity matters (e.g. audit trails, stock adjustment history)
- Numbers and currency formatted via `Intl.NumberFormat` with a pinned locale (`en-US`) — consistent separators in every screenshot

### Documentation Standards
- Each learning guide completable by a mid-level React developer in ≤ 2 hours
- Every code block must be copy-pasteable and runnable
- "Checkpoint: your app should now..." verification points every 15-20 steps
- Mermaid diagrams for architecture concepts
- Link to official docs for deeper dives

### Security Baseline
- No secrets in client code (verified by static analysis)
- Auth tokens handled by the Asgardeo SDK with `webWorker` storage (WSO2's recommended mode; see Part 4) — never localStorage
- Security headers (CSP + companion set) configured at the Cloudflare level for the app's domain — see Part 12 for the directive list
- `pnpm audit` passes with zero HIGH/CRITICAL findings
- No `dangerouslySetInnerHTML` without DOMPurify
- All API calls go through RTK Query hooks backed by the worker-proxied `baseQuery` — the token is attached inside the SDK worker and never touches main-thread code, the store, or DevTools

---

## Getting Started

To kick off this project:

1. All artifacts live under `/webapps/` in this repo — this prompt runs from `/webapps/PROMPT.md`
2. Create the `webapp-template-review` skill (empty skeleton)
3. Begin with **Part 0** — OpenAPI spec + MSW mock backend

**First steps:**
```
1. Create the empty review skill at .claude/skills/webapp-template-review/SKILL.md
2. Enter plan mode and write a detailed implementation plan for Part 0:
   OpenAPI Spec & Mock Backend for StockPilot.
   Save to _plans/
```

---

## Success Criteria

The project is complete when:

- [ ] All 15 parts (0–14) are built, reviewed, and merged
- [ ] `pnpm dev` in `/webapps/webapp-demo/` runs the full StockPilot app with all features
- [ ] Running the extraction script produces a clean template in `/webapps/webapp-template/`
- [ ] Generator CLI produces a working project that passes all quality gates out of the box
- [ ] All 15 learning guides can be followed to build StockPilot from scratch
- [ ] `webapp-template-review` skill contains rules for every pattern
- [ ] The app works on mobile web (responsive + PWA installable)
- [ ] Lighthouse CI green: Performance ≥ 90, Accessibility ≥ 95, Best Practices ≥ 90; PWA installability verified via the Playwright manifest + service worker check
- [ ] Zero HIGH/CRITICAL `pnpm audit` findings in both demo and template
- [ ] `webapp-template-audit` and `webapp-template-security-scan` skills are active
- [ ] CI/CD pipeline catches regressions before merge
