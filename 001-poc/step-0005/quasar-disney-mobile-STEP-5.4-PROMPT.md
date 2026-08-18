# quasar-disney-mobile — STEP-5.4: Composed home + silent CW + shell chrome

> **How to run:** Tell your agent *"run substep 5.4"*. Self-contained — executable cold in a fresh chat.

## Context

Endpoints, hooks, tiles, and `CarouselRow` exist (5.1–5.3). This substep implements ADR-0006 client merge, doc 04 silent CW reload, the vertical `HomeFeedList`, and the shell `AppHeader`.

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-5-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-5-PLAN.md`
- `Code/quasar-disney-mobile-docs/architecture/04-data-model.md` §6
- `Code/quasar-disney-mobile-docs/adr/ADR-0006-two-endpoint-home-composition.md`
- `Code/quasar-disney-mobile-docs/architecture/07-ui-design-system.md` §3.3–§3.4 (AppHeader, skeletons, empty CW)
- `Code/quasar-disney-mobile-app/src/features/storefront/hooks/`, `ui/CarouselRow.tsx`
- `Code/quasar-disney-mobile-app/src/app/navigation/AppTabsNavigator.tsx`
- `Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md` §3.2

## Scope

**Owns:** `useComposedHome` hook (+ pure `composeHomeContainers` helper), `HomeFeedList`, `AppHeader` in `src/app/shell/`, focus-based silent CW refetch hook, T1 composition tests, T2 silent-reload test.

**Does not:** final `HomeScreen` polish / card tap wiring (5.5), three-query LoadingGate (STEP-6), `ErrorState` molecule if deferred — build minimal inline error or add shared `ErrorState` here if home needs it.

## Your task

1. On `step-0005-storefront-feature`.
2. **`composeHomeContainers(homeFeedPages, cwContainers)`** — pure function (T1 target):
   - Extract hero (`variant === 'hero'`) from HomeFeed — first hero only
   - Extract `progress` container from CW response (not from HomeFeed — ADR-0006)
   - Remaining HomeFeed containers exclude hero and any stray `progress`
   - Order: `[hero?, progress?, ...remaining]`
   - If CW container has zero `resources`, **omit** the progress row (doc 07 §3.4)
3. **`useComposedHome`** — orchestrates two `usePaginatedContainers` instances (HomeFeed + CW initial fetch) and returns `{ rows, loadMoreRows, hasMoreRows, isLoading, error, silentReloadCw }`.
4. **`useSilentContinueWatchingReload`** — on home tab focus (`useFocusEffect` from React Navigation):
   - Refetch CW only; merge result into composed rows replacing the prior `progress` container
   - No full-screen loader; stale-while-revalidate (doc 04 §6)
5. **`HomeFeedList`** — vertical `FlatList` of `CarouselRow`; `onEndReached` → vertical `loadMoreRows`; list header = `AppHeader`; skeleton rows while initial load
6. **`AppHeader`** — shell organism (doc 07 §3.3):
   - Dinsey- compact mark / wordmark, cast + overflow icon buttons (inert in 1a OK)
   - Safe-area top inset applied here only
   - Reads `useGetMeQuery` or selector for `userName` if shown — **must not import auth feature**; use RTK hook from storefront's re-export or shell reading `baseApi` hook only if already exported — prefer shell importing `useGetMeQuery` from a neutral path: **`@features/auth/api` is forbidden from shell if it creates auth↔storefront coupling** — check doc 03: shell → auth is allowed. Shell may import auth hooks for chrome.
7. **T1:** `composeHomeContainers` ordering tests with factories
8. **T2:** silent reload test — initial compose, refetch CW with different card, assert only progress row changed

## Verification

- Run **now:** `npx tsc --noEmit && CI=true npm test -- composeHome silentReload useComposedHome && npx eslint src/features/storefront src/app/shell && npx prettier --check src/features/storefront src/app/shell`

## Keeping the docs true  (always)

Shell importing `useGetMeQuery` for header chrome is allowed (doc 03 §8.2). Storefront still must not import auth.

## Definition of done

- [ ] Composed home order matches ADR-0006 (T1).
- [ ] Silent CW reload replaces only progress row (T2).
- [ ] `AppHeader` + `HomeFeedList` render together.
- [ ] Vertical paging on HomeFeed works.
- [ ] Tier A commands pass.

## Next

`run substep 5.5` — HomeScreen + card tap + final gate.
