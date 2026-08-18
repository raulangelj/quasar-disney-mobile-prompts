# quasar-disney-mobile — STEP-2 PLAN: Scaffold app repo & foundation

**Phase:** Phase 1 — POC
**Owner:** Raul Angel
**Status:** Done
**Date:** 2026-08-17
**Branch:** `step-0002-app-scaffold`
**Repos (projection):** `quasar-disney-mobile-app` (create) → `quasar-disney-mobile-docs` (`registries/repos.yml` + doc pointers) → `prompts` (index only)

> Create the application repo `quasar-disney-mobile-app`, get a bare React Native TypeScript app booting on iOS and Android, and wire the composition-root foundation (layout, theme, store factory, lint/CI, shared atoms, navigation shell) so STEP-3 can start as soon as `src/api/` exists and STEPs 4–5 can land features without fighting the scaffold.

## Motivation

STEP-1 locked the architecture. There is still no application repo (`registries/repos.yml` lists only the docs hub and `prompts/`). Auth, storefront, and the contract/mock layer all need a host: folders, Emotion theme, `createStore()`, ESLint import rules, and a JS CI gate. This STEP is that host.

It is also the top schedule risk (**RISK-0003**). Sign-off is **2026-08-18**. The original Saturday-midday scaffold checkpoint has already passed because STEP-1 was still in flight. Treat 2.1 as the critical path: if Android is not booting after a focused attempt, drop it for the rest of this STEP, continue on iOS, and bring Android up before STEP-6.

STEP-3 (Andres, contract + mocks) is designed to run **in parallel after 2.2**. 2.1–2.2 should land on `main` as a thin first PR so he has a cloneable repo and an empty `src/api/` tree. 2.3–2.6 continue on the same `step-0002-app-scaffold` branch.

## Decisions already locked

- root `.throughstone/local-user.md` — read Experience level and Communication style; do not copy those values into this PLAN.
- `registries/risks.yml` — **RISK-0003** (scaffold on two platforms) is this STEP's critical path; **RISK-0006** / **OQ-13** (outline wordmarks) close here; **RISK-0001** / **RISK-0011** (no T3 render tests in 1a — write a11y props anyway); **RISK-0017** (A2 lint is pattern-matching).
- `architecture/12-test-strategy.md` §11 — foundation obligations: `createStore()` factory, Jest `@env` stub + native-module mocks, stamp CI, ESLint four rules + Prettier. Contract-layer factories / adapter constructor seams belong to **STEP-3**.
- `architecture/03-architecture-overview.md` §8 — `src/{app,features,shared,api}/`; import rules; shell is composition root. Repo name `quasar-disney-mobile-app` at `Code/quasar-disney-mobile-app/`.
- `architecture/15-native-app-architecture.md` — bare RN, no Expo, Hermes, online-only NetInfo overlay (interface-up, ADR-0014), portrait lock, internet-only permissions, local installs.
- `architecture/07-ui-design-system.md` — Emotion (ADR-0019); theme × mode (`dinsey` / `ember` × `app` / `auth`); Inter bundled (ADR-0012); closed atom list §3.1; navigation §4; i18n `es-419` forced.
- `architecture/09-environments.md` — one gitignored `.env`; committed `.env.example` from the hub template's *keys* (`API_BASE_URL`, `DEMO_EMAIL`, `DEMO_PASSWORD`); `react-native-dotenv` (ADR-0015).
- `architecture/10-observability.md` §5.2 — one root error boundary inside the theme provider; **OQ-33 closed below**.
- `architecture/11-interface-contracts.md` §3.1 — split by the planning session: this STEP creates the `src/api/` tree and points the README at doc 11; **STEP-3** transcribes §7 into `src/api/types/` and stands up `baseApi` / mocks. Doc 11 stays normative until those types exist.
- `architecture/08-infrastructure-deployment.md` — lockfiles only; no `.nvmrc` / toolchain pin file. CI is the JS GitHub Actions gate (ADR-0018), not Bitrise.
- Proprietary project-license posture (`.throughstone/project-license`) — run `scripts/apply-project-license.sh` on the new repo (no project `LICENSE`; copy `LICENSE-THROUGHSTONE` + write `LICENSING.md`).
- ADR-0001, ADR-0003, ADR-0004, ADR-0011, ADR-0012, ADR-0013, ADR-0014, ADR-0015, ADR-0018, ADR-0019, ADR-0020.

