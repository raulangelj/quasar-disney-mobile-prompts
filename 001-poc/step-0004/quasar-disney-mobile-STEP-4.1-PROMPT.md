# quasar-disney-mobile — STEP-4.1: Auth slice + login / getMe endpoints

> **How to run:** Tell your agent *"run substep 4.1"*. Self-contained — executable cold in a fresh chat.

## Context

STEP-3 merged the contract layer: wire types, axios client, `baseApi`, mock adapter with credential compare and JWT validation. STEP-2 left a **stub** auth slice (`isAuthenticated: boolean`). This substep replaces it with the real persisted session model and injects the two auth endpoints auth screens will call in 4.3–4.5.

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-4-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-4-PLAN.md`
- `Code/quasar-disney-mobile-docs/architecture/03-architecture-overview.md` §6 Flow 1, §8.1 (`features/auth/api.ts`)
- `Code/quasar-disney-mobile-docs/architecture/04-data-model.md` §1.5 (Session shape in slice)
- `Code/quasar-disney-mobile-docs/architecture/11-interface-contracts.md` §7.4 (operations 1–2), §8.4 (401 scoping)
- `Code/quasar-disney-mobile-docs/architecture/16-identity-auth.md` §3–§6
- `Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md` §3 (T1 auth tests)
- `Code/quasar-disney-mobile-app/src/api/baseApi.ts`, `src/api/types/auth.ts`, `src/app/store/createStore.ts`, `src/app/store/persistConfig.ts`
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`
- `Code/quasar-disney-mobile-app/README.md`

## Scope

**Owns:** `src/features/auth/state/authSlice.ts` (replace stub), `src/features/auth/api.ts` (new), auth-related exports from `src/features/auth/` if needed, T1 tests, any store wiring to register injected endpoints.

**Does not:** UI screens or molecules (4.2+), navigator changes beyond what tests need (4.5), storefront endpoints, LoadingGate visibility logic (4.5), mock handler changes (STEP-3).

## Your task

1. On branch `step-0004-auth-feature` from `main` (STEP-3 must be merged).
2. **Auth slice** (doc 04 §1.5, doc 16 §3):
   - State: `{ accessToken: string | null; expiresAt: number | null }` (Unix seconds in slice).
   - Reducers/actions: `setSession` (from login fulfilled payload — parse ISO `expiresAt` from wire to Unix for storage), `clearSession` (logout / 401 path).
   - Remove stub `isAuthenticated` / `setAuthenticated`.
   - `selectIsAuthenticated`: true when `accessToken` is non-null (mock validates `exp` server-side on authenticated ops).
   - `selectAccessToken` for the request interceptor (may already read from store via injectStore).
3. **`features/auth/api.ts`** — `authApi = baseApi.injectEndpoints({...})`:
   - `login` mutation — `POST /auth/login`, body `LoginRequest`, returns `LoginResponse`.
   - `getMe` query — `GET /me`, provides `'User'` tag or similar; cache holds `{ id, userName }`.
   - Export hooks: `useLoginMutation`, `useGetMeQuery`, `useLazyGetMeQuery` (if needed for boot).
   - On `login.matchFulfilled`, dispatch `setSession` with token + parsed expiry.
   - Do **not** handle session clearing on login failure here — that's `baseQueryWithAuth` for `/me` only (doc 11 §8.4).
4. **Logout helper** (exported from auth feature): dispatches `clearSession` + `baseApi.util.resetApiState()` — used by 4.5 and shell later.
5. Ensure `createStore()` still registers `authReducer` under key `auth` and persist whitelist remains **exactly** `['auth']`.
6. Wire mock adapter in dev/test the same way STEP-3 established (if not already at app entry).
7. **T1 tests** (`src/features/auth/state/authSlice.test.ts` or colocated):
   - Login fulfilled writes token + `expiresAt`.
   - `clearSession` nulls both fields.
   - `selectIsAuthenticated` reflects presence of token.
   - `createStore.test.ts` still passes — persist whitelist unchanged.

## Verification

- Run **now:** `npx tsc --noEmit && CI=true npm test -- authSlice createStore && npx eslint src/features/auth src/app/store && npx prettier --check src/features/auth src/app/store`
- No T2 yet — adapter integration tests land in 4.4/4.5.
- Import rules: no storefront imports; no direct axios in feature code.

## Keeping the docs true  (always)

If the auth slice field names differ from doc 04, update doc 04 Version Log or add a clarifying note — prefer matching the doc. Do not persist RTK Query cache.

## Definition of done

- [ ] Stub auth slice removed; real session model in place.
- [ ] `login` and `getMe` endpoints injected and exported as hooks.
- [ ] T1 tests green for reducer/selectors/persist whitelist.
- [ ] TSDoc on new public functions.
- [ ] Tier A commands pass for touched paths.

## Next

`run substep 4.2` — auth molecules + i18n (can parallel with 4.3 prep once 4.1 merges locally).
