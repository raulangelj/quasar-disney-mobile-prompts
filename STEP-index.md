# quasar-disney-mobile — STEP Index

The living roadmap. Every STEP, its status, and a one-line scope. **This is the first
place to look to understand where the project is.** Keep it current as STEPs are planned,
worked, and completed.

> Status values: **Planned** · **In progress** · **Done** (archived to `prompts/`) ·
> **Deferred** (consciously not needed now; keep a revisit trigger) ·
> **Abandoned** (reserved but won't be built — keep the row so the number is never reused).
> Flip a STEP to **In progress** when you start it, so the overlap warning can see it.
> STEP numbers are global and never reset (see `METHOD.md` §1, §8).
> **What to do next** is always derivable from this index — see the next-action resolver in
> `METHOD.md` §10.
>
> **Reserving a number (teams):** adding a STEP row *is* reserving its number, on `prompts/`'s
> shared trunk (not a `step-NNNN` branch). Pull `prompts/`, take `max + 1`, add the row, then
> **commit and push immediately** — before branching or working. If the push is rejected, pull,
> renumber, push again. Before every push, even a clean merge, scan for duplicates
> (`grep -oE '^\|[[:space:]]*STEP-[0-9]+' prompts/STEP-index.md | grep -oE 'STEP-[0-9]+' | sort | uniq -d`)
> — two appended rows merge with no
> conflict into a silent duplicate. See `runbooks/collaboration.md`.
> **Owner** = who's on it; **Repos** = the repos it expects to touch (a *projection* that may
> change — it powers the overlap warning, it doesn't reserve anything). Solo, leave them blank.

## Phase 1 — POC

> **Phase plan set in STEP-1.2** — see `Code/quasar-disney-mobile-docs/architecture/02-phasing-roadmap.md`.
> Phase 1 is a functional POC with **visual fidelity** to supplied Disney+ reference screens, split
> into **1a** (gated by an immovable stakeholder sign-off on **2026-08-18**) and **1b** (remainder,
> undated). 1a = two-step auth (welcome → email → password + inline error) · storefront with 2
> config-driven carousel variants · Emotion theme · RTK Query `baseApi` + axios interceptors ·
> API contract + `axios-mock-adapter` mocks · tests for every reducer/endpoint/hook (UI tests
> deferred). Later: **Phase 2** hero carousel chrome, filter rail, details screen, Bitrise native
> CI · **Phase 3** backend integration (we build no backend) · **Phase 4+** playback, profiles,
> downloads.

| STEP | Title | Owner | Status | Repos (projection) | Scope (one line) |
|------|-------|-------|--------|--------------------|------------------|
| STEP-1 | Architecture | | Done | `quasar-disney-mobile-docs`, `prompts` | Architecture-first: design docs + ADRs, no code. Archived `prompts/001-poc/step-0001/`. |
| STEP-2 | Scaffold app repo & foundation | Raul Angel | Done | `quasar-disney-mobile-app`, `quasar-disney-mobile-docs`, `prompts` | Bare RN + TS scaffold, theme, atoms, navigation shell, CI gate. Archived `prompts/001-poc/step-0002/`. Boot fixes in app PR #4. |
| STEP-3 | Contract types, baseApi & mocks | Andres Montoya | In progress | `quasar-disney-mobile-app`, `quasar-disney-mobile-docs`, `prompts` | Transcribe doc 11 §7 into `src/api/types/` (types become normative), then stand up axios interceptors, RTK Query `baseApi`, and `axios-mock-adapter` with typed fixtures, constructor-injected latency/clock/failure, and Bearer `exp` checks on operations 2–5. Unit and integration tests cover envelope/cursors, `ApiError` normalization, 401 scoping, and expired JWT (doc 11 §11.3, doc 12 T1–T2). Parallel with STEP-2 after the api tree exists; no RN UI. **Planned 2026-08-17 — 5 substeps, branch `step-0003-contract-mocks`; PLAN + substep prompts in `Upcoming Prompts/`.** |
| STEP-4 | Auth feature | Raul Angel | In progress | `quasar-disney-mobile-app` | Welcome → email → password on the light surface, with `login`/`getMe` injectEndpoints, auth slice persisted to encrypted storage, and the F2 inline error driven by a simulated failed fetch. Tests: auth reducer/selectors, login success/failure, session restore, and `getMe` 401 → Welcome (T1/T2). **PLAN approved 2026-08-18 — branch `step-0004-auth-feature`; wait for STEP-3 on `main` before 4.1.** |
| STEP-5 | Storefront feature | Andres Montoya | Planned | `quasar-disney-mobile-app` | Dark-theme home: header, four-tab bar with `ComingSoon` placeholders, config-driven carousel (continue-watching, standard portrait, 3:4 hero stand-in), pagination wrappers, hero + progress composition, silent CW reload, and card tap → title alert. Tests: composition, `loadMore`/`hasMore`, unknown-variant drop+warn, and both paging axes (T1/T2). Depends on STEP-2 and STEP-3 merged. |
| STEP-6 | Integration, theme-swap & release smoke | Raul Angel | Planned | `quasar-disney-mobile-app` | Wire auth and storefront through the shell (cold-start gate, theme by session, connectivity overlay) and verify F1–F3 and A1–A6 on both platforms, including a second test theme that re-skins both modes. Cut the release build and install on two devices per `runbooks/release-deploy.md`; the manual smoke is the Phase-1 e2e layer. Depends on STEP-4 and STEP-5. |
| STEP-7 | Live + landscape carousel variants | Andres Montoya | Planned | `quasar-disney-mobile-app`, `quasar-disney-mobile-docs` | Phase 1b: unpark `'live'` and landscape in the data model/contract and add them as carousel configuration (VIVO badge, red progress, landscape tiles), not new carousel components. Unit tests for the new variant mapping. Not on the 18 Aug path; depends on STEP-6 (or at least STEP-5). |
| STEP-8 | UI tests + QA smoke checklist | Raul Angel | Planned | `quasar-disney-mobile-app` | Phase 1b: T3 render tests for atoms and screens (theme tokens, a11y props) plus the formal QA smoke checklist deferred from 1a. Completes Phase 1. Depends on STEP-7 so the new variants are covered. |

<!-- Implementation STEPs outlined 2026-08-17 by the planning session. No Check-in STEP in this
     phase (cadence 20; first due ~STEP-15–25). STEP-2 PLAN approved 2026-08-17; execution
     starts on `run substep 2.1`. -->

### STEP-3 substeps (implementation)

> PLAN and prompts live in `Upcoming Prompts/` until the STEP is archived.
> Execution starts only on an explicit `run substep 3.N` command.

| Substep | Title | Status | Produces |
|---------|-------|--------|----------|
| 3.1 | Wire types — transcribe doc 11 §7 | Done | `src/api/types/` (`card`, `container`, `envelope`, `auth`, `errors`, barrel), page-size constants; doc 11 §2/§3 handover + Version Log; app README contract-of-record line |
| 3.2 | axios client, interceptors, `baseQueryWithAuth`, `baseApi` | Done | `src/api/client/` (paths, instance, `injectStore`, both interceptors, `axiosBaseQuery`, `baseQueryWithAuth`), `src/api/sessionCleared.ts`, `src/api/baseApi.ts`; doc 03 §8.1/§8.2 seam + Version Log |
| 3.3 | Mock adapter — seams, mock JWT, five handlers | Done | `src/api/mocks/` — `createMockAdapter.ts`, `context.ts`, `jwt.ts`, `cursor.ts`, `base64url.ts`, `handlers/*.ts` (five operations + Bearer guard); `axios-mock-adapter` dependency |
| 3.4 | Demo fixtures, placeholder art, test factories | Done | `src/api/mocks/fixtures/` (`ids`, `artwork`, `homeFeed`, `continueWatching`, `user`, barrel, fixture-invariant test), `src/shared/assets/placeholder-art/*.svg` (7), `card`/`container`/`page` `*.factory.ts`; doc 03 §8.1 + Version Log |
| 3.5 | T2 integration tests + final verification gate | Planned | `src/api/**/*.test.ts` (T2), green `tsc --noEmit` · `jest` · `eslint` · `prettier --check` |

### STEP-4 substeps (implementation)

> PLAN and prompts live in `Upcoming Prompts/` until the STEP is archived.
> Execution starts only on an explicit `run substep 4.N` command. **Wait for STEP-3 on `main` before 4.1.**

| Substep | Title | Status | Produces |
|---------|-------|--------|----------|
| 4.1 | Auth slice + login / getMe endpoints | Planned | Real auth slice, `features/auth/api.ts`, T1 reducer/selectors/persist tests |
| 4.2 | Auth molecules + i18n | Planned | `AuthSheetLayout`, `WelcomeHero`, `CredentialsForm`; `auth.*` i18n keys |
| 4.3 | Welcome + EmailEntry screens | Planned | Real Welcome/Email screens, theme long-press, email validation + navigation |
| 4.4 | PasswordEntry + F2 inline error | Planned | Password screen, login UX, T2 login + INVALID_CREDENTIALS scoping tests |
| 4.5 | Session restore, /me 401, shell wiring | Planned | getMe-only LoadingGate, boot/post-login restore, T2 401 tests, manual auth smoke |

### STEP-2 substeps (implementation)

> PLAN and prompts archived to `prompts/001-poc/step-0002/`.
> Execution starts only on an explicit `run substep 2.N` command.

| Substep | Title | Status | Produces |
|---------|-------|--------|----------|
| 2.1 | Create app repo & boot both platforms | Done | `Code/quasar-disney-mobile-app/` (bare RN + TS), license, GitHub remote, iOS + Android boot |
| 2.2 | Source layout, env, register repo | Done | `src/{app,features,shared,api}/`, `.env.example`, README, `repos.yml`. Unblocks STEP-3; thin PR to `main` |
| 2.3 | Tooling, `createStore()`, CI | Done | ESLint A5/DF1/A2, Prettier, Jest `@env` + native mocks, `createStore()`, GitHub Actions Tier A |
| 2.4 | Theme, fonts, i18n, outlined wordmarks | Done | Emotion both brands × both modes, Inter, `es-419`, outlined brand SVGs, token-parity test |
| 2.5 | Shared atoms & icons | Done | Doc 07 §3.1 atoms + §6 SVG icons (T3 deferred) |
| 2.6 | Navigation shell, overlay, error boundary | Done | Two navigators, ComingSoon tabs, NetInfo overlay, LoadingGate stub, root error boundary |

### STEP-1 substeps (architecture sessions)

> Like every STEP, STEP-1 has **one owner**, run on one machine — substeps aren't split
> across people (see `runbooks/collaboration.md` §3). But architecture is a shared
> foundation, so **decide it as a group**: the best setup is the whole team in a room walking
> the sessions together while one person drives the keyboard and commits the docs.

| Substep | Session | Status | Output doc |
|---------|---------|--------|------------|
| 1.1 | System Overview, Requirements & Non-Goals | Done | `architecture/01-system-overview.md` |
| 1.2 | Phasing & Roadmap | Done | `architecture/02-phasing-roadmap.md` |
| 1.3 | Architecture Overview & Component Boundaries | Done | `architecture/03-architecture-overview.md` |
| 1.3a | Native App Architecture *(conditional)* | Done | `architecture/15-native-app-architecture.md` |
| 1.4 | Data Model, Ownership & Retention | Done | `architecture/04-data-model.md` |
| 1.5 | Scaling & Performance | Done | `architecture/05-scaling-performance.md` |
| 1.6 | Security & Threat Model | Done | `architecture/06-security-threat-model.md` |
| 1.6a | Identity & Auth *(conditional)* | Done | `architecture/16-identity-auth.md` |
| 1.7 | UI / Design System | Done | `architecture/07-ui-design-system.md` |
| 1.8 | Infrastructure & Deployment | Done | `architecture/08-infrastructure-deployment.md` |
| 1.9 | Environments | Done | `architecture/09-environments.md` |
| 1.10 | Observability | Done | `architecture/10-observability.md` |
| 1.11 | Interface Contracts | Done | `architecture/11-interface-contracts.md` |
| 1.12 | Test Strategy | Done | `architecture/12-test-strategy.md` |
| 1.13 | Glossary | Done | `architecture/13-glossary.md` |
| 1.14 | Cross-Cutting Review | Done | `prompts/001-poc/step-0001/quasar-disney-mobile-STEP-1-REVIEW.md` |

<!-- Conditional sessions: enumerate every conditional-*.md template and include/defer/skip it
     in the STEP-1 PLAN's "Conditional sessions considered" table. Add an index row only when
     one is included. Slot included conditionals under a LETTERED substep after the related
     owning session, and run them BY NAME, not number (for example, "run the identity-auth
     session" → conditional-identity-auth.md). The output doc takes the next number above the
     core set. EXAMPLE ONLY — do not parse this as a real row; real rows start at the left
     margin above this comment, with the assigned substep and doc number:
       | 1.Xa | Conditional topic | Planned | `architecture/NN-topic.md` |
     If a conditional row is later added and then consciously not needed under the current
     project shape, mark it Deferred with the revisit trigger in the PLAN/risk register. -->

## How to add a STEP
See `prompts/README.md` for the authoring recipe.
