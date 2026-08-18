# quasar-disney-mobile — STEP-3 PLAN: Contract types, baseApi & mocks

**Phase:** Phase 1 — POC (1a)
**Owner:** Andres Montoya
**Status:** Done 2026-08-18 — review passed, merged to `main`, archived
**Date:** 2026-08-17
**Branch:** `step-0003-contract-mocks`
**Repos (projection):** `quasar-disney-mobile-app` (primary) → `quasar-disney-mobile-docs` (doc 11 handover, doc 03 §8.1, Version Logs) → `prompts` (index). Merge the app repo first, then the docs hub in the same PR window.

> STEP-3 turns `architecture/11-interface-contracts.md` from a normative document into running
> TypeScript: the wire types, the axios instance and its two interceptors, the RTK Query
> `baseApi` with `baseQueryWithAuth`, and an `axios-mock-adapter` transport on that same instance
> with typed fixtures and constructor-injected latency, clock, and failure seams. When it lands,
> doc 11 §2.2's inversion flips — **the TS types become the authoring source** — and STEP-4 and
> STEP-5 have generated hooks to build against.

## Motivation

Everything above the API boundary in Phase 1a is blocked on this STEP. Auth (STEP-4) and
storefront (STEP-5) consume `injectEndpoints` hooks on `baseApi`; neither can start until
`baseApi`, `baseQueryWithAuth`, and a transport exist. Doc 02 §9 assigns this to Dev B in parallel
with the scaffold precisely because it is the other half of the critical path.

Three things make it more than plumbing:

1. **Doc 11 §2.2's inversion.** Today the contract of record lives in the docs hub and the
   authoring source does not exist. §3.1 states the transcription as an *obligation* so the types
   get written from a specification rather than from memory of a conversation. This STEP discharges
   it.
2. **ADR-0017 / ADR-0020's 401 scoping is executable here or nowhere.** A login failure and an
   expired token are both `401`. If the session-clearing reaction fires on the first, a wrong
   password bounces the user out of the credentials screen and **criterion F2 — half of what the
   2026-08-18 demo exists to show — never renders**. Because mocks sit on the *same* axios instance
   (ADR-0020), that branch runs and is tested in 1a instead of first executing against a real
   backend in Phase 3.
3. **`tsc --noEmit` is the contract test** (doc 11 §11.2), and it is only meaningful if the
   fixtures are declared `Container[]` / `Card[]` and never `any`. An untyped fixture import would
   make the entire type-as-contract strategy decorative.

**Not in this STEP:** any React Native UI, any screen, any feature `api.ts`, the auth slice, the
storefront hooks, the shell wiring. STEP-3 ships `src/api/` and nothing that renders.

## Dependency gate — read before starting

**The application repo does not exist on this machine yet.** `registries/repos.yml` still lists
only the two docs repos. **STEP-2 (Scaffold app repo & foundation) is in flight under its own
owner, Raul Angel**, who is creating the app repo — so this is a wait, not a blocker. STEP-2's
index row sets the gate: *"STEP-3 may start once the repo and `src/api/` tree exist."*

Concretely, STEP-3 needs these from STEP-2 before substep 3.1 can run:

| Needed from STEP-2 | Used by | Doc obligation |
|---|---|---|
| `quasar-disney-mobile-app` repo created, registered in `registries/repos.yml`, deps installed | all substeps | doc 11 §3.1, doc 03 standing rule |
| `src/api/` tree (`types/`, `client/`, `mocks/`) and `src/features/`, `src/shared/` | 3.1–3.4 | doc 03 §8.1 |
| `createStore()` **factory**, not a module-level singleton | 3.5 | doc 12 §4.2, §11 item 1 |
| Jest mapping `@env` to a committed stub; `react-native-encrypted-storage` and NetInfo mocked | 3.3, 3.5 | doc 12 §4.3, §5, §11 item 2 |
| `.env.example` with `API_BASE_URL`, `DEMO_EMAIL`, `DEMO_PASSWORD` | 3.2, 3.3 | doc 09 §4.2 |
| ESLint (four rules) + Prettier + `.github/workflows/ci.yml` Tier A | 3.5 | doc 12 §7.1, §11 items 3–4 |

**If STEP-2 slips past its checkpoint** (RISK-0003 — the original 2026-08-15 scaffold date has
already passed), doc 02 §9's recorded mitigation applies: the STEP-3 owner **stops contract work
and pairs on the scaffold**. The scaffold is the single point of failure for the whole schedule.

