# quasar-disney-mobile — STEP-2 REVIEW (Scaffold app repo & foundation)

**Date:** 2026-08-18
**Branch:** `step-0002-app-scaffold` (+ follow-up `step-0002-boot-fixes`)
**Status:** Passed

Final read-through of substeps 2.1–2.6 against the STEP-2 PLAN definition of done, architecture docs, and the Tier A JS gate. Doc drift from implementation was reconciled in the docs hub; remaining deferred items stay in the risk register by reference. Post-merge iOS boot fixes landed in app PR #4.

## What was checked

1. **Definition of done** — every PLAN checkbox against disk on `step-0002-app-scaffold`.
2. **Substep completeness** — 2.1–2.6 all marked Done in PLAN and `prompts/STEP-index.md`.
3. **Architecture alignment** — doc 03 §8 layout, doc 07 navigation/shell/atoms, doc 10 §5.2 error boundary, doc 15 §2 NetInfo overlay, ADR-0014 (`isConnected` only).
4. **Import rules** — ESLint zones (auth ↮ storefront, shared ↮ features, DF1 axios/fetch in `src/api/` only).
5. **Test plan** — T1 tests present where required; T3 deferred with RISK-0001 recorded; final gate green locally.
6. **Risk register** — RISK-0003 and RISK-0006 closed; RISK-0001 remains open (T3 deferred).

## Definition of done — PASS

| Criterion | Result |
|-----------|--------|
| App repo exists, license posture, remote | Pass (2.1) |
| iOS + Android boot | Pass — both platforms (RISK-0003 closed) |
| `src/{app,features,shared,api}/` tree | Pass (2.2) |
| `repos.yml` entry | Pass (2.2, merged earlier) |
| `createStore()` + persist whitelist `['auth']` | Pass (2.3) |
| ESLint + Prettier + GitHub Actions | Pass (2.3) |
| Emotion theme both brands × modes + token-parity test | Pass (2.4) |
| Outlined brand SVGs (OQ-13 / RISK-0006) | Pass (2.4) |
| Shared atoms + 13 icons, a11y props | Pass (2.5) |
| Navigation shell + overlay + LoadingGate + error boundary | Pass (2.6) |
| OQ-33 closed in doc 10 | Pass (2.6) |
| Final gate `tsc · jest · eslint · prettier` | Pass locally |

## Doc drift — reconciled

| Area | Change |
|------|--------|
| `architecture/10-observability.md` | OQ-33 closed; §5.2 fallback UI matches shipped `RootErrorBoundary` (v0.1.3) |
| `registries/risks.yml` | RISK-0006 closed (outlined wordmarks in 2.4) |
| `architecture/assets/README.md` | Outlined-wordmarks runtime pointer updated (2.4) |
| App README | Boot instructions describe navigation shell, not template hello screen |

No architecture doc claims the cold-start three-query `LoadingGate` is active — it remains hidden until STEP-3/4.

## Post-merge boot fixes (app PR #4)

After PR #3 merged, iOS simulator showed a white screen. Root cause: React Navigation requires `react-native-gesture-handler`; `PersistGate` loading rendered before theme context; and several shell wrappers lacked `flex: 1`.

| Fix | File(s) |
|-----|---------|
| Add `react-native-gesture-handler` | `package.json`, `index.js`, `App.tsx`, `ios/Podfile.lock` |
| `PersistLoading` (rehydrate UI outside `ModeProvider`) | `src/app/shell/PersistLoading.tsx`, `App.tsx` |
| Provider order: `ThemeProvider` before `PersistGate` | `App.tsx` |
| `RootErrorBoundary` + navigator `contentStyle` + `AppShell` flex | shell + navigation files |
| Jest: `react-native-gesture-handler/jestSetup` | `jest.setup.js`, `jest.config.js` |

Tier A gate re-run green after fixes.

## Findings

### Accepted (no fix required)

| Finding | Disposition |
|---------|-------------|
| T3 / navigation-config tests absent | By design — RISK-0001; doc 12 §3.5 |
| Jest worker may not exit cleanly after App test (PersistGate / NetInfo listener) | Non-blocking; all 8 tests pass; revisit if CI hangs |
| `selectIsAuthenticated` stub stays `false` | Expected until STEP-4 |
| `@react-native/new-app-screen` still in `package.json` but unused | Minor cleanup candidate; not blocking merge |

### None blocking

No high-severity defects found against the STEP-2 scope.

## Test gate

```sh
npx tsc --noEmit && CI=true npm test && npx eslint . && npx prettier --check .
```

All green on 2026-08-18 (including post-merge boot-fix PR #4).

## Manual smoke (owner)

Before sign-off binary work (STEP-6), confirm on device:

- [x] Boot → Welcome placeholder (auth stack) — iOS simulator 2026-08-18
- [ ] Airplane mode → no-internet overlay + REINTENTAR
- [ ] Tabs (when auth stub toggled) → Home + three ComingSoon routes

## PRs

| Repo | Branch / PR | Scope |
|------|-------------|-------|
| `quasar-disney-mobile-app` | `step-0002-app-scaffold` · [#3](https://github.com/raulangelj/quasar-disney-mobile-app/pull/3) (merged) | 2.3–2.6: tooling through navigation shell |
| `quasar-disney-mobile-app` | `step-0002-boot-fixes` · [#4](https://github.com/raulangelj/quasar-disney-mobile-app/pull/4) | Post-merge iOS boot + Jest gesture-handler mock |
| `quasar-disney-mobile-docs` | `step-0002-app-scaffold` · [#3](https://github.com/raulangelj/quasar-disney-mobile-docs/pull/3) (merged) | OQ-33, RISK-0006, doc drift |
| `prompts` | `main` | STEP-2 substeps 2.4–2.6 Done; archive + STEP-2 Done |

## After merge

1. ~~Archive `Upcoming Prompts/quasar-disney-mobile-STEP-2-*` → `prompts/001-poc/step-0002/`~~ Done 2026-08-18
2. ~~Mark STEP-2 **Done** in `prompts/STEP-index.md`~~ Done 2026-08-18
3. ~~Flip PLAN definition-of-done review/archive checkboxes~~ Done 2026-08-18

## Next action

**STEP-3** (Andres — contract + mocks) continues on `step-0003-contract-mocks`. **STEP-4** / **STEP-5** depend on STEP-2 + STEP-3 on `main`. Merge app PR #4 to complete STEP-2 on trunk.
