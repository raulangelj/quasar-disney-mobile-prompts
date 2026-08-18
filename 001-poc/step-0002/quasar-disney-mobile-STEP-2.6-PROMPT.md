# quasar-disney-mobile — STEP-2.6: Navigation shell, overlay, error boundary

> **How to run:** Tell your agent *"run substep 2.6"*. Self-contained — executable cold in a fresh chat.

## Context

Atoms and theme exist (2.5). This substep is the composition root the features plug into: two navigators, ComingSoon tabs, NetInfo overlay, LoadingGate stub, root error boundary (**OQ-33** closed in the PLAN). Auth and storefront screens stay placeholders — STEPs 4 and 5 replace them.

This is the last STEP-2 substep. After it, run the STEP test gate, then the STEP review (PR), then archive.

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-2-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-2-PLAN.md` (OQ-33 resolution, definition of done)
- `Code/quasar-disney-mobile-docs/architecture/07-ui-design-system.md` §3.2–§3.3, §4 navigation, §5 ModeProvider at navigator boundary, §7 safe areas, §8 overlay a11y, §10 overlay motion, §13 platform conventions
- `Code/quasar-disney-mobile-docs/architecture/03-architecture-overview.md` §4, §6 Flow 1 (overlay sits on top; does not unmount)
- `Code/quasar-disney-mobile-docs/architecture/15-native-app-architecture.md` §2 (NetInfo, ADR-0014: `isConnected` only, no reachability probe)
- `Code/quasar-disney-mobile-docs/architecture/10-observability.md` §5.2 (error boundary)
- `Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md` §3.5 (do not test navigation config)
- `Code/quasar-disney-mobile-docs/adr/ADR-0004-*.md`, `ADR-0014-*.md`
- `inputs/ui/disney-plus-reference-screens.md` §7 (no-internet reference)
- `Code/quasar-disney-mobile-app/README.md`
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`

## Scope

**Owns:** `src/app/` composition: `Provider` + persist gate + `ThemeProvider` + error boundary + `NavigationContainer` + `SafeAreaProvider` + NetInfo overlay + LoadingGate; Auth stack placeholders; App tabs with Home placeholder + three ComingSoon routes; molecules needed by the shell (`TabBarItem`, `ErrorState` if used).

**Does not:** real Welcome/email/password (STEP-4), real Home feed (STEP-5), `getMe` boot gate queries (needs STEP-3 `baseApi`), long-press `ember` swap (STEP-4 on Wordmark), release-build smoke (STEP-6), T3 tests.

## Your task

1. On `step-0002-app-scaffold`.
2. **Dependencies:** `@react-navigation/native`, `@react-navigation/native-stack`, `@react-navigation/bottom-tabs`, `react-native-screens`, `react-native-safe-area-context` (already required by doc 03 §7).
3. **Root tree** (inside out):
   - `createStore()` once at the real entry (returns `{ store }` only — call `persistStore(store)` here). `Provider` is already wrapping `App.tsx` from 2.3; add persist `PersistGate`.
   - Emotion `ThemeProvider` (`dinsey`).
   - **Root error boundary** inside the theme provider (doc 10 §5.2). Fallback: Wordmark + Text + Button per PLAN OQ-33; i18n keys from 2.4; retry remounts via reset key; `console.error` the error in `__DEV__` only; **ships in both configurations**. Close **OQ-33** in `architecture/10-observability.md`.
   - `SafeAreaProvider` + `NavigationContainer`.
   - `StatusBar` `light-content` both modes (doc 07 §13).
4. **Navigators** (doc 07 §4):

```
RootNavigator                       switches on stub selectIsAuthenticated (false until STEP-4)
├─ AuthNavigator      stack · ModeProvider mode=auth · headerShown: false
│   ├─ Welcome        placeholder
│   ├─ EmailEntry     placeholder
│   └─ PasswordEntry  placeholder
└─ AppTabsNavigator   tabs  · ModeProvider mode=app  · custom tab bar
    ├─ Home           placeholder
    ├─ Novedades / Buscar / Perfil → ComingSoon
NoInternetOverlay · LoadingGate — shell-level, outside both navigators (siblings, not unmounting them)
```

   Mode is fixed at the navigator boundary via `ModeProvider`. No Redux flag for mode.

