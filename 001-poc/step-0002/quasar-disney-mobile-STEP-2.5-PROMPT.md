# quasar-disney-mobile — STEP-2.5: Shared atoms & icons

> **How to run:** Tell your agent *"run substep 2.5"*. Self-contained — executable cold in a fresh chat.

## Context

Theme, fonts, and i18n exist (2.4). This substep builds the **closed** shared-atom list and the 13 SVG icons so STEPs 4–5 do not invent parallel buttons. T3 render tests are **deferred** (doc 12 §2, RISK-0001) — write a11y props now anyway (RISK-0011).

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-2-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-2-PLAN.md`
- `Code/quasar-disney-mobile-docs/architecture/07-ui-design-system.md` §3.1 (atoms), §3.2 molecules that are **not** this substep's required set, §6 icons, §8 a11y, §10 motion / reduced motion
- `Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md` §2 (T3 deferred), §3.5
- `Code/quasar-disney-mobile-docs/registries/risks.yml` (RISK-0001, RISK-0011)
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md` (PascalCase `.tsx` for components)
- `Code/quasar-disney-mobile-app/README.md`
- `inputs/ui/disney-plus-reference-screens.md` as a visual cross-check, not as living truth where doc 07 disagrees

## Scope

**Owns:** `src/shared/ui/atoms/**` and `src/shared/ui/icons/**` per doc 07. A `useReducedMotion()` hook in `shared/` (components must not query `AccessibilityInfo` directly).

**Does not:** feature tiles (`PortraitTile`, etc. — STEP-5), auth sheet molecules (STEP-4), shell organisms (`AppHeader`, `TabBar`, `NoInternetOverlay`, `LoadingGate` — 2.6), navigation, T3 tests.

Optional if cheap and used by 2.6: molecules `SectionHeader`, `TabBarItem`, `ErrorState`, `EmptyState` in `shared/ui/molecules`. Prefer building those in **2.6** with the shell if that keeps this substep focused on the §3.1 table.

## Your task

1. On `step-0002-app-scaffold`.
2. Implement **every atom in doc 07 §3.1**:

   | Component | Notes |
   |-----------|--------|
   | `Text` | One variant per type token |
   | `Button` | `tone: onDark \| onLight`; `variant: solid \| ghost`; `size: md(48) \| sm(36)`; loading / disabled / pressed. Tone is an **explicit prop**, not derived from mode |
   | `IconButton` | Circular translucent; 44 pt `hitSlop` |
   | `Chip` | Rating copy |
   | `Badge` | `tone: live \| label \| provider` |
   | `ProgressBar` | `tone: watched \| live`; `value 0..1`; track always rendered |
   | `TextField` | empty / focused (2 px ring) / filled / error (2 px bottom underline + message). Error uses `accessibilityLiveRegion="assertive"` |
   | `PasswordField` | secure / revealed; label changes (*Mostrar / Ocultar contraseña*) |
   | `Link` | `tone: onDark \| onLight` |
   | `Divider`, `Spinner` (26/16), `Skeleton` (takes tile ratio) | Skeleton respects reduced motion (static fill) |
   | `Artwork` | `ratio: 2:3 \| 16:9 \| 3:4`; skeleton fallback; decorative = hidden from a11y |
   | `Wordmark` | `tone: light \| dark` — uses outlined SVGs from 2.4 |
   | `FilterPill` | `form: logo \| iconLabel`; `selected` (built now, used in Phase 2) |

3. **Icons** (doc 07 §6), `src/shared/ui/icons/`: cast, download, home filled + outline, bolt, search, profile, chevron-left, chevron-down, eye, eye-off, overflow, play. 24 pt grid, ~1.7 stroke, `color` from theme tokens. Hand-authored SVG components over `react-native-svg`. No `react-native-vector-icons`.
4. Styled via `@emotion/native`. Colors/spacing from theme tokens only (A2). Press feedback: tiles will scale later; buttons/pills/links/icons use opacity 0.7 over `motion.instant` (doc 07 §10). `useNativeDriver: true`.
5. `maxFontSizeMultiplier` caps: 1.6 body+, 1.3 caption, 1.2 micro (doc 07 §7).
6. TSDoc on every component. Filename `PascalCase.tsx`.
7. **Do not add T3 / RNTL render tests.** Record that in the PR/commit: RISK-0001. Do not add snapshot tests of Emotion output (doc 12 §3.5).

## Verification

- **Tests:** none for UI render. Reason: T3 deferred to STEP-8 / Phase 1b (RISK-0001). Typecheck and lint still run.
- Run **now:** `npx tsc --noEmit && npm test && npx eslint . && npx prettier --check .`
- Manual spot-check: temporarily render `Button` + `Text` + `Wordmark` on the hello screen (or a Story-less debug block behind `__DEV__`) to confirm tokens apply; remove the debug block before finishing if it would ship as a fake screen. 2.6 will compose real shell UI.
- Confirm no hex/`rgba(` outside `src/shared/theme/`.

## Keeping the docs true  (always)

If an atom API must differ from doc 07, update doc 07 and bump its Version Log — do not silently drift. RISK-0001 and RISK-0011 stay open.

## Definition of done

- [ ] All §3.1 atoms exist and consume theme tokens.
- [ ] 13 SVG icons exist and tint from tokens.
- [ ] A11y props on TextField error, PasswordField, IconButton hitSlop, Artwork hidden when decorative, Wordmark.
- [ ] `useReducedMotion()` hook; Skeleton/press honor it.
- [ ] No T3 tests (reason recorded: RISK-0001).
- [ ] tsc + existing jest + eslint + prettier green.
- [ ] TSDoc on components.

## Next

Mark 2.5 Done in the PLAN and index. **Fresh chat:** `run substep 2.6`.
