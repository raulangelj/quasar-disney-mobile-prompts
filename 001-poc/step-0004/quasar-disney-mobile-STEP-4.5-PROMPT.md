# quasar-disney-mobile — STEP-4.5: Session restore, /me 401, shell wiring

> **How to run:** Tell your agent *"run substep 4.5"*. Self-contained — executable cold in a fresh chat.

## Context

Auth slice, endpoints, and all three auth screens exist (4.1–4.4). This substep connects the feature to the shell: real `selectIsAuthenticated`, boot-time session validation via `getMe`, LoadingGate during getMe-only restore, and T2 tests for restore + 401. Last substep — run STEP review after the gate.

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-4-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-4-PLAN.md`
- `Code/quasar-disney-mobile-docs/architecture/03-architecture-overview.md` §6 Flow 1 (steps 3–7; **partial** — no HomeFeed/CW gate yet)
- `Code/quasar-disney-mobile-docs/architecture/16-identity-auth.md` §4 (401 reaction in baseQueryWithAuth)
- `Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md` §3 (T2 restore, getMe 401)
- `Code/quasar-disney-mobile-app/src/app/AppShell.tsx`, `RootNavigator.tsx`, `LoadingGate.tsx`
- `Code/quasar-disney-mobile-app/src/api/client/` (baseQueryWithAuth, sessionCleared)
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`

## Scope

**Owns:** Shell wiring for auth session lifecycle; getMe-only LoadingGate; T2 restore/401 tests; manual F1 auth smoke; STEP-4 review prep.

**Does not:** HomeFeed/CW boot queries (STEP-6); real storefront (STEP-5); T3 UI tests.

## Your task

1. On `step-0004-auth-feature`.
2. **`AppShell` / boot sequence** (after PersistGate rehydrates):
   - If no token → show Auth navigator (existing).
   - If token present → set `LoadingGate` **visible**, trigger `getMe` (lazy query or `useGetMeQuery` with skip flip).
   - `getMe` success → hide gate, `RootNavigator` shows App tabs (placeholder home OK).
   - `getMe` `UNAUTHORIZED` → `baseQueryWithAuth` clears session + reset API cache → hide gate → Welcome.
3. **Post-login path:** after 4.4 login success, ensure same getMe + gate path runs before showing app tabs (avoid flash of Welcome).
4. **`RootNavigator`:** confirm it reads real `selectIsAuthenticated` (remove any stub comments).
5. **Do not** wait on `getHomeFeed` / `getContinueWatching` — document in code comment that STEP-6 extends the gate.
6. **T2 tests:**
   - Rehydrate store with valid session blob → `getMe` succeeds → authenticated selector true.
   - Rehydrate with token mock rejects (expired or invalid) → session cleared → unauthenticated.
7. **Manual smoke (F1 auth leg):** iOS simulator — Welcome → email → demo password → lands on app tabs; kill app, relaunch → still authenticated (getMe restore); wrong password → inline error stays on password screen.
8. Run full **STEP gate:** `npx tsc --noEmit && CI=true npm test && npx eslint . && npx prettier --check .`

## Verification

- Full Tier A gate green.
- T2 restore + 401 tests pass.
- Manual smoke checklist in PR description.
- Import/architecture lint rules still pass.

## Keeping the docs true  (always)

Do not update doc 03 to claim three-query gate is live. Optional README note: auth flow complete; cold-start feed gate is STEP-6.

## Definition of done

- [ ] Session restore and login success both validate via `getMe`.
- [ ] `/me` 401 returns user to Welcome with cleared session.
- [ ] LoadingGate shows during getMe-only boot (not permanent).
- [ ] T2 + full STEP gate green.
- [ ] Manual auth smoke passed.
- [ ] Ready for STEP-4 REVIEW + PR.

## Next

STEP-4 review — open PR on `step-0004-auth-feature`, cross-read PLAN/REVIEW, archive to `prompts/001-poc/step-0004/` when merged.