### Working alongside STEP-2 (overlap heads-up)

STEP-2 and STEP-3 are **in flight at the same time on the same repo** — by design (doc 02 §9's
parallel window), not by accident. `runbooks/collaboration.md` asks that the overlap be named
rather than discovered:

| | STEP-2 — Raul Angel, `step-0002-*` | STEP-3 — Andres Montoya, `step-0003-contract-mocks` |
|---|---|---|
| Owns | `src/app/`, `src/shared/{theme,ui,i18n,analytics}/`, `src/features/*` stubs, RN + native config, ESLint/Prettier/Jest config, CI workflow, `.env.example` | **`src/api/` only**, plus `src/shared/assets/placeholder-art/` (3.4) |
| Touches in the other's area | pre-registers `baseApi.reducer` / `baseApi.middleware` slots as stubs | nothing — 3.5 *consumes* `createStore()`, it does not edit it |

The one collision hazard doc 02 §9 names is the shared registration files (root store, navigation
config, theme index). STEP-2 creates those with **both features' slots pre-registered as stubs**,
so each branch edits only its own line. `src/shared/assets/` is a new folder under STEP-2's
directory: 3.4 creates it and flags the doc 03 §8.1 addition rather than reshaping anything
STEP-2 owns.

## Decisions already locked

- root `.throughstone/local-user.md` — read **Experience level** before user-facing questions or
  explanations, and **Communication style** before planning discussions.
- `registries/risks.yml` — **RISK-0003** (scaffold is the schedule risk), **RISK-0004** (backend
  team inherits a contract they did not draft), **RISK-0008** (shared demo login is the identity
  model), **RISK-0016** (coverage reported, not gated), **RISK-0017** (the A2 lint rule is
  pattern-matching).
- `architecture/12-test-strategy.md` — the tier definitions (T0/T1/T2), §3's must-cover list, §4's
  isolation rules, §7.1's merge gate, and §11 item 5, which is an obligation on *this* STEP:
  latency, clock, and failure injection are **constructor parameters**, and test factories ship
  alongside the demo fixtures.
- `architecture/11-interface-contracts.md` — the whole document, but especially §5 (five
  operations), §6.1–§6.4 (envelope, two cursors, conventions, ISO `expiresAt`), §6.5 (transport
  profiles), §7 (payload shapes), §8 (error model and the 401 collision), §9.2 (mock handlers
  validate the token), §11.2–§11.3 (typed fixtures; the behavioral test list), and §12 (the
  same-PR update rule).
- **ADR-0020** — one RTK Query `baseApi`; axios is the engine; request interceptor attaches
  `Authorization`, response interceptor maps status → `ApiError`, `baseQueryWithAuth` reacts;
  Phase 1 mocks are `axios-mock-adapter` on the same instance.
- **ADR-0017** — the session-clearing 401 policy lives **above** the raw interceptor, in
  `baseQueryWithAuth`, and switches on `code`, never on a path.
- **ADR-0016** — TypeScript interfaces are the contract; no OpenAPI artifact until OQ-10/OQ-34.
- **ADR-0002** (amended by 0020) — axios stays confined to `src/api/client/`.
- **ADR-0006 / ADR-0007** — two `Container[]` endpoints; cards live in `resources`, not `items`;
  `progress` never appears in a HomeFeed response.
- **ADR-0015** — config arrives through the `react-native-dotenv` babel transform (`@env`).
- **ADR-0003** — only the auth slice is persisted; the RTK Query cache is never persisted.
- `architecture/03-architecture-overview.md` §8.1–§8.2 — source layout and import rules.
- `architecture/04-data-model.md` §1.2–§1.5, §6–§7 — entity fields, the 7-day mock `exp`, and the
  cold-start fetch topology.
- `architecture/09-environments.md` §4.2 (three env keys), §6.2 (**injection is a test seam, not a
  runtime toggle — no debug menu, no shake gesture, nothing in the release binary**).
- `architecture/16-identity-auth.md` §1 (the auth decision is the mock adapter, not a screen `if`),
  §6 (mock JWT claims `sub` / `exp` / `iat`; `Authorization: Bearer`).
- `architecture/15-native-app-architecture.md` §8 — page sizes, opaque `nextCursor`, cursors live
  in the RTK Query cache.
