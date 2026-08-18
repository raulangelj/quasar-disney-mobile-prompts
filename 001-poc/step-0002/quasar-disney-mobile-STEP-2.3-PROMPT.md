# quasar-disney-mobile — STEP-2.3: Tooling, createStore(), CI

> **How to run:** Tell your agent *"run substep 2.3"*. Self-contained — executable cold in a fresh chat.

## Context

The repo and `src/` tree exist (2.2, on `main`). This substep lands doc 12 §11 items 1–4: `createStore()` factory, Jest `@env` + native mocks, ESLint architecture rules + Prettier, GitHub Actions Tier A. It also adds the foundation dependencies the store needs (RTK, persist, encrypted-storage, NetInfo) **without** implementing `baseApi`, interceptors, or mocks.

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-2-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-2-PLAN.md`
- `Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md` §4.2, §4.3, §5, §7, §11
- `Code/quasar-disney-mobile-docs/architecture/03-architecture-overview.md` §8.2
- `Code/quasar-disney-mobile-docs/adr/ADR-0003-persist-auth-secure-storage.md`
- `Code/quasar-disney-mobile-docs/adr/ADR-0015-*.md`, `ADR-0018-*.md`, `ADR-0020-*.md`
- `Code/quasar-disney-mobile-docs/templates/ci/code-repo-ci.yml` + `templates/ci/README.md`
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`
- `Code/quasar-disney-mobile-app/README.md`

## Scope

**Owns:** ESLint/Prettier, Jest seams, `createStore()` + persist config + stub `auth` reducer, `react-native-dotenv` Babel plugin, GitHub Actions job, README Testing/Setup updates.

**Does not:** theme tokens (2.4), atoms (2.5), navigation/overlay (2.6), `createApi` / axios-mock-adapter / fixtures (STEP-3).

## Your task

1. Branch `step-0002-app-scaffold` in the app repo (recreate from `main` if 2.2 merged). Continue docs-hub edits on the same branch name only if README-in-hub isn't needed; prefer putting run instructions in the **app** README.
2. **Dependencies (runtime where required):** `@reduxjs/toolkit`, `react-redux`, `redux-persist`, `react-native-encrypted-storage`, `@react-native-community/netinfo`. **Dev:** ESLint + typescript-eslint + `eslint-plugin-import` (or the RN community ESLint config plus restricted-paths), Prettier, `react-native-dotenv`. Follow `runbooks/dependency-supply-chain.md` at a glance (known license, not a surprise telemetry SDK).
3. **`createStore()`** in `src/app/store/` (camelCase `.ts` files):
   - Factory, **not** a module-level singleton. App shell will call it once in 2.6; tests call it per test.
   - Stub `auth` reducer: `{ isAuthenticated: false }` (or equivalent) so persist has a slice name. STEP-4 replaces this file's contents; keep the **slot** (`auth:` key) stable.
   - **Do not** register `baseApi.reducer` / middleware yet — leave a clearly commented slot in `createStore()` so STEP-3 adds one line.
   - `persistConfig.whitelist = ['auth']`. Storage engine: `react-native-encrypted-storage`. Do not persist anything else.
   - Export `injectStore` only if you need it for interceptors later; if unused, skip — STEP-3 can add it. Do not import `axios` here.
4. **Jest**
   - React Native preset. `coverageProvider: 'babel'`. Colocated `*.test.ts`.
   - Map `@env` to a **committed stub** (e.g. `src/test/env.stub.ts` or `jest/env.ts`) with fixed `DEMO_EMAIL` / `DEMO_PASSWORD` / `API_BASE_URL`. Tests never read the real `.env`.
   - Manual mocks for `react-native-encrypted-storage` (in-memory `getItem`/`setItem`) and `@react-native-community/netinfo` (official mock or a tiny controllable stub).
   - Do **not** blanket-silence `console.warn`.
   - Tests to write now:
     - `createStore.test.ts`: two `createStore()` calls do not share state.
     - persist whitelist is exactly `['auth']` (assert on the exported config). Do not yet assert RTK Query cache absence — `baseApi` does not exist.
5. **`react-native-dotenv`:** Babel plugin so `import { DEMO_EMAIL } from '@env'` works in app code. Keep the 2.2 `declare module '@env'`.
6. **ESLint** (doc 12 §7.1 — all four):
   - `import/no-restricted-paths`: `features/auth` ↛ `features/storefront` and reverse; `shared/**` ↛ `features/**`.
   - Restricted import of `axios` and `fetch` outside `src/api/`.
   - `no-restricted-syntax` (or equivalent) banning hex / `rgba(` color literals outside `src/shared/theme/`. Honest about RISK-0017 reach.
   - Ban `styled-components` imports (ADR-0019).
   Allow test files to import `src/api/mocks/` factories later (doc 12 §4.1) — don't write a rule that will block STEP-3 tests.
7. **Prettier** with a committed config. Match typical RN/TS (no tabs bikeshed).
8. **CI:** stamp `templates/ci/code-repo-ci.yml` → `.github/workflows/ci.yml`. **Replace the failing placeholder.** Node **20**, `npm ci`, then:

   `npx tsc --noEmit && npm test && npx eslint . && npx prettier --check .`

   Speed budget: under two minutes. No `npm audit` (RISK-0010). No coverage threshold (RISK-0016); coverage **text summary** in the job log is fine.
9. `tsconfig`: `strict: true`, `noUncheckedIndexedAccess` off, `noEmit` for `tsc`. Path aliases if useful (`src/*`).
10. Update app README Testing + Setup. Confirm iOS still boots (Metro + store provider can wait until 2.6 if wrapping `App` now is trivial — wrapping with `Provider` here is allowed and recommended so 2.4/2.6 don't fight entry files).

## Verification

- Run **now:** `npx tsc --noEmit && npm test && npx eslint . && npx prettier --check .`
- Happy path: empty store factory works; two stores independent.
- Persist config whitelist is `['auth']` only.
- ESLint fails a **deliberate** sample if you add a checked-in fixture under a test that you then delete — or document a tiny `scripts/`-less sanity: temporarily add a hex in `src/app/` and confirm lint catches it, then revert. Do that once.
- CI file must not still `exit 1` on "Configure me".

## Keeping the docs true  (always)

If `createStore` path differs from "shell owns the store", update doc 03 §4 wording only if it named a wrong path. RISK-0016/0017 stay open. Do not add Codecov.

## Definition of done

- [ ] `createStore()` factory + stub auth slot + persist whitelist `['auth']`.
- [ ] Jest `@env` stub + encrypted-storage + NetInfo mocks.
- [ ] Two T1 tests above, green.
- [ ] Four ESLint architecture rules + Prettier.
- [ ] `.github/workflows/ci.yml` runs T0 + jest + eslint + prettier on Node 20.
- [ ] `tsc --noEmit && npm test && npx eslint . && npx prettier --check .` green locally.
- [ ] README Testing section matches.
- [ ] TSDoc on exported store helpers.

## Next

Mark 2.3 Done in the PLAN and index. **Fresh chat:** `run substep 2.4`.
