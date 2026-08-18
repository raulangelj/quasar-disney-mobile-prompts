# quasar-disney-mobile — STEP-3.5: T2 integration tests & final verification gate

> **How to run:** Tell your agent *"run substep 3.5"*. Self-contained — runnable cold. This is the
> last substep of STEP-3.

## Context

Substeps 3.1–3.4 each shipped their own T0/T1 tests. This substep adds the layer none of them
could: **T2 integration** — a real store, `baseApi.middleware`, the axios instance, and the mock
adapter wired together — and then runs the gate that proves the STEP is done.

Doc 12 §2 keeps T1 and T2 separate for one concrete reason:

> Doc 11 §11.3 lists tests that are only meaningful with middleware, error normalization, and the
> adapter wired together — **§8.4's 401 scoping above all**. Testing the auth reducer alone would
> never catch a wrong password bouncing the user out of the credentials screen, which is half of
> criterion **F2**.

That test is the single highest-value thing in this substep. It is also the reason ADR-0017 moved
the 401 policy above the transport in the first place: so that the branch protecting F2 runs on
every push in Phase 1a instead of executing for the first time against a real backend in Phase 3.

## Read these first

- root `.throughstone/local-user.md`
- **`Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md`** — §2 (**T1 vs T2, and why
  they stay separate**), §3.1–§3.3 (the must-cover list), §4.2 (`createStore()` per test), §4.3
  (`@env` stub), §4.4 (latency `0`, frozen clock), §5 (**`console.warn` is spied, not silenced**;
  what is mocked at the native boundary), §7.1 (**what blocks merge**), §7.3 (what is not gated),
  §8 (coverage is reported, not gated)
- **`Code/quasar-disney-mobile-docs/architecture/11-interface-contracts.md`** §8.3–§8.4 (one
  `ApiError`; the 401 collision), §11.3 (**the behavioral test table this substep implements**),
  §6.2 (two cursors)
- `Code/quasar-disney-mobile-docs/adr/ADR-0017-401-policy-above-the-transport.md`,
  `.../ADR-0020-rtk-query-base-api.md`, `.../ADR-0018-two-tier-ci-js-gate-before-bitrise.md`
- `Code/quasar-disney-mobile-docs/architecture/02-phasing-roadmap.md` §4 (criteria **F2**, **A3**,
  **A5**, **A6**) and §5 (DF1, DF2, DF12)
- `Code/quasar-disney-mobile-docs/architecture/04-data-model.md` §6 (fetch topology — note which
  parts belong to the shell, not here)
- `Upcoming Prompts/quasar-disney-mobile-STEP-3-PLAN.md` — the STEP test plan
- `Code/quasar-disney-mobile-app/README.md`

## Scope

**Owns:** the T2 integration tests under `src/api/`, and running the STEP's final gate.

**Does not touch:** production code, except to fix defects these tests uncover.

**Explicitly out of scope:** doc 12 §3.1's **boot gate** test (the loader holding until `/me` +
HomeFeed + CW all resolve) and the **persist whitelist** test. Both need the shell and the auth
slice, which are STEP-2/STEP-4/STEP-6. Say so in the substep summary rather than leaving the
apparent gap unexplained.

## Your task

### 1. Set up the T2 harness

Each test builds its own world (doc 12 §4.2 — never a shared singleton):

- `createStore()` from the scaffold, with `baseApi.reducer` and `baseApi.middleware` registered.
- `injectStore(store)` so the request interceptor can read the token.
- `createMockAdapter(axiosInstance, { latencyMs: 0, now: frozenClock, data: fixtures })` — latency
  zeroed so the suite stays inside doc 12 §7.1's **two-minute** budget, clock frozen so `exp` cases
  are deterministic.
- **Endpoints defined test-locally** via `baseApi.injectEndpoints(...)`. The real `login` / `getMe`
  endpoints belong to STEP-4 and the feed endpoints to STEP-5; defining them in the test file keeps
  feature ownership intact while still exercising the full store → middleware → baseQuery →
  interceptor → adapter path.
- Hooks, where a hook is the natural surface, via RNTL `renderHook`.

### 2. The must-cover T2 tests

**a. 401 scoping — the F2 guard** (doc 11 §8.4, doc 12 §3.1, ADR-0017)

| Scenario | Expected |
|---|---|
| `POST /auth/login` with wrong credentials → `INVALID_CREDENTIALS` | Session is **not** cleared; `resetApiState` is **not** dispatched; the error surfaces to the caller so the credentials screen can render its inline error |
| `GET /me` with an expired token → `UNAUTHORIZED` | Session **is** cleared (`sessionCleared` dispatched) **and** `resetApiState()` runs; any cached authenticated data is gone afterwards |
| Both, back to back in one store | The first leaves the store usable; the second tears it down. No path string is compared anywhere in the decision |

Assert on dispatched actions, not on internal calls — the point is the observable reaction.

**b. Two-axis `nextCursor` exhaustion** (doc 11 §6.2, doc 12 §3.2)

- **Vertical:** page `/home-feed` until the envelope's `nextCursor` is `null`; assert every
  container arrives exactly once and no page is fetched after exhaustion.
- **Horizontal:** page one container's `/containers/{id}/resources` until *its* `nextCursor` is
  `null`; same assertions on cards.
- **The client trusts `nextCursor` over arithmetic on counts.** Prove it: serve a final page whose
  length equals `limit` but whose `nextCursor` is `null`, and assert no further request is made.
  A count-based implementation passes every other test and fails this one — which is the whole
  reason doc 11 §6.2 states the rule.