- `coding-standards/typescript.md` and `coding-standards/api.md` — `strict: true`, no `any`,
  `camelCase.ts` for non-components, **colocated `*.test.ts`** (not `__tests__/`), `console` is the
  sanctioned log mechanism, `ApiError` is the error type.

## Substeps

| # | Title | Status | Produces | Depends on | Open questions |
|---|-------|--------|----------|------------|----------------|
| 3.1 | Wire types — transcribe doc 11 §7 | **Done** 2026-08-18 | `src/api/types/` (`card.ts`, `container.ts`, `envelope.ts`, `auth.ts`, `errors.ts`, `index.ts`), page-size constants; doc 11 §2.2/§3 handover + Version Log; app README contract-of-record line | STEP-2 repo + `src/api/` tree | Q2 |
| 3.2 | axios client, interceptors, `baseQueryWithAuth`, `baseApi` | **Done** 2026-08-18 | `src/api/client/` (instance, `paths.ts`, `injectStore.ts`, request + response interceptors, `axiosBaseQuery.ts`, `baseQueryWithAuth.ts`), `src/api/sessionCleared.ts`, `src/api/baseApi.ts` | 3.1 | **Q1** |
| 3.3 | Mock adapter — seams, mock JWT, five handlers | **Done** 2026-08-18 | `src/api/mocks/createMockAdapter.ts`, `jwt.ts`, `cursor.ts`, `handlers/*.ts` | 3.1, 3.2 | — |
| 3.4 | Demo fixtures, placeholder art, test factories | **Done** 2026-08-18 | `src/api/mocks/fixtures/` (`ids.ts`, `artwork.ts`, `homeFeed.ts`, `continueWatching.ts`, `user.ts`, `index.ts`, `fixtures.test.ts`), `src/shared/assets/placeholder-art/*.svg` (7 files), `card.factory.ts` / `container.factory.ts` / `page.factory.ts` + `factories.test.ts`; doc 03 §8.1 + Version Log | 3.1, 3.3 | Q4 **closed** |
| 3.5 | T2 integration tests + final verification gate | **Done** 2026-08-18 | `src/api/integration/` — `t2World.factory.ts` + four T2 suites; shell wiring (`createStore()` registers `baseApi`, `App.tsx` calls `injectStore`); docs 03 / 11 / 12 Version Logs; green `tsc --noEmit` · `jest` · `eslint` · `prettier --check` | 3.2, 3.3, 3.4; STEP-2's `createStore()` and Jest config | — |

Each substep has a self-contained prompt in this folder
(`quasar-disney-mobile-STEP-3.M-PROMPT.md`), runnable cold in a fresh chat.

**3.1 outcome (2026-08-18).** `src/api/types/` transcribes doc 11 §7 and §8 —
`card.ts`, `container.ts`, `envelope.ts`, `auth.ts`, `errors.ts`, `index.ts` — with the three
page-size constants as the only runtime values. Doc 11 is at **v0.4.0**: §2.1/§2.2/§3 record
`src/api/types/` as the authoring source and the inversion as resolved, and the app README
carries the contract-of-record line. **No wire-shape change** — §7 needed no correction, so
§4.3/§12 did not apply. Q2 closed as recommended: `src/types/env.d.ts` already declared all
three `@env` keys (STEP-2.3), so 3.1 absorbed nothing. `tsc --noEmit` · `jest` · `eslint` ·
`prettier --check` all green on `step-0003-contract-mocks`; no `any` in the new files. Nothing
deferred, so `registries/risks.yml` is unchanged.

**3.2 outcome (2026-08-18).** `src/api/client/` ships `paths.ts`, `axiosInstance.ts`,
`injectStore.ts`, `requestInterceptor.ts`, `responseInterceptor.ts`, `axiosBaseQuery.ts`, and
`baseQueryWithAuth.ts`; `src/api/sessionCleared.ts` and `src/api/baseApi.ts` (empty endpoints,
`reducerPath: 'api'`) sit beside them. `axios@^1.19.0` installed — already a doc 03 §7 hard
dependency, so no doc change. The 401 policy switches on `code` with **no path comparison
anywhere** in `baseQueryWithAuth`; the response interceptor carries the ADR-0017 comment pointing
at it. The `baseApi` ↔ `baseQueryWithAuth` cycle is broken with a deferred `require` typed via
`typeof import(…)`, commented so it is not "tidied" back to the top of the file. Doc 11 §8.2's
status → `code` table needed no correction, so §4.3/§12 did not apply. **Q1 closed** as
recommended; doc 03 is at **v0.4.3** with the seam in §8.2 and `sessionCleared.ts` in §8.1's tree.
36 T1 tests across the three files in the prompt's table; `tsc --noEmit` · `jest` · `eslint` ·
`prettier --check` all green; no `any`. Nothing deferred, so `registries/risks.yml` is unchanged.

