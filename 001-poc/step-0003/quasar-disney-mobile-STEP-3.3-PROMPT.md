# quasar-disney-mobile — STEP-3.3: Mock adapter — seams, mock JWT, five handlers

> **How to run:** Tell your agent *"run substep 3.3"*. Self-contained — runnable cold.

## Context

STEP-3.2 built the axios instance and the RTK Query layer above it. This substep gives that
instance something to talk to.

One framing matters more than any implementation detail here:

> **The mock adapter is a production artifact, not a test double.** Doc 11 §6.5 makes it one of
> two transport implementations of a single contract, and doc 09 §6.2 makes its deterministic
> failure path a *demo* requirement (criterion F2). The thing you would normally mock is, in this
> codebase, one of the primary systems under test.

That is why it runs on the **same** axios instance the app uses (ADR-0020) rather than beside it:
the interceptors execute in Phase 1a instead of first running against a real backend in Phase 3.
And it is why the handlers **validate the Bearer token's presence and `exp`** rather than accepting
anything — a permissive mock would leave the 7-day expiry path and the `/me` 401 → Welcome flow
untested until Phase 3, which is the exact branch that would then fail on first contact with a real
server (doc 11 §9.2).

## Read these first

- root `.throughstone/local-user.md`
- **`Code/quasar-disney-mobile-docs/architecture/11-interface-contracts.md`** — §5 (the five
  operations), §6.1–§6.3 (envelope, two cursors, `limit` defaults), §6.5 (**transport profiles —
  what the mock owes**), §7.4 (operation payloads), §8.2 (`code` → status table), §9.2 (**the mock
  handlers validate the token**), §11.3 (the behavioral test list)
- **`Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md`** — §3.1 (credential compare;
  `exp` validation), §3.3 (`ApiError`; failure injection is test-only; **the latency-default
  test**), §4.2 (no module-level mutable state), §4.3 (`@env` resolves to a committed stub in
  tests), §4.4 (**latency, clock, and failure injection are constructor parameters**), §5,
  §11 item 5
