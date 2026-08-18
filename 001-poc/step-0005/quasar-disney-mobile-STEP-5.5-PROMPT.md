# quasar-disney-mobile — STEP-5.5: HomeScreen + card tap + final gate

> **How to run:** Tell your agent *"run substep 5.5"*. Self-contained — executable cold in a fresh chat.

## Context

Feed composition, carousel rows, and shell chrome exist (5.1–5.4). This substep replaces the placeholder `HomeScreen`, wires card tap → Alert (**F3**), adds remaining T2 paging tests, runs the STEP gate, and prepares for review.

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-5-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-5-PLAN.md`
- `Code/quasar-disney-mobile-docs/architecture/03-architecture-overview.md` §6 Flow 2 step 4
- `Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md` §3.2
- `Code/quasar-disney-mobile-app/src/features/storefront/ui/HomeScreen.tsx`
- `Code/quasar-disney-mobile-app/src/api/integration/pagination.test.ts` (pattern)
- `Code/quasar-disney-mobile-docs/runbooks/release-deploy.md` (smoke context — full smoke is STEP-6)

## Scope

**Owns:** Real `HomeScreen`, `onCardPress` handler → `Alert.alert(title)`, shared `ErrorState` molecule if not yet built, T2 both-axis paging tests for storefront endpoints, manual F3 smoke, STEP-5 review prep.

**Does not:** three-query LoadingGate (STEP-6), theme-swap verification (STEP-6), release binary (STEP-6), T3 UI tests (STEP-8).

## Your task

1. On `step-0005-storefront-feature`.
2. **`HomeScreen`** — compose `HomeFeedList` + `useComposedHome` + silent reload hook:
   - Stable `onCardPress` callback: `Alert.alert(card.title)` (doc 03 / DF7)
   - Feed-level error → `ErrorState` + retry (refetch both feeds)
   - Remove placeholder "Home" text
3. **`ErrorState`** — if missing from shared molecules, add per doc 07 §3.2 (retry button, i18n)
4. **T2 paging tests** (`src/features/storefront/` or `src/api/integration/`):
   - Vertical: exhaust HomeFeed `nextCursor` through real `getHomeFeed` injected endpoints + mock adapter
   - Horizontal: exhaust `getContainerResources` cursors for a short row (reuse pagination.test patterns)
   - Assert client trusts `nextCursor` over counts
5. **Manual smoke (F3):**
   - Login with demo creds → home shows hero + CW + portrait rows
   - Scroll a row to end → more tiles load
   - Tap a tile → Alert shows title
6. **STEP gate:** `npx tsc --noEmit && CI=true npm test && npx eslint . && npx prettier --check .`
7. Update `src/features/storefront/README.md` with feature overview and hook names.
8. Open app PR; update `prompts/STEP-index.md` substeps to Done as you complete review steps.

## Verification

- Run **now (full STEP gate):** `npx tsc --noEmit && CI=true npm test && npx eslint . && npx prettier --check .`
- Manual F3 on iOS simulator (Android if available).

## Keeping the docs true  (always)

Bump doc 03 Version Log if `HomeScreen` wiring clarifies Flow 2. Do not claim three-query boot gate works — STEP-6.

## Definition of done

- [ ] Placeholder home removed; real feed visible after login.
- [ ] Card tap shows title alert (F3).
- [ ] T2 both-axis paging tests green.
- [ ] Full Tier A gate green.
- [ ] Feature README updated.
- [ ] Ready for STEP review / PR.

## Next

STEP review (PR + doc drift), then archive to `prompts/001-poc/step-0005/`. After merge, next action is **plan STEP-6** (or `run substep 6.1` once STEP-6 is planned).