**3.3 outcome (2026-08-18).** `src/api/mocks/` ships `createMockAdapter.ts`, `context.ts`,
`jwt.ts`, `cursor.ts`, `base64url.ts`, and `handlers/` (the five operations plus `guard.ts`,
`reply.ts`, `containers.ts`). `axios-mock-adapter@^2.1.0` installed as a **runtime** dependency —
already a doc 03 §7 hard dependency (Phase 1; removed in Phase 3), so no doc change. The adapter
runs on the shared `axiosInstance`, so handler tests exercise both interceptors and assert on the
normalized `ApiError` rather than an axios shape. All three seams are constructor parameters and
nothing else; the default latency draws from **400–600 ms** and one test pays the real delay to
prove the default is awaited, not merely in-band. Mock JWTs carry `sub` / `exp` / `iat` only, minted
with a hand-rolled base64url codec so no JWT — or `Buffer` / `btoa` — dependency outlives the mock;
`expiresAt` is ISO 8601 UTC on the wire while the claim stays numeric (doc 11 §6.4). ADR-0006 is
enforced **in the handler**: a home-feed dataset containing a `progress` container, or a hero
outside the first page, answers `500 INTERNAL` instead of serving a shape the contract forbids.
Doc 09 §6.2's boundary is pinned by a test that greps every shipped `src/**/*.ts` for a `failures:`
construction — only `mocks/context.ts` and `mocks/createMockAdapter.ts` may match, so a debug
affordance fails the suite rather than reaching a stakeholder's build. **No wire-shape change** —
doc 11 §7.4 needed no correction, so §4.3/§12 did not apply; the mock caps `limit` at 50, which
doc 11 §6.3 already licenses ("a hint the server may cap"). 103 T1 tests across the nine files in
the prompt's table; `tsc --noEmit` · `jest` · `eslint` · `prettier --check` all green; no `any`.
Nothing deferred, so `registries/risks.yml` is unchanged.

**Carried into 3.4 (not a gap in 3.3).** `MockDataset` defaults to an empty dataset with a
placeholder user, because the demo fixtures are 3.4's file. Handler tests therefore build their own
minimal containers inline rather than importing `*.factory.ts`, which 3.4 introduces; they should
move to the factories when those land.

**3.4 outcome (2026-08-18).** The seven placeholder-art SVGs are copied unchanged into
`src/shared/assets/placeholder-art/` (no new art authored — DF10), and `src/api/mocks/fixtures/`
ships `DEMO_DATASET`: a typed `Container[]` home feed of **one `hero` plus 15** rows (13
`standardPortrait`, 2 `standardLandscape` so STEP-7 has something real to unpark, **no `progress`
row** — ADR-0006), a one-row `progress` Continue Watching set carrying the three CW-only fields,
and the `/me` user. All 222 identifiers are fixed committed UUIDs in `ids.ts`, and every row holds
12 cards so the horizontal cursor is exercised by the demo data itself, not only by a test. Display
copy is Spanish and entirely invented. **Q4 is closed as planned:** `artwork` values are plain
filename strings and `artwork.ts` records that STEP-5 owns the render-time lookup. The three
factories ship as `*.factory.ts` beside the module owning the type, with the "test-time import is
not an import-rule violation" and "not reachable from the app entry" rules stated in their
docstrings. `fixtures.test.ts` asserts structure only — composition, ratio-per-variant, artwork
files present on disk, unique ids, `progress` fields confined to the `progress` row — and no test in
the repo asserts demo content. Gates: `tsc --noEmit`, `eslint`, `prettier --check` clean; `jest`
165 tests green, including 3.3's handler suite unchanged.

**Carried into 3.5 (not a gap in 3.2), and discharged there.** The shell did not register
`baseApi.reducer` / `baseApi.middleware` in `createStore()` or call `injectStore(store)` in
`src/app/App.tsx` — both lines were pre-marked as stubs by STEP-2 and both were outside 3.2's
stated scope. **3.5 made those two edits** (see its outcome below).