### Closed in this PLAN

| ID | Resolution |
|----|------------|
| **OQ-33** | Error-boundary fallback uses the **active** surface mode. Compose `Wordmark` + `Text` (`body`) + `Button` (`solid` / `md`, tone `onDark` in `app` mode and `onLight` in `auth` mode). i18n: `errorBoundary.message` = *Algo salió mal.* · `errorBoundary.retry` = *REINTENTAR*. Retry remounts the subtree via a reset key. No stack trace, no error code on screen. |
| **OQ-13** | Closed in **2.4** by outlining wordmark (and brand-strip) SVG `<text>` to paths. Owner: this STEP. |
| GitHub remote | Private repo `raulangelj/quasar-disney-mobile-app` (matches the other two remotes; proprietary). |

## Substeps

| # | Title | Produces | Depends on | Open questions |
|---|-------|----------|------------|----------------|
| 2.1 | Create app repo & boot both platforms | `Code/quasar-disney-mobile-app/` (bare RN + TS), license posture, GitHub remote, lockfiles, iOS + Android hello-world | — | Done 2026-08-18: both platforms booted; RISK-0003 closed |
| 2.2 | Source layout, env, register repo | `src/{app,features,shared,api}/` tree, `.env.example`, filled README, `repos.yml` entry. **Unblocks STEP-3.** Thin PR to `main`. | 2.1 | Done 2026-08-18 |
| 2.3 | Tooling, `createStore()`, CI | ESLint (A5/DF1/A2), Prettier, Jest `@env` + native mocks, `createStore()` with stub slots, GitHub Actions Tier A | 2.2 | Done 2026-08-18 |
| 2.4 | Theme, fonts, i18n, outlined wordmarks | Done | Emotion `ThemeProvider` + `ModeProvider`, `dinsey` + `ember` × both modes, Inter, `react-i18next` `es-419`, outlined brand SVGs, token-parity test | 2.3 | Done 2026-08-18 |
| 2.5 | Shared atoms & icons | Done | Doc 07 §3.1 atoms + §6 SVG icons in `shared/ui/`; a11y props written now | 2.4 | T3 deferred (RISK-0001) |
| 2.6 | Navigation shell, overlay, error boundary | Done | Root navigators + ComingSoon tabs, NetInfo overlay, LoadingGate stub, error boundary (OQ-33), feature placeholder screens | 2.5 | Done 2026-08-18 |

## Test plan

T3 component render tests are **out of this STEP** (doc 12 §2, RISK-0001). T2 integration against `baseApi` + mock adapter is **STEP-3**. Device e2e is declined (RISK-0015). The STEP gate is the Tier A JS job.

| Test tier / surface | Substep(s) | Tests to create or update | Run timing | Command / gate | Notes |
|---------------------|------------|---------------------------|------------|----------------|-------|
| Type check (T0) | 2.3–2.6 | `tsc --noEmit` across `src/` | Per substep from 2.3 | `npx tsc --noEmit` | `strict: true`; `noUncheckedIndexedAccess` off |
| Unit (T1) | 2.3 | `createStore()` returns independent stores; persist whitelist is exactly `['auth']` | Per substep | `npm test` | Full persist-vs-RTK-Query-cache assertion waits until STEP-3 registers `baseApi` |
| Unit (T1) | 2.4 | Theme token-parity: `dinsey` and `ember`, `app` and `auth` modes, identical key structure (doc 12 §3.4 / criterion A1 structural half) | Per substep | `npm test` | |
| Integration (T2) | — | — | — | — | Needs `baseApi` + adapter (STEP-3) |
| API / contract | — | — | — | — | STEP-3 transcribes types |
| End-to-end | 2.1, 2.6 | Manual: both platforms boot; after 2.6, overlay/error-boundary/ComingSoon are visible | Per those substeps | Xcode / Android Studio run | Not Detox |
| Security / authorization | — | — | — | — | No live auth yet |
| Performance / load | — | — | — | — | Declined (doc 12 §9) |
| Lint / format | 2.3–2.6 | ESLint four architecture rules + Prettier | Per substep from 2.3 | `npx eslint .` · `npx prettier --check .` | Also the CI job |
| Native boot | 2.1 | App launches on iOS simulator and Android emulator | 2.1 (Android drop allowed) | `npx react-native run-ios` / `run-android` | RISK-0003 |