5. **ComingSoon:** one screen, three routes, one i18n string. Tappable tabs (inert ≠ dead tap).
6. **Custom tab bar** using React Navigation's `tabBar` slot. Unlabeled icons; `accessibilityLabel` + `accessibilityState.selected`. Safe-area inset applied **here**, in `AppHeader` (even if placeholder), and later AuthSheet — not per screen.
7. **NoInternetOverlay:** NetInfo `isConnected` only (**ADR-0014**). Dark `app`-family surface even over auth. Copy from i18n (`offline.*`). White pill `REINTENTAR`. Auto-hide on reconnect; `REINTENTAR` runs the same restore path. `accessibilityViewIsModal`. Fade `motion.quick`. Does not unmount the navigator underneath.
8. **LoadingGate:** Wordmark + Spinner; i18n `loading.gate`. For this STEP it can stay **hidden** (no three-query gate until STEP-3/4). Export it so STEP-4/6 can flip it on. Do not fake a 3s splash.
9. **Feature placeholders** live under `src/features/auth/` and `src/features/storefront/` as empty screens so the shell imports features (composition root). Shared still must not import features.
10. **Analytics stub:** `src/shared/analytics/` hook that `console.log`s in `__DEV__` only. No vendor.
11. Close OQ-33 in architecture docs; bump Version Logs. Update app README run instructions (entry is now the shell).
12. Manual smoke on iOS (and Android if not dropped): app opens on Welcome placeholder (unauthenticated stub); toggle NetInfo / airplane mode shows overlay; force a render throw behind `__DEV__` if practical to see the fallback, then remove the tripwire.

## Verification

- **No T3 / navigation-config tests** (doc 12 §3.5). Reason: no logic to protect beyond wiring; T3 deferred (RISK-0001).
- Overlay connectivity **logic** may get a tiny T1 test if you extract a pure `shouldShowOverlay(isConnected)` — optional, not required.
- Run **now (STEP gate):** `npx tsc --noEmit && npm test && npx eslint . && npx prettier --check .`
- Manual: Welcome placeholder visible; three extra tabs → ComingSoon; overlay on airplane mode; error boundary composes tokens.
- Import rules: auth ↛ storefront; shared ↛ features; no `axios`/`fetch` outside `src/api/`.

## Keeping the docs true  (always)

Close OQ-33. If the shell folder path differs from `src/app/`, update doc 03 §8.1. Do not claim the cold-start three-query gate works — it does not until STEP-3/4.

## Definition of done

- [ ] Two navigators + ComingSoon tabs + placeholders in the feature folders.
- [ ] ModeProvider at navigator boundaries (`auth` / `app`).
- [ ] NetInfo overlay per doc 15 / ADR-0014.
- [ ] LoadingGate present but not blocking.
- [ ] Root error boundary per OQ-33; OQ-33 closed in docs.
- [ ] PersistGate + Provider wired to `createStore()`.
- [ ] STEP test gate green: `npx tsc --noEmit && npm test && npx eslint . && npx prettier --check .`
- [ ] No T3 tests (reason: RISK-0001).
- [ ] TSDoc on new shell components.
- [ ] App README matches how to run the shell.

## Next

Mark 2.6 Done in the PLAN and index. This is the last substep: **STEP review** (PR on `step-0002-app-scaffold` in the app repo + docs hub) plus the doc-drift check. After merge, archive PLAN + prompts from `Upcoming Prompts/` into `prompts/001-poc/step-0002/`, mark STEP-2 **Done** on `prompts/` `main`.

Tell the user the next action from `./doctor.sh status` (likely **plan STEP-3** if Andres has not started it, or **run STEP-4** if STEP-3 is already in flight). **Fresh chat** for that.
