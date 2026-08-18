# quasar-disney-mobile — STEP-3 REVIEW (Contract types, baseApi & mocks)

**Date:** 2026-08-18
**Branch:** `step-0003-contract-mocks` (app + docs), merged to `main`
**Status:** Passed

Final cross-read of substeps 3.1–3.5 against the STEP-3 PLAN definition of done, doc 11's
contract, doc 12's must-cover list, the import rules, and the Tier A JS gate. Doc drift was
reconciled in the docs hub as each substep landed; this review adds the phase-README row STEP-2's
archive missed. No PRs: `gh` is not installed on this machine, so both repos were merged locally
with `--no-ff` after the gate ran green on the merged branch.

## What was checked

1. **Definition of done** — every PLAN checkbox against disk on the merged `main`.
2. **Substep completeness** — 3.1–3.5 marked Done in the PLAN and `prompts/STEP-index.md`.
3. **Contract fidelity** — `src/api/types/` against doc 11 §7/§8; the five paths against §5; page
   sizes against §6.3.
4. **The load-bearing behaviors** — doc 11 §11.3's table, row by row, at its assigned tier.
5. **Import rules** — DF1 (axios/fetch), DF5/A5 (feature ↮ feature, shared ↮ features), and
   doc 03 §8.2's "the API module never imports a feature".
6. **Test plan** — every row of the PLAN's test plan; doc 12 §3's must-cover list; the gate.
7. **Risk register** — nothing new to accept; RISK-0008/0016/0017 unchanged and still open.

## Definition of done — PASS

| Criterion | Result |
|-----------|--------|
| `src/api/types/` transcribes doc 11 §7 completely | Pass (3.1) — `card`, `container`, `envelope`, `auth`, `errors`, barrel |
| Doc 11 §2.2 handover discharged, Version Log bumped, README contract-of-record line | Pass (3.1) — doc 11 v0.4.0; app README lines 3 and 72 |
| One axios instance; request interceptor on ops 2–5 only; response interceptor → `ApiError`; neither clears | Pass (3.2) |
| `baseQueryWithAuth` switches on `code`; clears **and** `resetApiState()` on `UNAUTHORIZED`; neither on `INVALID_CREDENTIALS` | Pass (3.2), proven end to end (3.5) |
| `baseApi` with `reducerPath: 'api'` and empty endpoints | Pass (3.2) |
| `axios-mock-adapter` on the **same** instance, five operations, envelope, both cursors, mock JWT `sub`/`exp`/`iat` 7-day TTL, `exp` validation on 2–5 | Pass (3.3) |
| Latency / clock / failure injection are constructor parameters; default latency nonzero in 400–600 ms; no runtime toggle | Pass (3.3) — reachability grep in `createMockAdapter.test.ts` |
| Demo fixtures typed with fixed UUIDs + seven placeholder-art files + invariant test; `*.factory.ts` factories | Pass (3.4) |
| Every in-scope must-cover row has a test at its tier | Pass — see the table below |
| Test plan complete per substep | Pass |
| `tsc --noEmit` · `jest` · `eslint` · `prettier --check` green | Pass — 197 tests / 23 suites, ~5 s |
| Architecture changes reflected with Version Log bumps (Q1 seam, doc 03 §8.1 folders) | Pass — doc 03 v0.4.3/v0.4.4/v0.4.5, doc 11 v0.4.0/v0.4.1, doc 12 v0.1.3 |
| STEP review passed; index updated; archived to `prompts/001-poc/step-0003/` | This document |

## Doc 11 §11.3 — behavioral coverage

| Test | Tier | Where |
|------|------|-------|
| `nextCursor` exhaustion on both axes, cursor trusted over count arithmetic | T2 | `integration/pagination.test.ts` |
| **401 scoping** — login failure keeps the session; `/me` 401 clears it | T2 | `integration/authScoping.test.ts` |
| Expired mock JWT → `UNAUTHORIZED` (operations 2–5) | T2 | `integration/expiredToken.test.ts` |
| Both transports normalize to one `ApiError` | T2 | `integration/errorNormalization.test.ts` |
| Injectable failure is reachable from tests only | T1 | `mocks/createMockAdapter.test.ts` (source walk) |
| Unknown `variant` drops the row and warns | T1, **STEP-5** | Out of scope here; the `console.warn` seam is guarded by `integration/consoleObservability.test.ts` |

## Import rules — PASS

- **DF1:** no file outside `src/api/` imports `axios` or `axios-mock-adapter`; the only `fetch(`
  occurrences are `NetInfo.fetch()` in the connectivity overlay, which is not the global.
- **DF5 / A5:** no auth ↔ storefront import; `shared/` imports no feature.
- **API module → features:** none. The two `features/` strings under `src/api/` are docstrings in
  `baseApi.ts` naming who will inject which endpoints.