- **`Code/quasar-disney-mobile-docs/architecture/09-environments.md`** §4.2 (the three env keys),
  §6.2 (**"injection is a test seam, not a runtime toggle — there is no debug menu, shake gesture,
  or hidden control in the release binary"**)
- `Code/quasar-disney-mobile-docs/architecture/16-identity-auth.md` §1 (**the auth decision is the
  mock adapter, not a screen `if`**), §3, §6 (mock JWT claims; 7-day `exp`; remint on next login)
- `Code/quasar-disney-mobile-docs/architecture/04-data-model.md` §1.4 (HomeFeed is hero + 15; CW is
  typically one `progress` container), §6 (fetch topology)
- `Code/quasar-disney-mobile-docs/architecture/05-scaling-performance.md` — the **400–600 ms**
  artificial latency band
- `Code/quasar-disney-mobile-docs/adr/ADR-0020-rtk-query-base-api.md` point 4,
  `.../ADR-0006-two-endpoint-home-composition.md`, `.../ADR-0007-container-card-shared-types.md`
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`, `.../api.md`
- `Code/quasar-disney-mobile-app/README.md`

## Scope

**Owns:** `src/api/mocks/createMockAdapter.ts`, `jwt.ts`, `cursor.ts`, `handlers/`.

**Depends on:** 3.1's types and 3.2's axios instance.

**Does not touch:** the demo fixture *content* (3.4 authors it — this substep takes the data it
serves as a parameter), any feature, any UI, the store.

## Your task

### 1. `createMockAdapter(instance, options)` — three constructor seams

```ts
createMockAdapter(axiosInstance, {
  latencyMs?: number | (() => number),   // default: a value drawn from 400–600 ms
  now?: () => number,                    // default: Date.now
  failures?: FailureInjection,           // default: none
  data?: MockDataset,                    // fixtures to serve — 3.4 supplies the demo set
})
```

Returns the `axios-mock-adapter` instance so a caller can restore it.

**All three seams are constructor parameters, and nothing else** (doc 12 §4.4):

| Seam | Default | In tests |
|---|---|---|
| Latency | doc 05's **400–600 ms** | `0` |
| Clock (`now()`) | `Date.now` | frozen |
| Failure injection | off | per test |

Why each is injected rather than toggled, worth a docstring: a suite paying 400–600 ms per call is
minutes of dead wall-clock per run, and a slow suite is one people stop running; constructor
injection beats fake timers, which get awkward around promise scheduling; and an `exp` test written
against `Date.now()` passes today and fails later for no reason.

**Hard boundary (doc 09 §6.2):** *deterministic* failure — wrong credentials rejected — ships in
`release`, always, because F2 requires the inline error on stage. *Injectable* failure — forcing
HomeFeed or Continue Watching to fail — is a **test seam only**. No runtime toggle, no debug menu,
no shake gesture, nothing reachable from the app entry. A test must be the only thing that can
construct it.

**No module-level mutable state** (doc 12 §4.2). Cursors and injected-failure flags live on the
instance the factory returns, so one test cannot leak state into the next.

### 2. `src/api/mocks/jwt.ts`

Mint and validate the mock JWT (doc 16 §6):

- Claims: **`sub`** (the user `id`, a UUID), **`exp`** (`now() + 7 days`), **`iat`**. Nothing else —
  **no `userName`, no email, no roles.**
- Encoding: base64url header/payload with a fixed fake signature segment. **Do not add a JWT
  library** — doc 11 §6.4's whole point is that no `jwt-decode` dependency enters doc 03 §7's
  hard-dependency list. This is the mock's own token, discarded with the mock in Phase 3.
- `mintToken(userId, now)` → `{ accessToken, expiresAt }` where **`expiresAt` is an ISO 8601 UTC
  string** (doc 11 §6.4) even though the claim inside the token is numeric. That asymmetry is
  deliberate; put the reason in the docstring.
- `validateBearer(header, now)` → the decoded claims, or a reason for rejection. Reject a missing
  header, a malformed token, and an **expired** one. Take `now` from the injected clock, never
  `Date.now` directly.
- Successful login **remints** — no refresh token exists and none should be added "for later"
  (doc 16 §6).

### 3. `src/api/mocks/cursor.ts`

Opaque cursor helpers used by both paging axes:

- Encode an offset into an opaque string (base64 of an index is fine — the point is that it is
  **opaque**, and the client never parses, compares, or constructs one).
- Decode defensively: an unrecognized cursor is a client bug, not a crash — reject it as
  `VALIDATION_FAILED` rather than throwing.
- A `paginate(items, cursor, limit)` helper returning `{ page, nextCursor }` where **`nextCursor`
  is `null` exactly when the list is exhausted** (doc 11 §6.2). This helper is what makes "the
  client trusts `nextCursor` over arithmetic on counts" true on the server side too.

### 4. `src/api/mocks/handlers/` — the five operations

Every response uses the wire shapes from doc 11 §7.4, and every error uses
`{ error: { code, message } }` with a **synthetic HTTP status** on the mocked response (doc 11
§6.5, §8.2).

**1 — `POST /auth/login`** (unauthenticated)
- Compare against `DEMO_EMAIL` / `DEMO_PASSWORD` read from **`@env`** — not a committed fixture and
  **not a screen-level `if`** (doc 16 §1, doc 09 §4.2). In tests `@env` resolves to the committed
  stub (doc 12 §4.3).
- Match → `200 { accessToken, expiresAt }`, freshly minted with a 7-day `exp`.
- No match → `401 INVALID_CREDENTIALS`. **"No such email" and "wrong password" return the same
  code** — no user enumeration (doc 11 §8.5). This is moot with one demo account (RISK-0008), but
  this project is a migration template and a split error here is exactly what gets copied forward.

**2 — `GET /me`** (Bearer)
- Validate presence **and** `exp` → `200 { id, userName }`. No email, no roles, not even a stub.
- Invalid or expired → `401 UNAUTHORIZED`.

**3 — `GET /home-feed?cursor=&limit=`** (Bearer)
- `limit` defaults to **16**; treat it as a hint the server may cap.
- `{ data: Container[], nextCursor }`. First page: one `variant: 'hero'` container plus 15 others.
  **Further vertical pages contain no second hero.**
- **A `progress` container is never a member of a `/home-feed` response** (ADR-0006) — assert this
  in the handler, not just in the fixtures.

**4 — `GET /continue-watching?cursor=&limit=`** (Bearer)
- `limit` defaults to **10**. `{ data: Container[], nextCursor }`, typically one container with
  `variant: 'progress'`.

**5 — `GET /containers/{containerId}/resources?cursor=&limit=`** (Bearer)
- `limit` defaults to **10**. `{ data: Card[], nextCursor }` — the **horizontal** axis.
- Unknown `containerId` → `404 NOT_FOUND`.

Every one of operations 2–5 rejects a missing, malformed, or expired Bearer with `UNAUTHORIZED`
before doing anything else (doc 11 §9.2). The request interceptor from 3.2 has already attached the
header; the handler's job is not to trust it.

**Never in an error payload:** stack traces, internal paths, SQL, or anything echoing the submitted
password (doc 11 §8.6).

## Verification

**T1 unit tests**, colocated `*.test.ts`. Construct the adapter per test — never share one.

| Test file | Must cover |
|---|---|
| `handlers/login.test.ts` | Credential compare from `@env`: success mints a token with a 7-day `exp` and an ISO `expiresAt`; wrong password → `401 INVALID_CREDENTIALS`; **unknown email returns the identical code**; the password never appears in the response or any log |
| `handlers/me.test.ts` | Valid Bearer → `{ id, userName }`; **missing** header → `UNAUTHORIZED`; **malformed** token → `UNAUTHORIZED`; **expired** token (frozen clock advanced past `exp`) → `UNAUTHORIZED` |
| `handlers/homeFeed.test.ts` | Default `limit` is 16; first page is one `hero` + 15; later pages contain **no** second hero; **no `progress` container ever appears**; `nextCursor` is `null` exactly at exhaustion; an unauthenticated call → `UNAUTHORIZED` |
| `handlers/continueWatching.test.ts` | Default `limit` is 10; the container's `variant` is `progress`; Bearer enforced |
| `handlers/containerResources.test.ts` | Default `limit` is 10; **horizontal** cursor exhaustion; unknown `containerId` → `404 NOT_FOUND`; Bearer enforced |
| `cursor.test.ts` | Round-trip encode/decode; `nextCursor` is `null` **only** at exhaustion; an unrecognized cursor is rejected as `VALIDATION_FAILED` and does not throw |
| `jwt.test.ts` | Claims are exactly `sub`/`exp`/`iat`; **`userName`, email, and roles are absent**; `exp` is 7 days out from the injected clock; `expiresAt` is a valid ISO 8601 UTC string; validation uses the injected clock, not `Date.now` |
| `createMockAdapter.test.ts` | **The default latency is nonzero and inside 400–600 ms** (doc 12 §3.3 — the likeliest way DF2 quietly dies is someone zeroing the default after seeing it zeroed in tests); `latencyMs: 0` is honoured; the injected clock is honoured; **failure injection is reachable only through the constructor** — grep the source to prove no runtime path constructs it; two adapters built in the same file share no cursor or failure state |

Beyond the happy path, cover: empty result sets, a `limit` of 0 or larger than the dataset, a
cursor pointing past the end, and an injected failure on each of HomeFeed and Continue Watching.

Run before marking done:

```
npx tsc --noEmit
npx jest src/api/mocks
```

## Keeping the docs true  (always)

- If a handler cannot serve exactly what doc 11 §7.4 specifies, **the doc is what changes** — fix
  its table, bump the Version Log, and update the types, mocks, and tests in the same PR (§12).
  Do not let the mock quietly define a different wire shape than the contract the backend team is
  handed (RISK-0004).
- Doc 11 §9.2 is already written as a contract obligation; if this substep discovers it needs
  wording it does not have, amend it rather than working around it.
- Record any deferred behavior in `registries/risks.yml` with severity, owner, and revisit trigger.
- `.env` stays gitignored; only `.env.example` is committed. The demo credentials are fake, but
  they still do not get hardcoded into a fixture or a screen.

## Definition of done

- [ ] `createMockAdapter` runs `axios-mock-adapter` on the **same** axios instance from 3.2, with
      latency, clock, and failure injection as **constructor parameters** and no runtime toggle.
- [ ] The default latency is nonzero and inside 400–600 ms, and a test pins it.
- [ ] Mock JWTs carry `sub`/`exp`/`iat` only, with a 7-day TTL, minted without a JWT library; the
      login response's `expiresAt` is an ISO 8601 UTC string.
- [ ] All five operations are served with the `{ data, nextCursor }` envelope and doc 11 §7.4's
      payloads; operations 2–5 reject a missing, malformed, or expired Bearer with `UNAUTHORIZED`.
- [ ] Errors use `{ error: { code, message } }` with a synthetic status, and never echo the
      password or leak internals.
- [ ] No module-level mutable state; adapters are constructed per test.
- [ ] `npx jest src/api/mocks` and `npx tsc --noEmit` are green; no `any`; docstrings throughout.
- [ ] Any accepted risk or deferred debt is in `registries/risks.yml`, or explicitly not applicable.

## Next

Update this substep's status in the STEP PLAN, then tell the user: **"run substep 3.4"**, in a fresh
chat.