**3.5 outcome (2026-08-18).** `src/api/integration/` ships a `t2World.factory.ts` that builds one
wired world per test — real `createStore()`, `baseApi.middleware`, the shared axios instance, the
adapter at zero latency on a frozen clock — plus four suites: **401 scoping** (login failure keeps
the session and its cache; `/me` on an expired token dispatches `sessionCleared` **and**
`resetApiState()` and empties the cache; both back to back in one store; a source grep proving no
path comparison in `baseQueryWithAuth`), **expired JWT** across operations 2–5 asserted from both
the transport and the store, **two-axis `nextCursor` exhaustion** including the trust-the-cursor
case on each axis (a final page whose length equals `limit` while its cursor is `null`), and
**`ApiError` normalization** across a mocked `4xx`, a mocked `5xx`, a transport failure with no
response, and an injected feed failure. A fifth file guards that nothing in the setup chain
silences `console.warn` (doc 12 §5).

The shell wiring landed with it: `createStore()` registers the reducer and middleware, keeps `api`
off the persist whitelist, and gains an optional `extraMiddleware` slot — a `configureStore` store
is sealed, and the 401 reaction is dispatched from *inside* the chain where a wrapper around
`store.dispatch` never sees it. `App.tsx` calls `injectStore(store)`.

**One finding, recorded in doc 11 §8.4 (v0.4.1):** `resetApiState()` aborts the in-flight query
that triggered it, so a caller awaiting an `UNAUTHORIZED` query observes a torn-down query rather
than an error result. That is the intended reaction — the session is over — but the obvious test
cannot be written against RTK Query, so the doc now says so. Docs 03 (v0.4.5) and 12 (v0.1.3)
were bumped too; doc 12's T2 row is corrected to record that **RNTL is not yet a dependency** —
the API module owns no hooks, so T2 drives test-local endpoints through the store, and
`renderHook` arrives with STEP-4/5.

Gates: `tsc --noEmit`, `eslint`, `prettier --check` clean; `jest` **197 tests / 23 suites green**
in ~5 s, identical across two consecutive runs and under `--runInBand`. Coverage is reported, not
gated (RISK-0016): `src/api/client/` 98.6%, `src/api/mocks/` 99.1%.

## Test plan

Tests ship **with** each substep (T0 + T1 run before that substep is marked done), and 3.5 adds
the T2 integration layer and runs the full merge gate. This split is deliberate: doc 12 §2 keeps
T1 and T2 separate because the 401-scoping behavior that protects F2 is invisible to any test that
does not have a real store, `baseApi.middleware`, and the adapter wired together — and discovering
it broken at the end of the STEP is the expensive outcome.

