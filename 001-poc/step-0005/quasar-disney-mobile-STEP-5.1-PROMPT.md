# quasar-disney-mobile — STEP-5.1: Feed endpoints + pagination hooks

> **How to run:** Tell your agent *"run substep 5.1"*. Self-contained — executable cold in a fresh chat.

## Context

STEP-3 merged mock handlers for the three feed operations; STEP-4 merged auth. The storefront feature folder still has placeholder screens only. This substep injects RTK Query endpoints and the pagination wrapper hooks every later substep builds on.

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-5-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-5-PLAN.md`
- `Code/quasar-disney-mobile-docs/architecture/03-architecture-overview.md` §6 Flow 2, §8.1
- `Code/quasar-disney-mobile-docs/architecture/11-interface-contracts.md` §5–§7 (operations 3–5)
- `Code/quasar-disney-mobile-docs/architecture/15-native-app-architecture.md` §8
- `Code/quasar-disney-mobile-docs/adr/ADR-0005-storefront-pagination-hooks.md`
- `Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md` §3.2
- `Code/quasar-disney-mobile-app/src/api/baseApi.ts`, `src/api/types/`, `src/features/auth/api.ts` (inject pattern)
- `Code/quasar-disney-mobile-app/src/api/client/paths.ts`
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`
- `Code/quasar-disney-mobile-app/README.md`

## Scope

**Owns:** `src/features/storefront/api.ts`, pagination hooks under `src/features/storefront/hooks/`, barrel exports, T1 hook tests.

**Does not:** UI components (5.2+), composed-home merge (5.4), shell gate changes (STEP-6), mock handler changes (STEP-3).

## Your task

1. On branch `step-0005-storefront-feature` from `main`.
2. **`features/storefront/api.ts`** — `storefrontApi = baseApi.injectEndpoints({...})`:
   - `getHomeFeed` query — `GET /home-feed`, args `{ cursor?: string }`, returns `Page<Container>`.
   - `getContinueWatching` query — `GET /continue-watching`, same paging args/shape.
   - `getContainerResources` query — `GET /containers/{containerId}/resources`, args `{ containerId, cursor?: string }`, returns `Page<Card>`.
   - Export hooks: `useGetHomeFeedQuery`, `useLazyGetHomeFeedQuery`, `useGetContinueWatchingQuery`, `useLazyGetContinueWatchingQuery`, `useGetContainerResourcesQuery`, `useLazyGetContainerResourcesQuery`.
   - Use `serializeQueryArgs` / `merge` / `forceRefetch` or manual page accumulation in hooks — **hooks own the merged page**, not the generated query alone (ADR-0005).
3. **`usePaginatedContainers`** — wraps a lazy container query (HomeFeed or CW):
   - Returns `{ containers, loadMore, hasMore, isLoading, isFetching, error }`.
   - Accumulates `data` arrays across fetches; `hasMore` derives from last page's `nextCursor !== null`.
   - `loadMore` no-ops when `!hasMore` or a fetch is in flight.
4. **`useCarouselPage(containerId)`** — wraps `getContainerResources`:
   - Same surface: `{ cards, loadMore, hasMore, isLoading, isFetching, error }`.
5. **T1 tests** (`src/features/storefront/hooks/*.test.ts`):
   - Use `t2World.factory` pattern or a storefront-local test store + injected endpoints.
   - Prove vertical merge: two pages append; `hasMore` false when `nextCursor` is null.
   - Prove horizontal merge for `useCarouselPage`.
   - Trust `nextCursor` — do not infer exhaustion from item counts alone (doc 12 §3.2).

## Verification

- Run **now:** `npx tsc --noEmit && CI=true npm test -- storefront/hooks && npx eslint src/features/storefront && npx prettier --check src/features/storefront`
- Import rules: no auth imports; no direct axios in feature code.

## Keeping the docs true  (always)

If hook names differ from doc 15 §8 illustrations, that's fine — export names must be stable and documented in the feature README. Do not persist RTK Query cache.

## Definition of done

- [ ] Three feed endpoints injected on `baseApi`.
- [ ] `usePaginatedContainers` and `useCarouselPage` exported with ADR-0005 surface.
- [ ] T1 pagination hook tests green.
- [ ] TSDoc on public hooks.
- [ ] Tier A commands pass for touched paths.

## Next

`run substep 5.2` — artwork resolver + tile molecules.
