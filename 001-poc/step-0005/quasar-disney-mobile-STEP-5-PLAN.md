# quasar-disney-mobile — STEP-5 PLAN: Storefront feature

**Phase:** Phase 1 — POC (1a)
**Owner:** Raul Angel
**Status:** Done 2026-08-18 — review passed, merged app PR #6, archived
**Date:** 2026-08-18
**Branch:** `step-0005-storefront-feature`
**Repos (projection):** `quasar-disney-mobile-app` → `prompts` (index only)

> Build the dark-theme home: inject the three feed endpoints on `baseApi`, pagination wrappers, client merge of hero + continue-watching + remaining rows, config-driven carousel with two Phase-1a variants (continue-watching + standard portrait, hero as 3:4 stand-in), silent CW reload on revisit, and card tap → native Alert with the title. Satisfies the storefront half of the Phase-1a sign-off path (**F3** card tap, visual fidelity on home).

## Motivation

STEP-2 shipped the tab shell with a placeholder `HomeScreen`. STEP-3 merged typed fixtures and paginated mock handlers. STEP-4 (Raul) landed auth — login now switches to app tabs, but home is still a stub. This STEP is the **dark** surface: the largest UI surface in 1a, the consumer of the contract Andres authored, and the feature STEP-6 wires into the three-query cold-start gate.

STEP-4 and STEP-5 are disjoint by design (**DF5**). Storefront must not import auth. The shell already mounts `HomeScreen` as the live tab; this STEP replaces the placeholder and registers real RTK Query endpoints.

**Dependency:** branch from `main` with STEP-3 and STEP-4 merged (needs `baseApi`, mock handlers, authenticated session to exercise feeds in manual smoke).

## Decisions already locked

- root `.throughstone/local-user.md` — read Experience level and Communication style; do not copy those values into this PLAN.
- `registries/risks.yml` — **RISK-0001** / **RISK-0011** (no T3 render tests in 1a — write a11y props anyway); **RISK-0005** (Dinsey- placeholder brand, not Disney marks — **DF10**).
- `architecture/03-architecture-overview.md` §6 Flow 2, §8.1 — storefront injects `getHomeFeed`, `getContinueWatching`, `getContainerResources`; pagination wrappers; composed-home merge; silent CW reload.
- `architecture/04-data-model.md` §1.2–§1.4, §6 — hero + CW + remaining composition; CW under hero; silent refetch replaces only the `progress` container; empty CW row omitted.
- `architecture/07-ui-design-system.md` §3.1–§3.4, §7 — tile molecules, one config-driven carousel organism, skeletons at real geometry, visible-count per variant, `AppHeader` in shell.
- `architecture/11-interface-contracts.md` §4.4, §6–§7 — unknown `variant` drops row + `console.warn`; page sizes; wire shapes.
- `architecture/12-test-strategy.md` §3.2 — T1 composition + unknown variant; T2 both paging axes + silent CW reload.
- `architecture/15-native-app-architecture.md` §8 — `useHomeFeed` / `useCarouselPage` hook surface (`{ items, loadMore, hasMore, isLoading, error }`).
- `inputs/ui/disney-plus-reference-screens.md` §5–§6 — home layout reference; substitute Dinsey- brand (**DF10**).
- ADR-0005, ADR-0006, ADR-0007, ADR-0013, ADR-0019, ADR-0020.
- STEP-3 PLAN Q4 — fixture artwork strings resolve at render time in the storefront card layer, not in wire data.

### Closed in this PLAN

| ID | Resolution |
|----|------------|
| **Artwork resolution** | Static filename → bundled asset map in `features/storefront/ui/placeholderArt.ts` (the seven SVGs under `src/shared/assets/placeholder-art/`). Add **`react-native-svg-transformer`** + Metro config so `require()` of SVGs yields a renderable source for the shared `Artwork` atom. Unknown filenames fall back to skeleton — no throw. |
| **Cold-start gate scope** | STEP-5 fetches feeds when `HomeScreen` mounts / focuses. The shell **does not** wait on HomeFeed + CW yet — that three-query gate is **STEP-6**. Post-login may briefly show placeholder loading skeletons on home until feeds resolve; acceptable until STEP-6. |
| **AppHeader ownership** | `AppHeader` organism lives in `src/app/shell/` (doc 07 §3.3). Built in 5.4; `HomeScreen` composes it above the feed list. |
| **Card tap** | Native `Alert.alert(card.title)` via a single shared handler passed down (doc 03 Flow 2 step 4, **DF7**). No details route. |
| **Phase-1a variants** | `'hero'` (3:4 stand-in), `'progress'` (continue-watching), `'standardPortrait'`. `'standardLandscape'` and `'live'` are **STEP-7** — unknown variants still drop + warn per §4.4. |
| **Hero chrome** | No spotlight chrome (Phase 2). Hero row renders like a large portrait tile using the 3:4 ratio (OQ-24). |
| **Filter rail** | Not in 1a (doc 02). No filter pills on home. |

## Substeps