**Run timing:** tests that exist run **per substep** from 2.3 onward. Final STEP gate (before review): `npx tsc --noEmit && npm test && npx eslint . && npx prettier --check .` in the app repo (the GitHub Actions job must be green on the PR).

## Open questions

None blocking. OQ-13 and OQ-33 are closed in this PLAN. Remaining architecture OQs (OQ-05 Bitrise, OQ-29 reachability, OQ-34 backend, etc.) are later phases.

## Ground rules

- **Calibrate communication from root `.throughstone/local-user.md`.** Do not copy those values into prompts as project facts.
- **Do not execute this STEP from the whole-STEP command.** Wait for `run substep 2.1` (then 2.2, …).
- **One owner, one branch name** `step-0002-app-scaffold` in every repo this STEP touches. Index edits (`prompts/STEP-index.md`) stay on `prompts/` `main`, never on the step branch.
- **Overlap:** no other STEP is `In progress`. STEP-3 projects the same app repo **by design** after 2.2. Warn Andres when 2.2 merges; do not block.
- **2.1–2.2 merge early.** After 2.2, open/merge a thin PR to `main` so STEP-3 can branch from trunk. Continue 2.3–2.6 on `step-0002-app-scaffold`.
- **Tests ship with the code** unless the substep records why (2.1 generated template; 2.2 folders/docs; 2.5/2.6 T3 deferred).
- **Code is documented as it's written** (`coding-standards/README.md` + `typescript.md`): TSDoc on every function/component.
- **No Expo. No `styled-components`. No `fetch`/`axios` outside `src/api/`.** Screens never fetch.
- **Do not transcribe doc 11 §7** and do not implement `baseApi` / interceptors / mocks — that is STEP-3. Leave stub files / empty folders only.
- **Do not build auth or storefront screens** — placeholder routes only.
- **Secrets:** commit `.env.example` only; gitignore `.env`. Jest maps `@env` to a committed stub.
- **Accepted risks stay visible.** Update RISK-0003 owner and RISK-0006 status when the matching substep lands. Do not silently close RISK-0001.
- **RN New Architecture / CLI defaults:** take the current stable CLI template defaults except where architecture forbids them (no Expo). Commit every lockfile the scaffold generates (doc 08 §4).
- **Native project name** `DinseyApp`; workspace folder `Code/quasar-disney-mobile-app/`; display name `Dinsey-`.

## Definition of done

- [x] `Code/quasar-disney-mobile-app/` exists as its own git repo, private GitHub remote set, proprietary license posture applied, README stamped and filled.
- [x] App boots on iOS; Android boots or is explicitly dropped per RISK-0003 with a follow-up before STEP-6.
- [x] `src/{app,features/auth,features/storefront,shared/{theme,ui,i18n,analytics},api/{types,mocks,client}}` exist; 2.1–2.2 PRs opened (app #1, docs #2) — merge to unblock STEP-3.
- [x] `registries/repos.yml` lists the app repo; doc 03 / doc 11 pointers updated for the split (tree here, types in STEP-3).
- [x] `createStore()` factory with stub `auth` slice and persist whitelist `['auth']`; Jest `@env` stub; encrypted-storage and NetInfo mocked in Jest.
- [x] ESLint enforces DF5/A5, DF1 (`axios`/`fetch` only in `src/api/`), A2 (no hex/`rgba(` outside `shared/theme/`); Prettier; GitHub Actions runs `tsc --noEmit` · `jest` · `eslint` · `prettier --check`.
- [x] Emotion theme: `dinsey` + `ember`, modes `app` + `auth`, token-parity test green; Inter linked; i18n forced `es-419`.
- [x] Brand SVGs outlined (RISK-0006 / OQ-13 closed).
- [x] Shared atoms (doc 07 §3.1) and 13 SVG icons exist with a11y props; no T3 tests (reason: RISK-0001).
- [x] Shell: two navigators, ComingSoon tabs, NetInfo overlay, LoadingGate stub, root error boundary (OQ-33).
- [x] The STEP test plan is complete: each code-changing substep either added/updated its tests or records why not.
- [x] Final gate green: `npx tsc --noEmit && npm test && npx eslint . && npx prettier --check .`
- [x] STEP review passed; `prompts/STEP-index.md` updated; STEP archived to `prompts/001-poc/step-0002/`. *(Review passed 2026-08-18; archived same day. App boot fixes in PR #4.)*