**c. Expired mock JWT end to end** (doc 11 §9.2, doc 12 §3.1)

Mint a token, advance the frozen clock past its 7-day `exp`, call each of operations 2–5, and
assert every one returns `UNAUTHORIZED` through the full stack — and that the `/me` case triggers
the session clear from (a).

**d. Both transports normalize to one `ApiError`** (doc 11 §8.3, doc 12 §3.3)

Assert that what reaches the caller is always `ApiError { code, status, message }` and **never an
axios error or a raw rejection** — for a mocked `4xx`, a mocked `5xx`, and a transport-level failure
with no response at all. This is criterion **A3**'s testable half: it is what makes the Phase-3 swap
invisible above the boundary.

**e. Failure injection reaches the storefront error path** (doc 09 §6.2, doc 12 §3.3)

With an injected HomeFeed failure, the caller receives a normalized `ApiError` — a storefront error
state, **not** the no-internet overlay (doc 04 §6). Also confirm by inspection that nothing outside
a test can construct the injection.

### 3. `console.warn` stays observable

If any test touches the unknown-`variant` path, **spy on `console.warn` — never silence it**. Doc 11
§4.4 requires the warn to be observable, and a blanket console silence in test setup would quietly
make that assertion unfallible (doc 12 §5). If the scaffold's Jest setup silences console, fix the
setup and note it.

The unknown-variant *drop-and-warn* behavior itself is storefront composition (doc 12 §3.2 assigns
it T1 in STEP-5) — do not implement it here. Only ensure the observability seam survives.

### 4. Run the full gate

The four commands Tier A runs on every push (doc 12 §7.1, ADR-0018):

```
npx tsc --noEmit
npx jest --coverage
npx eslint .
npx prettier --check .
```

Confirm specifically that:

- **`import/no-restricted-paths`** passes — no feature imports another feature; `shared/` never
  imports `features/` (**A5 / DF5**).
- **The axios restriction** passes — `src/api/` is the only importer of `axios` (**DF1**).
- **The A2 rule** (color/spacing literals outside `src/shared/theme/`) passes. It is
  pattern-matching and will not catch a runtime-computed color — that gap is **RISK-0017**, not a
  defect to chase here.
- The suite finishes **under two minutes**. If it does not, the cause is almost certainly a
  non-zero adapter latency leaking into a test.

Coverage is **reported, not gated** in 1a (doc 12 §8.1, RISK-0016) — read the text summary, do not
add a threshold.

### 5. Close the STEP

- Confirm every row of the PLAN's test plan is satisfied, or records why it is not applicable.
- Confirm the doc obligations from 3.1 (doc 11 §2/§3 handover + Version Log), 3.2 (doc 03 §8.2
  `sessionCleared` seam), and 3.4 (doc 03 §8.1 `src/shared/assets/`) all actually landed — this is
  the doc-drift half of the STEP review.
- Update the PLAN's substep statuses.

## Verification

The gate above **is** this substep's verification. In addition:

- Run the suite twice in a row and confirm identical results — order-dependent failures are the
  classic symptom of a shared store or a module-level mutable adapter (doc 12 §4.2), and they
  surface here or in CI at the worst possible moment.
- Run with `--runInBand` once to confirm no test depends on parallel scheduling.

## Keeping the docs true  (always)

- If a T2 test uncovers a behavior that contradicts doc 11 §11.3's table, **the doc and the code
  must agree before this STEP closes** — fix whichever is wrong, and if it is the doc, bump its
  Version Log.
- If any must-cover row from doc 12 §3 ends up untested for a defensible reason, record it in
  `registries/risks.yml` with severity, owner, and revisit trigger rather than letting it disappear.
- Doc 12 §8.2 wants a durable Markdown summary in `reports/test-results/` only for runs that carry
  weight — the 2026-08-18 sign-off build, check-ins, incidents, security reviews. **A STEP-closing
  run is not one of them**; do not write one unless this run is also the sign-off run.

## Definition of done

- [ ] T2 integration tests cover: 401 scoping in both directions, two-axis `nextCursor` exhaustion
      including the trust-the-cursor case, expired JWT across operations 2–5, single-`ApiError`
      normalization across mocked `4xx`/`5xx`/no-response, and injected failure reaching the error
      path.
- [ ] Every test builds its own `createStore()` and its own adapter; the suite is order-independent
      and passes with `--runInBand`.
- [ ] `console.warn` is spied, never silenced, anywhere in the test setup.
- [ ] `tsc --noEmit` · `jest` · `eslint` · `prettier --check` are **all green** on
      `step-0003-contract-mocks`, in under two minutes.
- [ ] The out-of-scope items (boot gate, persist whitelist) are named in the summary with their
      owning STEPs, not left as a silent gap.
- [ ] The doc obligations from 3.1, 3.2, and 3.4 are confirmed landed, with Version Logs bumped.
- [ ] Any accepted risk or deferred debt is in `registries/risks.yml`, or explicitly not applicable.

## Next

This is the last substep. Update the PLAN, then tell the user the STEP's next action: the **STEP
review** — the team's standard PR / code review plus the doc-drift check — after which STEP-3 is
archived to `prompts/001-poc/step-0003/` and marked `Done` in `prompts/STEP-index.md`. Then the
next-action resolver (`METHOD.md` §10) picks up STEP-4 or STEP-5, both of which unblock on this
STEP merging.