| Test tier / surface | Substep(s) | Tests to create or update | Run timing | Command / gate | Notes |
|---------------------|------------|---------------------------|------------|----------------|-------|
| T0 — type check | 3.1–3.5 | — (the types and typed fixtures *are* the test) | Per substep | `npx tsc --noEmit` | Doc 11 §11.2. Only meaningful because fixtures are `Container[]` / `Card[]`, never `any` |
| T1 — unit | 3.2 | Response interceptor maps each status → the right `ErrorCode`; request interceptor attaches `Bearer` on ops 2–5 and **not** on `/auth/login`; `baseQueryWithAuth` clears on `UNAUTHORIZED` and does **not** on `INVALID_CREDENTIALS` (switching on `code`, never a path) | Per substep | `npx jest src/api/client` | Doc 11 §8.3, §8.4; ADR-0017 |
| T1 — unit | 3.3 | Credential compare from `@env` — success and `INVALID_CREDENTIALS`; Bearer **presence and `exp`** on ops 2–5 → `UNAUTHORIZED`; cursor exhaustion on both axes at handler level; unknown container → `NOT_FOUND`; failure injection reachable only via the constructor; **default latency nonzero and inside 400–600 ms** | Per substep | `npx jest src/api/mocks` | Doc 12 §3.1, §3.3; doc 11 §9.2; doc 09 §6.2 |
| T1 — unit | 3.4 | The one **fixture-invariant** test against the real demo fixtures: first HomeFeed page is exactly one `hero` plus 15 others; no `progress` container ever appears in a HomeFeed response; every card carries the artwork ratios its container's variant needs | Per substep | `npx jest src/api/mocks/fixtures` | Doc 12 §4.1. Tests otherwise assert against **factories**, never demo content |
| T2 — integration | 3.5 | **401 scoping** end to end (login failure keeps the session; `/me` 401 clears it **and** `resetApiState()`); two-axis `nextCursor` exhaustion through the store, trusting `nextCursor` over count arithmetic; expired mock JWT → `UNAUTHORIZED`; both transports normalize to one `ApiError` | Final substep | `npx jest src/api` | Doc 11 §11.3; doc 12 §3.1–§3.3. Real `createStore()` + `baseApi.middleware` + axios + adapter; endpoints defined test-locally via `injectEndpoints` so feature ownership stays with STEP-4/5 |
| Security / authorization | 3.3, 3.5 | Covered by the Bearer-presence/`exp` rows above — an unauthenticated or expired call to operations 2–5 is rejected with `UNAUTHORIZED` | Per substep + final | as above | Doc 11 §9.2. Doc 16 §4: binary session gate, no roles or scopes to test |
| Lint / format | 3.5 | — | Final substep | `npx eslint . && npx prettier --check .` | Doc 12 §7.1. `import/no-restricted-paths` (A5/DF5) and the axios restriction (DF1) must pass with `src/api/` as the only axios importer |
| T3 — component render | — | **Not applicable.** STEP-3 ships no UI | — | — | Deferred to Phase 1b by doc 02 (RISK-0001) |
| End-to-end / device | — | **Not applicable.** Declined for Phase 1 | — | — | Doc 12 §2, RISK-0015. The manual release smoke is the e2e layer |
| Performance / load | — | **Not applicable** beyond the latency-default assertion in 3.3 | — | — | Doc 12 §9 declines a perf tier; there is no service to load |

**The gate that proves the STEP is done:** `tsc --noEmit` · `jest` · `eslint` · `prettier --check`
green on the branch — the same four commands Tier A runs on every push (doc 12 §7.1, ADR-0018),
inside the two-minute speed budget with adapter latency zeroed in tests.

## Open questions

| # | Question | Owner | Recommendation |
|---|---|---|---|
| **Q1** | Where does the session-clear action that `baseQueryWithAuth` dispatches live? The auth slice is STEP-4's, and the API module must not import a feature (doc 03 §8.2). | Andres Montoya (decide in 3.2) | **Define `sessionCleared` as an RTK `createAction('session/cleared')` owned by the API module** (`src/api/sessionCleared.ts`). `baseQueryWithAuth` dispatches it plus `baseApi.util.resetApiState()`; STEP-4's auth slice reduces it via `extraReducers`. Keeps the dependency direction feature → api, and lets 3.2 be built and tested with no auth slice in existence. This is a new seam no doc records — 3.2 must add it to doc 03 §8.2 and bump the Version Log, or write an ADR if it is contested. **Closed 2026-08-18 as recommended**: `src/api/sessionCleared.ts`, recorded in doc 03 §8.2 (v0.4.3). Not contested, so no ADR. |
| Q2 | Who owns the `@env` module type declarations (`src/types/env.d.ts`)? | STEP-2 owner | STEP-2 (it configures the babel transform and the Jest stub). 3.1 adds the declaration only if it is missing, and says so. |
| Q3 | Application repo name (**OQ-18**, still open in doc 03). | Eng leadership | The STEP index already writes it as `quasar-disney-mobile-app`; STEP-3 uses that name and does not reopen it. |
| Q4 | Who maps a fixture's artwork string to a renderable asset? | STEP-5 owner | **STEP-5.** `Card.artwork` values are plain `string` on the wire, as a real backend would send. STEP-3 copies the SVGs and has fixtures reference them by exact filename; the render-time lookup belongs to the storefront card component. Recorded so it is not discovered on Sunday. |

## Ground rules

- **Calibrate communication from root `.throughstone/local-user.md`.** Don't copy its values here.
- **Plan interactively.** Scope was confirmed with the owner on 2026-08-17: five substeps, tests
  per substep plus a final gate, and STEP-3 owns both the placeholder-art copy and the fixtures.
- **No React Native UI in this STEP.** No screen, no component, no feature `api.ts`, no slice.
  If a substep finds itself importing from `src/features/` or `src/shared/ui/`, it has left scope.
- **`src/api/` is the only place `axios` may be imported.** DF1 is an ESLint rule, not a habit.
- **No `any`, anywhere — fixtures above all.** Doc 11 §11.2 makes typed fixtures the condition on
  which the whole type-as-contract strategy rests. An untyped JSON import fails this STEP.
