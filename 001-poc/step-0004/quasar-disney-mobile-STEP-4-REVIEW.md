# quasar-disney-mobile — STEP-4 REVIEW (Auth feature)

**Date:** 2026-08-18
**Branch:** `step-0004-auth-feature` (app), merged to `main` via PR #5 (`1ffea43`)
**Status:** Passed

Final cross-read of substeps 4.1–4.5 against the STEP-4 PLAN definition of done, doc 07/11/16
auth contracts, import rules, and the Tier A JS gate. Doc 03 intentionally still describes the
**three-query** cold-start gate in Flow 1 step 3 — STEP-4 ships **getMe-only** validation per the
PLAN; STEP-6 extends the same `LoadingGate`.

## What was checked

1. **Definition of done** — every PLAN checkbox against disk on `step-0004-auth-feature` (including
   uncommitted 4.5 shell files in the working tree).
2. **Substep completeness** — 4.1–4.5 marked Done in the PLAN and `prompts/STEP-index.md`.
3. **Architecture alignment** — doc 03 §6 Flow 1 (partial), doc 07 §3.2–§3.3/§4/§5, doc 11 §7.4/§8.4,
   doc 16 session model, ADR-0003/0017/0019/0020.
4. **Import rules** — DF1 (no axios/fetch in features), DF5 (auth ↮ storefront), doc 03 §8.2 seam.
5. **Test plan** — T1 auth slice; T2 login scoping, session restore, `/me` 401; final gate.
6. **Risk register** — RISK-0001 (T3 deferred) unchanged; RISK-0008 (demo creds in `@env`) still open.

## Definition of done — PASS

| Criterion | Result |
|-----------|--------|
| Real auth slice (`accessToken`, `expiresAt`) persisted; stub removed | Pass (4.1) — `authSlice.ts`; no `setAuthenticated` stub |
| `features/auth/api.ts` injects `login` / `getMe`; hooks exported | Pass (4.1) |
| Welcome, EmailEntry, PasswordEntry match doc 07 / reference layout | Pass (4.2–4.4) — Dinsey- wordmarks, `AuthSheetLayout`, `CredentialsForm` |
| F2: wrong credentials → inline error, not alert/bounce | Pass (4.4) — `PasswordEntryScreen` + T2 in `api.integration.test.ts` |
| Login success + restore call `getMe`; switch to app navigator | Pass (4.5) — `useSessionValidation`, `RootNavigator` |
| `/me` `UNAUTHORIZED` clears session → Welcome | Pass (4.5) — T2 in `sessionRestore.integration.test.ts` |
| Long-press wordmark cycles `dinsey` ↔ `ember` | Pass (4.3) — `cycleActiveTheme` on Welcome |
| a11y props on interactive controls; error live region on password field | Pass — buttons/links labeled; `TextField` assertive `accessibilityRole="alert"` |
| STEP test plan complete per substep | Pass |
| Final gate green | Pass — 205 tests / 26 suites, ~3 s |
| STEP review passed; index updated; archived | This document — archive after merge |

## Test plan — PASS

| Tier | Substep | Coverage |
|------|---------|----------|
| T0 | 4.1–4.5 | `tsc --noEmit` green |
| T1 | 4.1 | `authSlice.test.ts` — set/clear session, selectors, `parseWireExpiresAt`; `createStore.test.ts` persist whitelist `['auth']` |
| T2 | 4.4 | `api.integration.test.ts` — login success stores session; `INVALID_CREDENTIALS` does not dispatch `sessionCleared` / `resetApiState` |
| T2 | 4.5 | `sessionRestore.integration.test.ts` — rehydrate + `getMe` success; expired token → `sessionCleared` + unauthenticated |
| T2 | (regression) | `api/integration/authScoping.test.ts` still green — login 401 vs `/me` 401 scoping |
| Lint / format | 4.1–4.5 | ESLint 0 errors (5 pre-existing inline-style warnings in shell); Prettier clean |

Manual F1 auth smoke (Welcome → email → password → app tabs; wrong password inline error; kill/relaunch
restore) is **not verified in this review session** — confirm on iOS simulator before merge and note
in PR #5.

## Import rules — PASS

- **DF1:** no runtime `axios`/`fetch` in `src/features/auth/` (only test files import the axios
  instance for mock wiring).
- **DF5:** auth does not import storefront; storefront does not import auth.
- **Hooks only:** screens use `useLoginMutation` / route params; no direct client imports.
- **sessionCleared seam:** auth slice reduces `sessionCleared` in `extraReducers`; API module does
  not import the feature.

## Findings

### Must fix before merge

| Finding | Disposition |
|---------|-------------|
| **4.5 shell wiring is uncommitted** on `step-0004-auth-feature` (`useSessionValidation.ts`, `sessionRestore.integration.test.ts`, `AppShell` / `LoadingGate` / `RootNavigator` / auth README edits) | Commit and push to PR #5 before merge — the gate was run against the full working tree |

### Accepted (no fix required)

| Finding | Disposition |
|---------|-------------|
| `t2World.factory.ts` still overlays `accessToken` via an `injectStore` view rather than reading the real slice | Follow-up from STEP-3 REVIEW — the overlay keeps API-layer T2 suites independent of feature login wiring. Feature-owned T2 (`api.integration.test.ts`, `sessionRestore.integration.test.ts`) uses the real auth slice. Optional cleanup in a later STEP; not blocking |
| Doc 03 §6 Flow 1 step 3 still names three boot queries | Intentional — getMe-only landed in STEP-4; HomeFeed/CW gate is STEP-6 (PLAN + code comments in `useSessionValidation` / `LoadingGate`) |
| T3 render tests absent | By design — RISK-0001 |
| Jest worker may not exit cleanly (`PersistGate` / NetInfo listeners) | Non-blocking — same as STEP-2/3; all 205 tests pass |
| ESLint inline-style warnings in shell (`AppShell`, `App.tsx`, etc.) | Pre-existing from STEP-2 boot fixes; 0 errors |
| Inert recovery / MyDisney links on PasswordEntry | Per PLAN — doc 16 forecloses recovery in Phase 1 |

### None blocking

No high-severity defects found against the STEP-4 scope.

## Doc drift — reconciled

| Area | Change |
|------|--------|
| `features/auth/README.md` | Notes auth flow + getMe-only gate complete; HomeFeed/CW gate is STEP-6 |
| `architecture/03-architecture-overview.md` | **No change** — three-query gate description stays until STEP-6 (per 4.5 prompt) |
| `registries/risks.yml` | No change |

## Test gate

```sh
npx tsc --noEmit && CI=true npm test && npx eslint . && npx prettier --check .
```

All green on 2026-08-18 — 205 tests, 26 suites, ~2.8 s.

## Pull request

Merged **PR #5** on `quasar-disney-mobile-app` (`step-0004-auth-feature` → `main`, `1ffea43`).

## Archive

Completed 2026-08-18 — PLAN, substeps 4.1–4.5, and this review live in `prompts/001-poc/step-0004/`.

## Next action

**STEP-5** (Andres — storefront feature) is the parallel track; it depends on STEP-2 and STEP-3
(already on `main`) and is disjoint from auth (DF5). **STEP-6** wires the full three-query
cold-start gate once STEP-5 endpoints exist.
