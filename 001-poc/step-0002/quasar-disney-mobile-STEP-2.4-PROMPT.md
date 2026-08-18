# quasar-disney-mobile — STEP-2.4: Theme, fonts, i18n, outlined wordmarks

> **How to run:** Tell your agent *"run substep 2.4"*. Self-contained — executable cold in a fresh chat.

## Context

Tooling and `createStore()` exist (2.3). This substep lands the visual kernel: Emotion theme (both brands × both surface modes), bundled Inter, forced `es-419` i18n, and outlined brand SVGs so iOS and Android render the same letterforms (**OQ-13** / **RISK-0006**).

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-2-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-2-PLAN.md`
- `Code/quasar-disney-mobile-docs/architecture/07-ui-design-system.md` (tokens §2, theme §5, stack §12, i18n §9, Inter §2.4)
- `Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md` §3.4 (token-parity test)
- `Code/quasar-disney-mobile-docs/adr/ADR-0011-*.md`, `ADR-0012-*.md`, `ADR-0013-*.md`, `ADR-0019-*.md`
- `Code/quasar-disney-mobile-docs/architecture/assets/README.md` + files under `architecture/assets/brand/`
- `Code/quasar-disney-mobile-docs/architecture/15-native-app-architecture.md` (svg dependency)
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`
- `Code/quasar-disney-mobile-app/README.md`

## Scope

**Owns:** `src/shared/theme/**`, Inter font linking, `src/shared/i18n/**`, copy of outlined brand assets into the app repo, token-parity test, `ThemeProvider` wrap at the app root.

**Does not:** atoms/icons (2.5), navigators/overlay (2.6), long-press theme swap on Welcome (STEP-4; tokens for `ember` must exist now), storefront artwork (can copy `placeholder-art/` into the app now or leave a note for STEP-5 — copying the SVGs is in scope if cheap, not required for this substep's tests).

## Your task

1. On `step-0002-app-scaffold`.
2. **Dependencies:** `@emotion/native`, `@emotion/react`, `react-native-svg`, `react-i18next`, `i18next`. No `styled-components`.
3. **Theme layout** (doc 07 §12):

   `src/shared/theme/{tokens/, themes/dinsey.ts, themes/ember.ts, ModeProvider.tsx, types.ts, emotion.d.ts}`

   - Augment `@emotion/react` `Theme` so styled templates are not `any`.
   - `Theme.name`: `'dinsey' | 'ember'`. Modes: `'app' | 'auth'` — **never** `light`/`dark`. **Never call `useColorScheme()`**.
   - `ModeProvider` flattens `modes[mode]` onto the theme (snippet in doc 07 §12).
   - Token values: copy doc 07 §2 exactly for `dinsey`. `ember` values are specified in §5 (warm graphite `#14100C`, amber `#FBBF24`, gradient `#4A2C10`, link `#B45309`); fill remaining keys so **key structure matches `dinsey` bit-for-bit** (that is the A1 test). Contrast-check was done in session 1.7; do not invent a third brand.
   - Type, space, radius, motion are mode-independent (doc 07 §5).
   - Default export: `dinsey` with a way for STEP-4 to switch `name` later (module-level current theme is fine; no Redux).
4. **Inter:** ship `Inter-Regular`, `SemiBold`, `Bold`, `ExtraBold` as named files. Link via `react-native.config.js` + `npx react-native-asset`. Do not rely on numeric `fontWeight` on Android (doc 07 §2.4).
5. **i18n:** `react-i18next`, semantic keys, **forced `es-419`**, no device-locale detection. RTL-safe layout props (`marginStart` / `paddingEnd`) from here on. Seed the string table with keys this STEP and 2.6 need, including:
   - `errorBoundary.message` = *Algo salió mal.*
   - `errorBoundary.retry` = *REINTENTAR*
   - `offline.message` = the doc 15 §2 Spanish paragraph
   - `offline.retry` = *REINTENTAR*
   - `comingSoon.title` (one string for three tabs)
   - `loading.gate` = *Cargando*
   Auth/storefront copy can wait for STEPs 4–5, but adding empty namespaces is fine.
6. **Wordmarks (OQ-13 / RISK-0006):** copy `architecture/assets/brand/` into the app (e.g. `src/shared/assets/brand/`). Convert SVG `<text>` to **outlines/paths** so letterforms do not depend on Avenir Next. Prefer outlined SVG over PNG. Tools: Inkscape (`--export-text-to-path` / object-to-path) or equivalent; do not leave `<text>` in the shipped files. Keep geometry. Close RISK-0006 when the four brand files are outlined (wordmarks light/dark, mark, strip).
7. **Token-parity test** (`src/shared/theme/themeParity.test.ts` or similar): assert `dinsey` and `ember`, modes `app` and `auth`, have **identical key structure** (recursive key sets). This is criterion A1's structural half (doc 12 §3.4). No renderer.
8. Wrap the app root with Emotion `ThemeProvider` (default `dinsey`) so 2.5 atoms can consume `theme`. `ModeProvider` can default to `auth` until 2.6 switches by navigator.
9. TSDoc on exported providers and token builders.

## Verification

- Run **now:** `npx tsc --noEmit && npm test && npx eslint . && npx prettier --check .`
- Token-parity test green for both brands × both modes.
- ESLint A2 rule: a hex in `src/app/` still fails; hexes inside `src/shared/theme/` pass.
- Grep the copied brand SVGs: no `<text` remaining (or only in comments).
- iOS still boots (font linking often breaks Android/iOS differently — smoke both if Android was not dropped).

## Keeping the docs true  (always)

Close **OQ-13** in `architecture/02-phasing-roadmap.md` and `architecture/07-ui-design-system.md` open-question tables (resolved: outlined in STEP-2.4). Close **RISK-0006** in `registries/risks.yml` (`status: closed`, date, reason). Bump Version Logs. If asset paths in `architecture/assets/README.md` should point at the app copy as the runtime source, add one sentence — the hub files remain provenance.

## Definition of done

- [ ] Emotion theme + ModeProvider; `dinsey` + `ember`; modes `app` + `auth`; no `useColorScheme()`.
- [ ] Inter linked on both platforms still in play.
- [ ] i18n forced `es-419` with the seed keys above.
- [ ] Brand SVGs outlined; RISK-0006 and OQ-13 closed.
- [ ] Token-parity T1 test green.
- [ ] Final-ish gate green for this substep: tsc + jest + eslint + prettier.
- [ ] TSDoc on new exports.

## Next

Mark 2.4 Done in the PLAN and index. **Fresh chat:** `run substep 2.5`.