- **Injection stays a test seam.** Latency, clock, and failure injection are constructor
  parameters of the mock adapter and nothing else. No runtime toggle, no debug menu, no shake
  gesture, nothing that survives into the release binary (doc 09 §6.2).
- **No module-level mutable state in the adapter.** Constructed per test; cursors and injected
  failure flags must not leak between tests (doc 12 §4.2).
- **Tests assert against factories, not demo fixtures** — with the single fixture-invariant test
  as the deliberate exception (doc 12 §4.1).
- **`console.warn` is spied, never silenced.** Doc 11 §4.4 requires the unknown-variant warn to be
  observable; a blanket console silence in test setup would make that assertion unfallible.
- **Tests ship with the code.** Each substep writes and runs its own T0/T1 before it is done; 3.5
  adds T2 and runs the full gate.
- **Code is documented as it's written.** Every type, function, and method gets a docstring;
  comment the *why* of non-obvious logic. The response interceptor in particular must carry a
  comment saying that the session-clearing 401 policy lives in `baseQueryWithAuth` and why
  (ADR-0017's recorded consequence).
- **Doc 11 §12's update rule is binding on every substep here.** Any change to an operation,
  payload, envelope, or error code updates this doc's tables, the TS types, the mocks, the tests,
  and the app README link **in the same PR**. A wire change that lands without them is an
  incomplete change.
- **Accepted risks stay visible.** If this STEP defers anything, add or update
  `registries/risks.yml` with severity, owner, and revisit trigger.

## Definition of done

- [x] `src/api/types/` exists and transcribes doc 11 §7 completely — `Card`, `AspectRatio`,
      `Container`, `ContainerVariant`, the `{ data, nextCursor }` envelope, the login and `/me`
      DTOs, `ErrorCode`, `ApiError`, and the wire error body.
- [x] **Doc 11 §2.2's handover is discharged:** §2.1/§3 point at `src/api/types/` as the authoring
      source, the doc records that the TS types are now normative, and its Version Log is bumped.
      The app repo README carries the contract-of-record line (doc 11 §3.1 item 2).
- [x] One axios instance in `src/api/client/`, with a request interceptor attaching
      `Authorization: Bearer <jwt>` on operations 2–5 only, and a response interceptor mapping
      status → `ApiError { code, status, message }`. Neither clears the session.
- [x] `baseQueryWithAuth` wraps `axiosBaseQuery`, switches on `code`, clears the session **and**
      calls `resetApiState()` on `UNAUTHORIZED`, and does neither on `INVALID_CREDENTIALS`.
- [x] `src/api/baseApi.ts` exports a `createApi` with `reducerPath: 'api'` and empty endpoints,
      ready for feature `injectEndpoints`.
- [x] `axios-mock-adapter` runs on that **same** instance, serving all five operations with the
      `{ data, nextCursor }` envelope, opaque cursors on both axes, mock JWTs carrying
      `sub`/`exp`/`iat` with a 7-day TTL, and `exp` validation on operations 2–5.
- [x] Latency, clock, and failure injection are constructor parameters; the **default** latency is
      nonzero and inside 400–600 ms; no runtime toggle exists.
- [x] Demo fixtures are typed `Container[]` / `Card[]` with fixed committed UUIDs, reference the
      seven copied placeholder-art files, and satisfy the fixture-invariant test. Test factories
      (`makeCard`, `makeContainer`, `makePage`) ship as `*.factory.ts` beside them.
- [x] Every must-cover row from doc 12 §3.1 (adapter half), §3.3, and doc 11 §11.3 that falls in
      this STEP's scope has a test, at its assigned tier.
- [x] The STEP test plan is complete: each code-changing substep either added/updated its relevant
      tests or records why tests were not applicable.
- [x] All tests named in the STEP test plan pass: `tsc --noEmit` · `jest` · `eslint` ·
      `prettier --check` green on `step-0003-contract-mocks`.
- [x] Any architecture decision this STEP changed is reflected in the docs with a Version Log bump
      or an ADR — in particular Q1's `sessionCleared` seam (doc 03 §8.2) and any folder added to
      doc 03 §8.1.
- [x] STEP review passed; `prompts/STEP-index.md` updated; STEP archived to
      `prompts/001-poc/step-0003/`.
