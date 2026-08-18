# quasar-disney-mobile — STEP-4.3: Welcome + EmailEntry screens

> **How to run:** Tell your agent *"run substep 4.3"*. Self-contained — executable cold in a fresh chat.

## Context

Molecules and i18n exist (4.2). Auth slice/endpoints exist (4.1). This substep replaces the Welcome and EmailEntry **placeholders** with real screens and wires navigation between them.

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-4-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-4-PLAN.md`
- `Code/quasar-disney-mobile-docs/architecture/07-ui-design-system.md` §4 (navigation, back rules), §5 (theme swap on Welcome)
- `Code/quasar-disney-mobile-docs/inputs/ui/disney-plus-reference-screens.md` §1–§2
- `Code/quasar-disney-mobile-app/src/app/navigation/AuthNavigator.tsx`, `types.ts`
- `Code/quasar-disney-mobile-app/src/features/auth/ui/screens.tsx` (replace)
- `Code/quasar-disney-mobile-app/src/shared/theme/themeRegistry.ts` (theme switch for long-press)
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`

## Scope

**Owns:** `WelcomeScreen`, `EmailEntryScreen`; update `screens.tsx`; navigation params (`email` passed to PasswordEntry route type); email format validation (client-side UX only).

**Does not:** PasswordEntry / login (4.4), session restore (4.5), `getMe`, mock adapter.

## Your task

1. On `step-0004-auth-feature`.
2. **Navigation types** — extend `AuthStackParamList` so `PasswordEntry` receives `{ email: string }`.
3. **`WelcomeScreen`**:
   - Full-bleed gradient + `WelcomeHero` + white pill CTA (`INICIAR SESIÓN` i18n).
   - CTA navigates to `EmailEntry`.
   - **Long-press** Dinsey wordmark cycles theme `dinsey` ↔ `ember` via `setActiveTheme` / registry from 2.4 (doc 07 §5). No Redux.
   - Footer copyright lines (generic Dinsey- placeholder text — no Disney legal copy).
4. **`EmailEntryScreen`**:
   - `AuthSheetLayout` with back → pop to Welcome.
   - `CredentialsForm` with email `TextField`, `Continuar` CTA.
   - Validate email format (simple regex or `TextField` rules); disable or show inline hint if empty/invalid.
   - On success navigate to `PasswordEntry` with `{ email }`.
5. Remove placeholder components for these two routes.
6. **Back navigation:** chevron, iOS edge-swipe, Android back all pop (React Navigation default + doc 07 §4).
7. Manual check: boot unauthenticated → Welcome visible → CTA → Email → back → Welcome.

## Verification

- Run: `npx tsc --noEmit && npx eslint src/features/auth src/app/navigation && npx prettier --check src/features/auth src/app/navigation`
- No automated tests required (T3 deferred; no business logic beyond validation — optional unit test for email validator if extracted).
- Import rules unchanged.

## Keeping the docs true  (always)

Theme swap is dev affordance only — do not document as user setting.

## Definition of done

- [x] Welcome and EmailEntry render with auth mode tokens.
- [x] Navigation Welcome ↔ Email works; params typed for PasswordEntry.
- [x] Long-press wordmark swaps theme.
- [x] a11y labels on CTAs and back control.
- [x] Lint/format clean.

## Next

`run substep 4.4` — PasswordEntry + F2 inline error + login T2 tests.
