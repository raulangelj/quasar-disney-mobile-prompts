# quasar-disney-mobile — STEP-3.2: axios client, interceptors, `baseQueryWithAuth`, `baseApi`

> **How to run:** Tell your agent *"run substep 3.2"*. Self-contained — runnable cold.

## Context

STEP-3.1 landed the wire types. This substep builds the transport layer underneath them: one axios
instance, its two interceptors, the RTK Query base query that wraps it, and the `baseApi` that
STEP-4 and STEP-5 will `injectEndpoints` onto.

The load-bearing decision here is **where the 401 reaction lives**, and it is worth understanding
before writing a line:

> A login failure and an expired token are both `401`. The "clear the session and return to
> Welcome" reaction must fire only on the second. If it fires on the first, a wrong password
> bounces the user out of the credentials screen and **criterion F2 — the inline error state, half
> of what the 2026-08-18 demo exists to show — never renders.**

The common React Native idiom puts a global 401 handler in the axios interceptor. **This project
deliberately does not** (ADR-0017). The request interceptor attaches the header; the response
interceptor maps status → `ApiError`; **`baseQueryWithAuth` reacts**, switching on `code` and never
on a path. A developer arriving from another codebase will look in the interceptor and not find it,
so the interceptor carries a comment saying where the policy lives and why.

## Read these first

- root `.throughstone/local-user.md`
- **`Code/quasar-disney-mobile-docs/adr/ADR-0020-rtk-query-base-api.md`** — the whole ADR; points
  1, 2, 3, and 6 are this substep
- **`Code/quasar-disney-mobile-docs/adr/ADR-0017-401-policy-above-the-transport.md`** — including
  the 2026-08-17 amendment that names `baseQueryWithAuth` as the home
- `Code/quasar-disney-mobile-docs/architecture/11-interface-contracts.md` §5 (paths), §6.3
  (content type, casing), §6.5 (transport profiles), §8.2–§8.4 (error model, one `ApiError`, the
  401 collision), §9.1 (Bearer header; which operations are authenticated), §10 (**what the
  boundary may and may not log**)
- `Code/quasar-disney-mobile-docs/architecture/03-architecture-overview.md` §8.1–§8.2
- `Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md` §2 (T1 vs T2), §3.1 (the
  401-scoping row), §3.3 (`ApiError` normalization), §5 (what is mocked), §7.1 (the merge gate)
- `Code/quasar-disney-mobile-docs/architecture/09-environments.md` §4.2 (`API_BASE_URL` is inert in
  Phase 1 — carried so Phase 3 is a value change, not a code change), §4.3
