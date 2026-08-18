# quasar-disney-mobile — STEP-5.3: Config-driven carousel + variant map

> **How to run:** Tell your agent *"run substep 5.3"*. Self-contained — executable cold in a fresh chat.

## Context

Hooks (5.1) and tile molecules (5.2) exist. Architecture requires **one** carousel organism for every variant (DF6 / ADR-0007), not per-variant components. This substep builds `CarouselRow`, the variant → layout config, and unknown-variant handling.

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-5-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-5-PLAN.md`
- `Code/quasar-disney-mobile-docs/architecture/07-ui-design-system.md` §3.2–§3.3, §7 (visible-count formula)
- `Code/quasar-disney-mobile-docs/architecture/11-interface-contracts.md` §4.4
- `Code/quasar-disney-mobile-app/src/features/storefront/hooks/`
- `Code/quasar-disney-mobile-app/src/features/storefront/ui/PortraitTile.tsx`, `ProgressTile.tsx`
- `Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md` §3.2 (unknown variant row)
- `Code/quasar-disney-mobile-app/src/api/integration/consoleObservability.test.ts` (warn seam)

## Scope

**Owns:** `features/storefront/ui/variantConfig.ts`, `features/storefront/ui/CarouselRow.tsx`, variant-mapping pure function + tests, horizontal `FlatList` with `onEndReached` → `loadMore`.

**Does not:** vertical home list (5.4), composed-home merge (5.4), `HomeScreen` (5.5), `'live'` / `'standardLandscape'` layouts (STEP-7).

## Your task

1. On `step-0005-storefront-feature`.
2. **`variantConfig.ts`** — per `ContainerVariant` in 1a:
   - `visibleCount` (doc 07 §7: hero ~1.15, portrait ~3.6, progress ~1.9 or per reference)
   - `aspectRatio` / tile component selector (`PortraitTile` vs `ProgressTile`)
   - `'hero'` uses 3:4 ratio stand-in (large portrait), not Phase-2 chrome
   - `'standardLandscape'` and `'live'` → **unknown for 1a** (handled by drop rule below) OR map to portrait stand-in only if you must render — **prefer drop + warn** for anything not in the 1a set
3. **`mapContainerToRowProps(container)`** — pure function:
   - Known 1a variants → config + first page of cards from `container.resources`
   - Unknown variant → `null` + `console.warn` with container id/variant (doc 11 §4.4); **no throw, no default layout**
4. **`CarouselRow`** organism:
   - Props: `container`, `onCardPress(card)`, uses `useCarouselPage(container.id)` for horizontal paging
   - Horizontal `FlatList` with computed tile width from visible-count formula
   - Trailing skeleton tile while `hasMore` (doc 07 §3.4)
   - `onEndReached` calls `loadMore`
   - Hero row: same organism, different visible-count / tile sizing
5. **T1 tests:**
   - `mapContainerToRowProps` drops `'live'` / garbage variant and warns (mock `console.warn`)
   - Known variants return non-null config
   - Optionally test tile width helper with a fixed screen width

## Verification

- Run **now:** `npx tsc --noEmit && CI=true npm test -- variantConfig CarouselRow mapContainer && npx eslint src/features/storefront && npx prettier --check src/features/storefront`
- Ensure existing `consoleObservability.test.ts` still passes.

## Keeping the docs true  (always)

Do not add `'live'` to the data model — that's STEP-7. Document any visible-count constants in code comments referencing doc 07.

## Definition of done

- [x] Single `CarouselRow` drives hero, progress, and standardPortrait.
- [x] Unknown variants dropped with warn (T1).
- [x] Horizontal paging wired to `useCarouselPage`.
- [x] Tier A commands pass.

## Next

`run substep 5.4` — composed home + silent CW + AppHeader.