| # | Title | Produces | Depends on | Open questions |
|---|-------|----------|------------|----------------|
| 5.1 | Feed endpoints + pagination hooks | `features/storefront/api.ts`, `hooks/usePaginatedContainers.ts`, `hooks/useCarouselPage.ts`, T1 hook tests | STEP-3 on `main` | — |
| 5.2 | Artwork resolver + tile molecules | Metro SVG transformer, `placeholderArt.ts`, `SectionHeader`, `PortraitTile`, `ProgressTile`, storefront i18n keys for CW labels | 5.1 | — |
| 5.3 | Config-driven carousel + variant map | `CarouselRow` organism, `variantConfig.ts`, unknown-variant drop + warn, T1 variant tests | 5.2 | — |
| 5.4 | Composed home + silent CW + shell chrome | `useComposedHome`, `HomeFeedList`, `AppHeader`, focus-triggered CW refetch, T1 composition + T2 silent-reload tests | 5.1, 5.3 | — |
| 5.5 | HomeScreen + card tap + final gate | Real `HomeScreen`, shared `onCardPress` → Alert, T2 paging-axis tests, manual F3 smoke, STEP review prep | 5.4 | — |

## Test plan

T3 render tests are **out of this STEP** (RISK-0001). Device e2e is STEP-6. The STEP gate is Tier A JS.

| Test tier / surface | Substep(s) | Tests to create or update | Run timing | Command / gate | Notes |
|---------------------|------------|---------------------------|------------|----------------|-------|
| Type check (T0) | 5.1–5.5 | `tsc --noEmit` | Per substep from 5.1 | `npx tsc --noEmit` | |
| Unit (T1) | 5.1 | Pagination hooks: `hasMore`/`loadMore` merge pages; trust `nextCursor` | 5.1 | `npm test` | ADR-0005 |
| Unit (T1) | 5.3 | Unknown `variant` dropped; `console.warn` emitted; known variants map to tile components | 5.3 | `npm test` | doc 11 §4.4 |
| Unit (T1) | 5.4 | Composition order: hero → `progress` → remaining HomeFeed containers; empty CW omitted | 5.4 | `npm test` | ADR-0006 |
| Integration (T2) | 5.4 | Silent CW refetch replaces only the `progress` container | 5.4 | `npm test` | doc 04 §6 |
| Integration (T2) | 5.5 | Vertical + horizontal `nextCursor` exhaustion via real injected endpoints + mock adapter | 5.5 | `npm test` | doc 12 §3.2; reuse `t2World.factory` pattern |
| Unit (T1) | 5.2 | `remainingMinutes` via i18n template, not hardcoded phrase | 5.2 | `npm test` | DF8 |
| End-to-end | 5.5 | Manual: login → home rows render → horizontal scroll loads more → tap card shows title alert | 5.5 | iOS simulator | F3 |
| Lint / format | 5.1–5.5 | ESLint + Prettier | Per substep | `npx eslint .` · `npx prettier --check .` | CI job |

**Run timing:** tests run **per substep** from 5.1. Final STEP gate before review: `npx tsc --noEmit && CI=true npm test && npx eslint . && npx prettier --check .`.

## Open questions

None blocking.

## Ground rules

- **Calibrate communication from root `.throughstone/local-user.md`.** Do not copy those values into prompts as project facts.
- **Do not execute this STEP from the whole-STEP command.** Wait for `run substep 5.1` (then 5.2, …).
- **One owner, one branch** `step-0005-storefront-feature` in the app repo. Index edits (`prompts/STEP-index.md`) stay on `prompts/` `main`.
- **Branch from `main`.** Rebase if trunk moves.
- **Tests ship with the code** — see test plan; T3 explicitly deferred.
- **Code is documented as it's written** (`coding-standards/README.md` + `typescript.md`): TSDoc on every function/component.
- **No Expo. No `styled-components`. No `fetch`/`axios` in features.** Screens/hooks use generated RTK Query hooks only.
- **Storefront ↛ Auth.** Shell may read auth selectors; storefront must not import auth or auth `api.ts`.
- **Do not wire the three-query LoadingGate** — home fetches on mount; STEP-6 extends the shell gate.
- **DF10:** no Disney/Marvel/Star Wars/hulu/ESPN marks in UI copy or assets.
- **Container.name** is rendered verbatim (feed data, not i18n). **Card.title** and CW label templates use i18n.

## Definition of done

- [x] `features/storefront/api.ts` injects `getHomeFeed`, `getContinueWatching`, `getContainerResources` on `baseApi`.
- [x] Pagination hooks expose `{ items, loadMore, hasMore, isLoading, error }` per ADR-0005 / doc 15 §8.
- [x] Composed home order: hero → CW `progress` → remaining HomeFeed containers (T1).
- [x] Config-driven carousel renders `hero`, `progress`, and `standardPortrait` variants; unknown variants drop + warn (T1).
- [x] Silent CW reload on home focus replaces only the `progress` container (T2).
- [x] Both vertical and horizontal paging axes exhaust `nextCursor` correctly (T2).
- [x] `HomeScreen` with `AppHeader`, skeleton loading, `ErrorState` on feed failure, card tap → Alert with title (F3 manual).
- [x] a11y: grouped tile labels, section headers as `header`, decorative art hidden (best effort without T3).
- [x] The STEP test plan is complete: each code-changing substep either added/updated its tests or records why not.
- [x] Final gate green: `npx tsc --noEmit && CI=true npm test && npx eslint . && npx prettier --check .`
- [x] STEP review passed; `prompts/STEP-index.md` updated; STEP archived to `prompts/001-poc/step-0005/` — merged app PR #6 2026-08-18.
