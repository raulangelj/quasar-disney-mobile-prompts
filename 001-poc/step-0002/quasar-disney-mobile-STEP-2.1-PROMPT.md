# quasar-disney-mobile — STEP-2.1: Create app repo & boot both platforms

> **How to run:** Tell your agent *"run substep 2.1"*. Self-contained — executable cold in a fresh chat.

## Context

STEP-2 scaffolds `quasar-disney-mobile-app` so later STEPs have a host. This substep is the critical path (**RISK-0003**): a bare React Native TypeScript app that launches on iOS and Android. No feature code, no `src/` layout yet (that is 2.2). Sign-off is 2026-08-18; if Android is not booting after a focused attempt, drop it for the rest of STEP-2 and continue on iOS.

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-2-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-2-PLAN.md`
- `Code/quasar-disney-mobile-docs/overview.md`
- `Code/quasar-disney-mobile-docs/architecture/03-architecture-overview.md` (§1, §7)
- `Code/quasar-disney-mobile-docs/architecture/15-native-app-architecture.md` (§1, §5, §7, §8 Hermes)
- `Code/quasar-disney-mobile-docs/architecture/08-infrastructure-deployment.md` (§4 lockfiles-only)
- `Code/quasar-disney-mobile-docs/architecture/02-phasing-roadmap.md` (§8 RISK-0003 fallback)
- `Code/quasar-disney-mobile-docs/registries/repos.yml`
- `Code/quasar-disney-mobile-docs/registries/risks.yml` (RISK-0003, RISK-0005)
- `Code/quasar-disney-mobile-docs/templates/repo-readme-template.md` (stamp in 2.2; in 2.1 a stub is enough)
- `runbooks/collaboration.md` — branch-per-STEP; index edits on `prompts/` trunk

## Scope

**Owns:** creating the sibling git repo, applying the proprietary license posture, GitHub remote, RN CLI init (TypeScript, iOS + Android, Hermes, no Expo), committing generated lockfiles, proving both platforms boot (or recording the Android drop).

**Does not:** `src/` feature layout, `.env.example` keys, ESLint/CI, theme, navigation, `baseApi`, transcribing doc 11 types, filling the README beyond a one-line placeholder (2.2).

## Your task

1. **Overlap check.** Read `prompts/STEP-index.md`. No other STEP should be `In progress`. STEP-3 projects this same repo later — heads-up only, not a block.
2. **Cut the branch in the repos this substep already touches** (`quasar-disney-mobile-docs` if you edit risks; `prompts` index edits stay on **`main`**). The new app repo is created on `step-0002-app-scaffold` as its first branch (or init on `main` then immediately branch — either is fine as long as the first pushed app-repo branch is `step-0002-app-scaffold`).
3. **Flip STEP-2 to `In progress`** on `prompts/STEP-index.md` **on `prompts/` `main`**, commit, push. Scan duplicates before push:
   `grep -oE '^\|[[:space:]]*STEP-[0-9]+' prompts/STEP-index.md | grep -oE 'STEP-[0-9]+' | sort | uniq -d`
4. **Init the app** at `Code/quasar-disney-mobile-app/` with the current stable `@react-native-community/cli` template:
   - Package manager: **npm** (`--pm npm`).
   - Native/module name: **`DinseyApp`**. Display name: **`Dinsey-`**.
   - TypeScript. **No Expo.** Do not add Expo modules "for convenience".
   - Accept CLI defaults for New Architecture / Hermes unless they contradict the docs (Hermes on is required).
   - Portrait lock both platforms (doc 15 §8 / doc 07 §7): iOS `UISupportedInterfaceOrientations` = portrait only; Android `android:screenOrientation="portrait"` on the main activity.
   - Android permissions: `INTERNET` + `ACCESS_NETWORK_STATE` only. iOS: no extra usage-description strings.
5. **License:** `Code/quasar-disney-mobile-docs/scripts/apply-project-license.sh Code/quasar-disney-mobile-app`. Expect no project `LICENSE` (Proprietary), plus `LICENSE-THROUGHSTONE` and `LICENSING.md`.
6. **Gitignore:** keep the RN template's; ensure `.env`, `.secrets/`, `coverage/` are ignored. Do not commit a `.env`.
7. **GitHub:** `gh repo create raulangelj/quasar-disney-mobile-app --private --source=Code/quasar-disney-mobile-app --remote=origin` (adjust if the directory already has origin). Push `step-0002-app-scaffold`.
8. **Lockfiles:** commit `package-lock.json`, `ios/Podfile.lock` (after `pod install`), and any Gradle lock the template generated. No `.nvmrc`, no authored toolchain pin file (doc 08 §4).
9. **Boot both platforms** from the workspace:
   - iOS simulator: `npx react-native run-ios`
   - Android emulator: `npx react-native run-android`
   Record versions actually used (Xcode, RN, Node) in the app README's Prerequisites as a one-liner — full README fill is 2.2.
10. **RISK-0003 checkpoint.** If Android is not launching after a focused debugging pass (pods/signing/emulator), **drop Android for the rest of STEP-2**, continue on iOS, and note the follow-up in `registries/risks.yml` (owner = Raul Angel, revisit = before STEP-6). Do not burn the night on the emulator.
11. Set RISK-0003 `owner` to `Raul Angel`. Do not close it until both platforms have booted (if Android dropped, leave it open).

## Verification

- **No unit tests in this substep** — the deliverable is generated RN template + a process that boots. Reason: there is no project code yet; 2.3 introduces Jest config that matches doc 12.
- Prove: the template app renders on iOS. Prove Android or record the drop.
- `git status` in the new repo is clean on `step-0002-app-scaffold`; remote exists and is private.
- `apply-project-license.sh` succeeded; no project `LICENSE` file.

## Keeping the docs true  (always)

- Do **not** add the `repos.yml` row yet — that is 2.2, once the tree and README exist.
- If you drop Android, update RISK-0003's description with the actual blocker (not a new risk).
- Secrets stay out of git.

## Definition of done

- [ ] `Code/quasar-disney-mobile-app/` is a git repo on `step-0002-app-scaffold` with a private GitHub remote.
- [ ] Proprietary license posture applied (`LICENSE-THROUGHSTONE`, `LICENSING.md`, no project `LICENSE`).
- [ ] Bare RN + TypeScript, no Expo, Hermes on, portrait locked, internet-only permissions.
- [ ] Lockfiles committed.
- [ ] iOS boots; Android boots **or** is explicitly dropped per RISK-0003.
- [ ] STEP-2 is `In progress` on `prompts/` `main` (pushed).
- [ ] Tests N/A for this substep (generated template); stated above.
- [ ] RISK-0003 owner updated.

## Next

Update 2.1 to Done in the PLAN and in `prompts/STEP-index.md` (on `prompts/` `main`). Tell the user: **start a fresh chat** and run **`run substep 2.2`**.
