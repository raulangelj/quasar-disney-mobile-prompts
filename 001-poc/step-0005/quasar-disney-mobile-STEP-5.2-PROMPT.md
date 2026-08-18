# quasar-disney-mobile — STEP-5.2: Artwork resolver + tile molecules

> **How to run:** Tell your agent *"run substep 5.2"*. Self-contained — executable cold in a fresh chat.

## Context

Pagination hooks exist (5.1). Fixtures reference placeholder art by **filename string** on the wire (STEP-3 Q4). This substep adds Metro SVG support, the filename → asset resolver, shared `SectionHeader`, and the two Phase-1a tile molecules.

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-5-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-5-PLAN.md`
- `Code/quasar-disney-mobile-docs/architecture/07-ui-design-system.md` §3.1–§3.2, §7–§8
- `Code/quasar-disney-mobile-docs/architecture/04-data-model.md` §1.2–§1.3
- `Code/quasar-disney-mobile-app/src/api/mocks/fixtures/artwork.ts`
- `Code/quasar-disney-mobile-app/src/shared/ui/atoms/Artwork.tsx`
- `Code/quasar-disney-mobile-app/src/shared/ui/icons/PlayIcon.tsx`
- `inputs/ui/disney-plus-reference-screens.md` §5–§6
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`

## Scope

**Owns:** Metro SVG transformer config, `features/storefront/ui/placeholderArt.ts`, `shared/ui/molecules/SectionHeader.tsx`, `features/storefront/ui/PortraitTile.tsx`, `features/storefront/ui/ProgressTile.tsx`, storefront i18n namespace (`storefront.*`), T1 i18n test for `remainingMinutes`.

**Does not:** carousel row organism (5.3), feed hooks (5.1), `AppHeader` (5.4).

## Your task

1. On `step-0005-storefront-feature`.
2. **Metro SVG transformer:**
   - Add `react-native-svg-transformer` dev dependency.
   - Update `metro.config.js` so `.svg` imports work as React components or image sources compatible with `Artwork`.
   - Add TypeScript module declaration for `*.svg` if needed.
3. **`placeholderArt.ts`** — `resolvePlaceholderArt(filename: string): ImageSourcePropType | undefined`:
   - Map the seven known filenames in `src/shared/assets/placeholder-art/` via static `require()`.
   - Unknown filenames → `undefined` (Artwork shows skeleton).
4. **`SectionHeader`** in `shared/ui/molecules/` — renders `Container.name` verbatim with `accessibilityRole="header"` (doc 07 §8).
5. **`PortraitTile`** — 2:3 art via `Artwork`; uses `resolvePlaceholderArt(card.artwork['2:3'])`; optional `Chip` for rating; pressable with token'd press animation; decorative art hidden from a11y tree.
6. **`ProgressTile`** — 16:9 art, centered `PlayIcon`, `ProgressBar` with `card.progress`, metadata block (`title`, `episodeLine`, rating chip). **Accessibility label** from i18n template including `remainingMinutes` as a number passed into the template — never concatenate hardcoded Spanish in TS (DF8).
7. **i18n** — add `storefront.progressLabel`, `storefront.remainingMinutes`, etc. under `es-419`.
8. **T1 test** — pure function or i18n test proving `remainingMinutes` flows through the translation template.

## Verification

- Run **now:** `npx tsc --noEmit && CI=true npm test -- PortraitTile ProgressTile placeholderArt SectionHeader && npx eslint src/features/storefront src/shared/ui/molecules && npx prettier --check src/features/storefront src/shared/ui/molecules metro.config.js`
- No T3 snapshot tests (RISK-0001).

## Keeping the docs true  (always)

If Metro config path differs from doc 03 §8.1 note, bump doc 03 Version Log. Do not put `require()` handles in fixtures.

## Definition of done

- [ ] SVG placeholder art renders in `PortraitTile` / `ProgressTile`.
- [ ] `SectionHeader` exported from shared molecules barrel.
- [ ] CW label uses i18n for minutes phrase (T1).
- [ ] a11y props on tiles per doc 07 §8 (grouped label on ProgressTile).
- [ ] Tier A commands pass.

## Next

`run substep 5.3` — config-driven carousel organism.
