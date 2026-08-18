# quasar-disney-mobile — STEP-4 PLAN: Auth feature

**Phase:** Phase 1 — POC (1a)
**Owner:** Raul Angel
**Status:** Done
**Date:** 2026-08-18
**Branch:** `step-0004-auth-feature` (merged app PR #5 → `main`)
**Repos (projection):** `quasar-disney-mobile-app` → `prompts` (index only)

> Build the two-step Welcome → email → password auth flow on the light `auth` surface, wire `login` and `getMe` onto `baseApi`, replace the stub auth slice with a persisted JWT session, and satisfy criterion **F2** (inline error on wrong credentials). Session restore and `/me` 401 → Welcome land here; the full three-query cold-start gate waits for STEP-5/6.

## Motivation

STEP-2 shipped the navigation shell with placeholder auth routes. STEP-3 (Andres) stands up the contract layer — types, axios/`baseApi`, mock adapter with credential compare and JWT validation. This STEP is Dev A's parallel track: the **light** surface mode, the only place tokens enter the client, and half of the Phase-1a sign-off path (**F1** auth leg, **F2** inline error).

STEP-5 (storefront) and STEP-4 are disjoint by design (DF5). Auth must not import storefront. The shell already mounts `AuthNavigator` / `AppTabsNavigator`; this STEP replaces placeholders and wires `selectIsAuthenticated` to real session state.

**Dependency:** branch from `main` **after STEP-3 merges** (needs mock `login`/`getMe`, `baseQueryWithAuth`, and wire types). Planning can proceed while 3.5 is in flight; do not execute 4.1 until STEP-3 is on trunk.

## Decisions already locked

- root `.throughstone/local-user.md` — read Experience level and Communication style; do not copy those values into this PLAN.
- `registries/risks.yml` — **RISK-0001** / **RISK-0011** (no T3 render tests in 1a — write a11y props anyway); **RISK-0008** (shared demo credentials in `.env`); **RISK-0005** (Dinsey- placeholder brand, not Disney marks — **DF10**).
- `architecture/03-architecture-overview.md` §6 Flow 1, §8 — auth feature owns auth slice + `features/auth/api.ts` injectEndpoints; screens use RTK Query hooks only; no `axios`/`fetch` outside `src/api/`.
- `architecture/07-ui-design-system.md` §3.2–§3.3, §4, §5 — `AuthSheetLayout`, `WelcomeHero`, `CredentialsForm`; three-route stack; back rules; long-press wordmark theme swap on Welcome; submit spinner in button; F2 error pattern (red underline, multi-line message, hint below).
- `architecture/11-interface-contracts.md` §7.4, §8.4, §9 — `POST /auth/login`, `GET /me`; `INVALID_CREDENTIALS` vs `UNAUTHORIZED`; session-clearing reaction in **`baseQueryWithAuth`** only (ADR-0017 / ADR-0020).
- `architecture/16-identity-auth.md` — demo creds from `@env`; auth decision is the mock adapter; persist auth slice only; `/me` cache not persisted; no signup/recovery screens.
- `architecture/04-data-model.md` §1.5 — auth slice: `accessToken` + numeric `expiresAt` (Unix); user from `getMe` RTK cache only.
- `architecture/12-test-strategy.md` §3 — T1 for reducer/selectors; T2 for login 401 scoping and session restore; T3 deferred.
- `inputs/ui/disney-plus-reference-screens.md` §1–§4 — layout reference; substitute **Dinsey-** wordmarks and in-repo placeholder art (**DF10**).
- ADR-0003, ADR-0009, ADR-0010, ADR-0017, ADR-0019, ADR-0020.

### Closed in this PLAN

| ID | Resolution |
|----|------------|
| **Cold-start gate scope** | STEP-4 wires **getMe-only** restore gate (LoadingGate visible while `/me` resolves after rehydrate or post-login). The full loader until `/me` + HomeFeed + CW is **STEP-6** once STEP-5 endpoints exist. Login success switches to the app navigator after `getMe` succeeds — home feed can still be the STEP-2 placeholder. |
| **Theme swap** | Long-press Dinsey wordmark on Welcome cycles `dinsey` ↔ `ember` (doc 07 §5). Module-level theme switch, not Redux. |
| **Recovery / magic-link links** | Password screen shows the reference *¿Problemas…?* link as **inert** (no navigation, no WebView) — doc 16 forecloses recovery in Phase 1. Same for *Más información sobre MyDisney*. |
| **Brand strip on Welcome** | Use shipped `brand-strip.svg` + placeholder tiles — not Disney/Marvel/ESPN marks. |

## Substeps

| # | Title | Produces | Depends on | Open questions |
|---|-------|----------|------------|----------------|
| 4.1 | Auth slice + `login` / `getMe` endpoints | `features/auth/state/authSlice.ts` (real model), `features/auth/api.ts`, store wiring, T1 tests | STEP-3 on `main` | — |
| 4.2 | Auth molecules + i18n | `AuthSheetLayout`, `WelcomeHero`, `CredentialsForm`; `es-419` auth keys | 4.1 | — |
| 4.3 | Welcome + EmailEntry screens | `WelcomeScreen`, `EmailEntryScreen`; theme long-press; email validation + navigation | 4.2 | — |
| 4.4 | PasswordEntry + F2 inline error | `PasswordEntryScreen`; login submit; back/editar rules; T2 login tests | 4.1, 4.3 | — |
| 4.5 | Session restore, `/me` 401, shell wiring | Real `selectIsAuthenticated`; boot/post-login `getMe`; LoadingGate for getMe-only; T2 restore/401 tests; manual smoke | 4.1, 4.4 | — |

## Test plan

T3 render tests are **out of this STEP** (RISK-0001). Device e2e is STEP-6. The STEP gate is Tier A JS.

| Test tier / surface | Substep(s) | Tests to create or update | Run timing | Command / gate | Notes |
|---------------------|------------|---------------------------|------------|----------------|-------|
| Type check (T0) | 4.1–4.5 | `tsc --noEmit` | Per substep from 4.1 | `npx tsc --noEmit` | |
| Unit (T1) | 4.1 | Auth reducer: login stores token + `expiresAt`; logout clears; `selectIsAuthenticated`; persist whitelist still `['auth']` only | 4.1 | `npm test` | doc 12 §3, doc 16 §3 |
| Integration (T2) | 4.4 | Login success stores session; `INVALID_CREDENTIALS` on login does **not** clear session / bounce stack | 4.4 | `npm test` | F2 / doc 11 §8.4 |
| Integration (T2) | 4.5 | Rehydrate + `getMe` success → authenticated; `/me` `UNAUTHORIZED` → session cleared → unauthenticated | 4.5 | `npm test` | doc 16 §4 |
| End-to-end | 4.5 | Manual: Welcome → email → password → app tabs (placeholder home); wrong password shows inline error | 4.5 | iOS simulator | F1 auth leg |
| Lint / format | 4.1–4.5 | ESLint + Prettier | Per substep | `npx eslint .` · `npx prettier --check .` | CI job |
| Security / authorization | 4.4–4.5 | 401 scoping covered by T2 above | 4.4–4.5 | `npm test` | ADR-0017 |

**Run timing:** tests run **per substep** from 4.1. Final STEP gate before review: `npx tsc --noEmit && CI=true npm test && npx eslint . && npx prettier --check .`.

## Open questions

None blocking. Execution waits on STEP-3 merge only.

## Ground rules

- **Calibrate communication from root `.throughstone/local-user.md`.** Do not copy those values into prompts as project facts.
- **Do not execute this STEP from the whole-STEP command.** Wait for `run substep 4.1` (then 4.2, …).
- **One owner, one branch** `step-0004-auth-feature` in the app repo. Index edits (`prompts/STEP-index.md`) stay on `prompts/` `main`.
- **Branch from `main` after STEP-3 merges.** Rebase if trunk moves.
- **Tests ship with the code** — see test plan; T3 explicitly deferred.
- **Code is documented as it's written** (`coding-standards/README.md` + `typescript.md`): TSDoc on every function/component.
- **No Expo. No `styled-components`. No `fetch`/`axios` in features.** Screens use generated RTK Query hooks only.
- **Auth ↛ Storefront.** Shell may read auth selectors; auth must not import storefront.
- **Do not build storefront home feed** — placeholder `HomeScreen` stays until STEP-5.
- **Do not wire the three-query LoadingGate** — getMe-only in this STEP; STEP-6 completes the gate.
- **Secrets:** `DEMO_EMAIL` / `DEMO_PASSWORD` from `@env` only; never hardcode in screens.
- **DF10:** no Disney/Marvel/Star Wars/hulu/ESPN marks in UI copy or assets.

## Definition of done

- [x] Real auth slice (`accessToken`, `expiresAt`) persisted via existing encrypted-storage adapter; stub `setAuthenticated` removed.
- [x] `features/auth/api.ts` injects `login` and `getMe` on `baseApi`; hooks exported for screens.
- [x] Welcome, EmailEntry, PasswordEntry screens match doc 07 / reference layout using Dinsey- brand assets.
- [x] F2: wrong credentials show inline error on PasswordEntry — not an alert, not a navigator bounce (T2 proves 401 scoping).
- [x] Successful login + session restore call `getMe` and switch to app navigator when authenticated.
- [x] `/me` `UNAUTHORIZED` clears session and returns to Welcome (T2).
- [x] Long-press wordmark on Welcome cycles `dinsey` ↔ `ember`.
- [x] a11y props on interactive controls (labels, error live region on password field — best effort without T3).
- [x] The STEP test plan is complete: each code-changing substep either added/updated its tests or records why not.
- [x] Final gate green: `npx tsc --noEmit && CI=true npm test && npx eslint . && npx prettier --check .`
- [x] STEP review passed; `prompts/STEP-index.md` updated; STEP archived to `prompts/001-poc/step-0004/` — merged app PR #5 2026-08-18.