- **One deliberate test-time exception:** `src/api/integration/t2World.factory.ts` imports
  `src/app/store/createStore`. Doc 03 §8.2 governs runtime paths and doc 12 §4.1 puts test-time
  imports out of its scope; the file is `*.factory.ts`, and `mocks/factories.test.ts` now asserts
  that no shipped source imports one. Recorded in doc 03 §8.1 (v0.4.5).

## Findings

### Accepted (no fix required)

| Finding | Disposition |
|---------|-------------|
| `resetApiState()` aborts the in-flight query, so a caller awaiting an `UNAUTHORIZED` query sees it torn down rather than errored | Intended — the session is over. Recorded in doc 11 §8.4 (v0.4.1) so the absent error is not later read as a defect |
| `createStore()` gained an optional `extraMiddleware` slot | Needed: a `configureStore` store is sealed and the 401 reaction is dispatched from inside the chain, where a `store.dispatch` wrapper cannot observe it. Doc 03 v0.4.5 |
| RNTL is not a dependency; T2 drives test-local endpoints through the store | Correct order — the API module owns no hooks. Doc 12 v0.1.3; `renderHook` arrives with STEP-4/5 |
| Doc 12 §3.1's **boot gate** row untested | Needs the shell — **STEP-6**. Its persist-whitelist sibling is already covered at T1 by `createStore.test.ts` (STEP-2.3), still green with `api` registered |
| `src/app/` and `src/shared/ui/` coverage is low | By design — RISK-0001 (T3 in 1b), RISK-0016 (no numeric gate in 1a). `src/api/client/` 98.6%, `src/api/mocks/` 99.1% |
| Two STEP-3.3/3.4 test files edited by 3.5 | `createMockAdapter.test.ts`'s source walk now treats `*.factory.ts` as test code (doc 12 §4.1 already did), and `factories.test.ts` gained the reachability guard that keeps that exclusion honest |

### Follow-up for STEP-4 (not blocking)

The T2 world supplies `auth.accessToken` through an `injectStore` view, because the scaffold's stub
auth slice has no token field. When STEP-4's real slice lands — with the `sessionCleared`
`extraReducers` case doc 03 §8.2 describes — `t2World.factory.ts` should read the token from the
store and drop the overlay, and the 401-scoping suite gains a state-level assertion it cannot make
today. **Owner: Raul Angel, in STEP-4.**

### None blocking

No high-severity defects found against the STEP-3 scope.

## Doc drift — reconciled

| Area | Change |
|------|--------|
| `architecture/11-interface-contracts.md` | v0.4.0 handover (3.1); v0.4.1 §8.4 records the observed `UNAUTHORIZED` reaction (3.5) |
| `architecture/03-architecture-overview.md` | v0.4.3 `sessionCleared` seam; v0.4.4 `src/shared/assets/`; v0.4.5 shell wiring + `src/api/integration/` |
| `architecture/12-test-strategy.md` | v0.1.3 — T2 row corrected; RNTL deferred to the first feature hook |
| `prompts/001-poc/README.md` | **STEP-2's row was never added when it was archived** — this review adds STEP-2 and STEP-3 |
| `registries/risks.yml` | No change. Nothing in this STEP was deferred that is not already carried |

## Test gate

```sh
npx tsc --noEmit && npx jest --coverage && npx eslint . && npx prettier --check .
```

All green on 2026-08-18 — on `step-0003-contract-mocks`, again after merging `origin/main` into it
(the `react-native-gesture-handler` boot fix), and once more on `main` before pushing. The suite
was run twice consecutively with identical results and once with `--runInBand`: order-independent,
~5 s, well inside doc 12 §7.1's two-minute budget.

## Merges

| Repo | Branch | Result |
|------|--------|--------|
| `quasar-disney-mobile-app` | `step-0003-contract-mocks` | Merged `--no-ff` to `main` (`75144a0`) |
| `quasar-disney-mobile-docs` | `step-0003-contract-mocks` | Merged `--no-ff` to `main` (`c6c3873`) |
| `prompts` | `main` | Substep statuses, archive, STEP-3 Done |

No pull requests: `gh` is unavailable locally. If the team wants the diff reviewed on GitHub, the
branch is still pushed and can be compared against `main`.

## After merge

1. Archive `Upcoming Prompts/quasar-disney-mobile-STEP-3-*` → `prompts/001-poc/step-0003/`
2. Mark STEP-3 **Done** in `prompts/STEP-index.md`; add the phase-README rows
3. Flip the PLAN's definition-of-done checkboxes

## Next action

**STEP-4** (Raul — auth feature) and **STEP-5** (Andres — storefront) both unblock now that
STEP-3 is on `main`; STEP-4 is already In progress. STEP-4 owns the auth slice that reduces
`sessionCleared` and the follow-up noted above.