- `Code/quasar-disney-mobile-docs/architecture/16-identity-auth.md` §6 (Bearer header, mock JWT)
- `Code/quasar-disney-mobile-docs/architecture/10-observability.md` §2.3 (the never-log list)
- `Code/quasar-disney-mobile-docs/adr/ADR-0015-config-via-babel-transform.md`
- `Code/quasar-disney-mobile-docs/adr/ADR-0003-persist-auth-secure-storage.md` (why the cache is
  never persisted, and what `resetApiState()` is protecting)
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`, `.../api.md`
- `Code/quasar-disney-mobile-app/README.md`

## Scope

**Owns:** `src/api/client/`, `src/api/sessionCleared.ts`, `src/api/baseApi.ts`.

**Does not touch:** mocks, fixtures, any feature `api.ts`, the auth slice, any UI. `injectEndpoints`
calls belong to STEP-4 and STEP-5 — `baseApi` ships here with **empty** endpoints.

## Your task

### 1. `src/api/client/paths.ts`

The five operation paths as constants (doc 11 §5):

```
POST /auth/login
GET  /me
GET  /home-feed
GET  /continue-watching
GET  /containers/{containerId}/resources
```

**No `/v1/` segment** (doc 11 §4.1) — if Phase 3's host wants a prefix it goes in the
`API_BASE_URL` *value*, not in these constants. Note in a docstring that operation 5 is
deliberately **flat** rather than nested under a feed, so a row is addressable regardless of which
feed it arrived in.

### 2. `src/api/client/axiosInstance.ts`

One `axios.create({ baseURL: API_BASE_URL, headers: { 'Content-Type': 'application/json; charset=utf-8' } })`,
reading `API_BASE_URL` from `@env` (ADR-0015). Document that the value is **inert in Phase 1** —
there is no host — and that this is the single field the transport profile flips at Phase 3
(doc 11 §1.2, §6.5).

This is the only axios instance in the codebase, and `src/api/` is the only directory permitted to
import axios (DF1, enforced by ESLint).

### 3. `src/api/client/injectStore.ts`

The request interceptor needs the current access token, and the store is constructed by the shell.
Export `injectStore(store)` and an internal accessor (ADR-0020 point 2: *"the shell calls
`injectStore(store)` after `configureStore`"*). A module-level store *reference* set once by the
composition root is the point of this seam — do not reach into a store singleton, which doc 12 §4.2
forbids.

Guard the un-injected case: reading the token before `injectStore` has run returns `undefined`
rather than throwing, so an early call degrades to an unauthenticated request that the mock
handlers reject cleanly with `UNAUTHORIZED`.

### 4. Request interceptor — attach `Authorization`

Attach `Authorization: Bearer <jwt>` when a token exists, on **operations 2–5 only**. Operation 1
(`POST /auth/login`) is unauthenticated (doc 11 §9.1) and must not carry the header.

A path check is correct *here* — this is about which requests get a header, which is genuinely a
property of the operation. It is the **session-clearing reaction** that must never be a path
comparison (ADR-0017). Say that in a comment so the two are not conflated later.

**Never log the `Authorization` value** (doc 10 §2.3).

### 5. Response interceptor — normalize to one `ApiError`

Map every failure into `ApiError { code, status, message }` (doc 11 §8.3) so that **features and
hooks never see an axios error or a raw rejection** — that is precisely what makes the Phase-3 swap
invisible above the boundary.

- Prefer the body's `error.code` when the response carries the `{ error: { code, message } }`
  shape.
- Fall back by status when it does not: `401 → UNAUTHORIZED`, `400 → VALIDATION_FAILED`,
  `404 → NOT_FOUND`, anything else (including a network error with no response) `→ INTERNAL`.
- Synthesize a `status` when there is no response at all, so the shape is total.

**This interceptor does not clear the session.** Add the comment ADR-0017 asks for: the
session-clearing 401 policy lives in `baseQueryWithAuth`, because a global handler here would also
fire on login failure and destroy F2.

**Boundary logging:** the API module may log operation name, `status`, `code`, and duration. Never
request or response **bodies**, never the `Authorization` value (doc 10 §2.3, doc 11 §10). `console`
is the sanctioned mechanism (doc 12 §10.1) — there is no logging library in this project.

### 6. `src/api/sessionCleared.ts` — the seam (PLAN Q1)

`baseQueryWithAuth` must clear the session, but the auth slice is STEP-4's and the API module must
not import a feature (doc 03 §8.2).

Resolve it with an action the **API module owns**:

```ts
export const sessionCleared = createAction('session/cleared')
```

STEP-4's auth slice reduces it via `extraReducers`. The dependency direction stays feature → api,
and this substep can be built and tested with no auth slice in existence.

**This is a new seam no architecture doc records.** Add it to doc 03 §8.2's import-rule table and
bump that doc's Version Log. If you judge it contested, write an ADR instead — but do not leave it
undocumented.

### 7. `src/api/client/axiosBaseQuery.ts`

An RTK Query `BaseQueryFn<{ url, method, data?, params? }, unknown, ApiError>` over the instance
above. On success return `{ data }`; on failure return `{ error }` where `error` is the normalized
`ApiError`.

### 8. `src/api/client/baseQueryWithAuth.ts` — the 401 policy

Wrap `axiosBaseQuery`. **Switch on `code`, never on a path:**

| `code` | Reaction |
|---|---|
| `UNAUTHORIZED` | `dispatch(sessionCleared())` **and** `dispatch(baseApi.util.resetApiState())`, then return the error |
| `INVALID_CREDENTIALS` | **Neither.** Return the error so the credentials screen can render F2's inline error |
| anything else | Return the error unchanged |

`resetApiState()` matters because the RTK Query cache holds `/me` and the catalog; leaving it
populated after a session ends would let stale authenticated content survive a logout (ADR-0003,
ADR-0020 point 5).

**Circular-import note:** `baseApi` is built *with* `baseQueryWithAuth`, and `baseQueryWithAuth`
needs `baseApi.util.resetApiState()`. Break the cycle by resolving `baseApi` lazily inside the
handler (a deferred `require`/dynamic import, or dispatching the reset action by its type string) —
whichever your bundler and `tsc` both accept cleanly. Leave a comment explaining why the import is
deferred, so nobody "tidies" it back into a top-level import.

### 9. `src/api/baseApi.ts`

```ts
createApi({ reducerPath: 'api', baseQuery: baseQueryWithAuth, endpoints: () => ({}) })
```

Empty endpoints by design — auth injects `login` and `getMe` (STEP-4), storefront injects
`getHomeFeed`, `getContinueWatching`, and `getContainerResources` (STEP-5). Document that the shell
registers `baseApi.reducer` and `baseApi.middleware` and calls `injectStore(store)`, and that the
`api` reducer path is **excluded from the persist whitelist** (ADR-0003).

## Verification

Write **T1 unit tests** colocated as `*.test.ts` (doc 12 §10.1). These run without a store — the
store-level behavior is 3.5's T2 layer.

| Test file | Must cover |
|---|---|
| `src/api/client/requestInterceptor.test.ts` | Header attached on operations 2–5 when a token exists; **not** attached on `POST /auth/login`; nothing attached when no token is injected; the `Authorization` value never reaches a log |
| `src/api/client/responseInterceptor.test.ts` | Each status maps to the right `ErrorCode`; a body-supplied `error.code` wins over the status fallback; a network error with no response becomes `INTERNAL` with a synthesized `status`; the returned object is always a complete `ApiError { code, status, message }`; **the interceptor never dispatches anything** |
| `src/api/client/baseQueryWithAuth.test.ts` | `UNAUTHORIZED` dispatches `sessionCleared` **and** `resetApiState`; `INVALID_CREDENTIALS` dispatches **neither** (this is the F2 guard); `NOT_FOUND` / `INTERNAL` pass through untouched; the decision is made on `code` with no path inspected |

Cases to include beyond the happy path: missing token, malformed error body, a 401 with no body at
all, and a success response (to prove nothing fires on the good path).

Run before marking done:

```
npx tsc --noEmit
npx jest src/api/client
```

## Keeping the docs true  (always)

- **Required:** doc 03 §8.2 gains the `sessionCleared` seam (task 6) and its Version Log is bumped —
  or an ADR records it.
- If the interceptor's status → `code` fallback table differs in any way from doc 11 §8.2, that is
  a contract change: update doc 11's table, bump its Version Log, and update the types, mocks, and
  tests in the same PR (doc 11 §12).
- If this substep defers anything (an untested branch, a known-imperfect normalization), record it
  in `registries/risks.yml` with severity, owner, and revisit trigger.
- No secrets in the repo; `.env` stays gitignored and `.env.example` documents the keys.

## Definition of done

- [ ] One axios instance in `src/api/client/`, reading `API_BASE_URL` from `@env`; no other
      directory imports axios.
- [ ] Request interceptor attaches `Authorization: Bearer <jwt>` on operations 2–5 and never on
      `/auth/login`.
- [ ] Response interceptor normalizes every failure into `ApiError { code, status, message }` and
      **clears nothing**, carrying the comment that points at `baseQueryWithAuth`.
- [ ] `baseQueryWithAuth` clears the session **and** resets the API cache on `UNAUTHORIZED`, does
      neither on `INVALID_CREDENTIALS`, and switches on `code` with no path comparison anywhere.
- [ ] `src/api/baseApi.ts` exports a `createApi` with `reducerPath: 'api'` and empty endpoints.
- [ ] `sessionCleared` exists in the API module and is recorded in doc 03 §8.2 (Version Log bumped)
      or in an ADR.
- [ ] The three T1 test files above are green: `npx jest src/api/client`.
- [ ] `npx tsc --noEmit` green; no `any`; every function and type carries a docstring.
- [ ] Any accepted risk or deferred debt is in `registries/risks.yml`, or explicitly not applicable.

## Next

Update this substep's status in the STEP PLAN, then tell the user: **"run substep 3.3"**, in a fresh
chat.
