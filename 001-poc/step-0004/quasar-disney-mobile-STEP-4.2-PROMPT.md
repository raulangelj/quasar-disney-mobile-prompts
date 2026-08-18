# quasar-disney-mobile — STEP-4.2: Auth molecules + i18n

> **How to run:** Tell your agent *"run substep 4.2"*. Self-contained — executable cold in a fresh chat.

## Context

4.1 (or parallel if UI-only first) provides the data layer. This substep builds the reusable auth UI building blocks and Spanish copy so 4.3–4.4 assemble screens without duplicating layout logic.

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-4-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-4-PLAN.md`
- `Code/quasar-disney-mobile-docs/architecture/07-ui-design-system.md` §3.2 (molecules), §3.4 (submit spinner), §7 (safe areas), §9 (i18n)
- `Code/quasar-disney-mobile-docs/inputs/ui/disney-plus-reference-screens.md` §1–§4 (adapt copy to Dinsey-)
- `Code/quasar-disney-mobile-app/src/shared/ui/atoms/` (Button, Text, TextField, PasswordField, Link, Wordmark)
- `Code/quasar-disney-mobile-app/src/shared/i18n/locales/es-419.json`
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`

## Scope

**Owns:** `src/features/auth/ui/AuthSheetLayout.tsx`, `WelcomeHero.tsx`, `CredentialsForm.tsx` (or equivalent names per doc 07 §3.2); auth i18n keys in `es-419.json`; barrel exports.

**Does not:** full screen wiring / navigation (4.3–4.4), API calls, theme swap handler (4.3 Welcome), T3 snapshot tests.

## Your task

1. On `step-0004-auth-feature`.
2. **`AuthSheetLayout`** — white sheet over gradient top third (doc 07 / reference §2):
   - Props: `children`, optional `onBack` (renders chevron in circular translucent control), safe-area aware (top inset on gradient region, sheet padding from `theme.space.authSheetPadding`).
   - Gradient uses `theme.colors.gradient.top/bottom` in `auth` mode.
   - Centered Dinsey wordmark in gradient header when no back button; with back, wordmark centered between edges.
3. **`WelcomeHero`** — collage + headline region for Welcome (reference §1, **Dinsey-** assets only):
   - Staggered placeholder tiles using existing placeholder-art or simple colored rects with glow — no Disney marks.
   - `brand-strip.svg` for the brand row (already outlined in repo).
   - Props for headline, caption, CTA slot (screen passes `Button`).
4. **`CredentialsForm`** — shared email/password form chrome (reference §2–§4):
   - MyDisney-style sub-wordmark → use **Dinsey** wordmark variant appropriate for light sheet.
   - Heading, body, field slot(s), primary CTA slot, optional footer/legal block.
   - Support **error state** on password field: red 2px bottom border, error `Text` below field, hint below error (doc 07 §3.4 / reference §4).
   - Submit loading: CTA shows `Spinner` inside button, label hidden, width held (doc 07 §3.4).
5. **i18n** — add semantic keys under `auth.*` for Welcome, EmailEntry, PasswordEntry, validation messages, F2 long-form error prose (reference §4 — adapt text, keep multi-line), case-sensitivity hint, inert link labels. **Forced `es-419` only.**
6. **a11y:** `accessibilityLabel` on back button, CTA, password toggle; error text associated with field (`accessibilityLiveRegion` on error where supported).

## Verification

- Run: `npx tsc --noEmit && npx eslint src/features/auth && npx prettier --check src/features/auth src/shared/i18n`
- **No new tests required** if molecules are presentational only — reason: no testable logic without screens (T3 deferred RISK-0001). Optional pure validation helper tests if extracted.
- Storybook not in repo — verify by importing into a throwaway screen or wait for 4.3.

## Keeping the docs true  (always)

Do not add Disney trademark strings. If molecule file paths differ from doc 07 §3.2 table, bump doc 07 Version Log only if the table is normative paths — otherwise note in PR description.

## Definition of done

- [ ] Three molecules exported from `features/auth/ui/`.
- [ ] i18n keys cover all auth copy needed by 4.3–4.4.
- [ ] a11y props on interactive elements.
- [ ] TSDoc on exported components.
- [ ] Lint/format clean.

## Next

`run substep 4.3` — Welcome + EmailEntry screens.
