# quasar-disney-mobile — STEP-4.4: PasswordEntry + F2 inline error

> **How to run:** Tell your agent *"run substep 4.4"*. Self-contained — executable cold in a fresh chat.

## Context

Welcome and EmailEntry work (4.3). Auth endpoints and slice work (4.1). This substep completes the credentials UI, wires the `login` mutation, and satisfies criterion **F2**: wrong password shows an **inline** error on the password screen without clearing the session or popping the stack.

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-4-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-4-PLAN.md`
- `Code/quasar-disney-mobile-docs/architecture/07-ui-design-system.md` §4 (back during submit, editar link), §3.4 (error + submit spinner)
- `Code/quasar-disney-mobile-docs/architecture/11-interface-contracts.md` §8.4 (INVALID_CREDENTIALS vs UNAUTHORIZED)
- `Code/quasar-disney-mobile-docs/inputs/ui/disney-plus-reference-screens.md` §3–§4
- `Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md` §3 (T2 login / 401 scoping)
- `Code/quasar-disney-mobile-app/src/features/auth/api.ts`, `src/test/env.stub.ts` (demo creds for tests)
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`

## Scope

**Owns:** `PasswordEntryScreen`; replace placeholder; login submit UX; T2 tests for login success and INVALID_CREDENTIALS scoping.

**Does not:** Boot restore / getMe 401 (4.5), navigator switch persistence (4.5), three-query LoadingGate, storefront.

## Your task

1. On `step-0004-auth-feature`.
2. **`PasswordEntryScreen`**:
   - Read `email` from route params; show in body with **(editar)** `Link` → navigate back to `EmailEntry` with email prefilled if practical.
   - `PasswordField` + case-sensitivity hint + primary CTA `Iniciar sesión`.
   - Inert links: *Más información…*, *¿Problemas para iniciar sesión?* (no ops — PLAN closed).
   - **Submit:** `useLoginMutation` with `{ email, password }` from `@env` demo values in dev.
   - **Loading:** spinner in button, disable double submit.
   - **Success:** do not navigate manually here if 4.5 owns shell reaction — either dispatch session only and let 4.5 trigger `getMe` + nav, **or** call `getMe` and rely on `selectIsAuthenticated` (coordinate with PLAN: login fulfilled already sets slice in 4.1; RootNavigator switches when selector true — may need lazy getMe in 4.5). Minimum for 4.4: after login success, session in store; navigation switch can be verified in 4.5.
   - **F2 error:** on `INVALID_CREDENTIALS`, show multi-line red error under field (reference §4), red bottom border on field, password visible state optional per reference; **stay on PasswordEntry** — no Alert, no pop.
   - **Back:** clears password field, keeps email; **block back** while mutation pending (doc 07 §4).
3. **T2 tests** (`src/features/auth/api.integration.test.ts` or similar):
   - Store + `baseApi` + mock adapter: successful login with demo creds → auth slice populated.
   - Wrong password → mutation error with `INVALID_CREDENTIALS`; auth slice **still empty**; no `resetApiState` side effect.
   - Use `@env` stub values consistent with mock handler.

## Verification

- Run **now:** `npx tsc --noEmit && CI=true npm test -- auth && npx eslint src/features/auth && npx prettier --check src/features/auth`
- Manual: wrong password → inline error; correct password → session stored (app nav may complete in 4.5).
- Credentials never logged (doc 10).

## Keeping the docs true  (always)

If error codes differ from doc 11, fix code not docs.

## Definition of done

- [ ] PasswordEntry replaces placeholder with F2 error UX.
- [ ] Back/editar/submit rules match doc 07 §4.
- [ ] T2 login + 401 scoping tests pass.
- [ ] a11y: error announced (live region), password toggle labels.
- [ ] Lint/format clean.

## Next

`run substep 4.5` — session restore, `/me` 401, shell wiring, manual smoke, STEP gate.
