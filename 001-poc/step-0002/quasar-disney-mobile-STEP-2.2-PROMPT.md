# quasar-disney-mobile — STEP-2.2: Source layout, env, register repo

> **How to run:** Tell your agent *"run substep 2.2"*. Self-contained — executable cold in a fresh chat.

## Context

2.1 created a booting RN repo. This substep turns it into the architecture's folder map, writes `.env.example`, fills the README, and registers the repo. **When this merges to `main`, STEP-3 may start** (Andres: contract types, `baseApi`, mocks). Do not transcribe doc 11 §7 — empty `src/api/types/` plus a pointer is enough.

PLAN: `Upcoming Prompts/quasar-disney-mobile-STEP-2-PLAN.md`

## Read these first

- root `.throughstone/local-user.md`
- `Upcoming Prompts/quasar-disney-mobile-STEP-2-PLAN.md`
- `Code/quasar-disney-mobile-docs/architecture/03-architecture-overview.md` §8 (layout + import rules)
- `Code/quasar-disney-mobile-docs/architecture/11-interface-contracts.md` §2–§3.1 (handoff rule)
- `Code/quasar-disney-mobile-docs/architecture/09-environments.md` §4 (`.env` keys, ADR-0015)
- `Code/quasar-disney-mobile-docs/architecture/13-glossary.md` (API module / `@env` terms)
- `Code/quasar-disney-mobile-docs/templates/repo-readme-template.md`
- `Code/quasar-disney-mobile-docs/templates/env-example.txt` (use its *convention*, replace example keys with doc 09's three)
- `Code/quasar-disney-mobile-app/README.md` (read before editing)
- `Code/quasar-disney-mobile-docs/registries/repos.yml`
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`

## Scope

**Owns:** `src/` tree, entry-file retarget, `.env.example` + `declare module '@env'`, README, `repos.yml`, clarifying doc 11 §3.1 so transcription is STEP-3's job, thin PR of 2.1+2.2 to `main`.

**Does not:** Emotion theme, ESLint/Jest/CI (2.3), atoms, navigation, `baseApi`, axios, mocks, wire types.

## Your task

1. Work on `step-0002-app-scaffold` in the app repo and docs hub. `prompts/STEP-index.md` edits stay on `prompts/` `main`.
2. **Create the tree** (empty modules with a short `README.md` or barrel only where needed so Git keeps directories):

```
src/app/                       # will become the composition root in 2.6
src/features/auth/
src/features/storefront/
src/shared/theme/
src/shared/ui/
src/shared/i18n/
src/shared/analytics/
src/api/types/                 # empty — STEP-3 transcribes doc 11 §7 here
src/api/mocks/
src/api/client/
```

   Add `src/api/baseApi.ts` as a **stub comment only** (or a file that exports nothing but a TSDoc pointing at STEP-3). Do not call `createApi` yet if `@reduxjs/toolkit` is not a dependency — adding RTK is 2.3.

3. Point the RN entry (`index.js` / `App.tsx`) at `src/app/` — a temporary `src/app/App.tsx` that still renders the template hello screen is fine. Keep iOS/Android booting.
4. **`.env.example`** committed with exactly:

   ```
   API_BASE_URL=https://api.example.invalid
   DEMO_EMAIL=demo@example.invalid
   DEMO_PASSWORD=replace-me
   ```

   One-line comment per key, matching doc 09 §4.2. Create a gitignored `.env` locally from it (do not commit). Add `src/types/env.d.ts` (or similar) with `declare module '@env'` listing those three keys as `string`. Do **not** install `react-native-dotenv` until 2.3 (Jest/`babel` land together). The declaration can exist first.
5. **README** from `templates/repo-readme-template.md`: Overview (this is the RN client; docs hub owns architecture; link `architecture/03-architecture-overview.md` and `architecture/11-interface-contracts.md` as the **contract of record until STEP-3 types exist**), stack, prerequisites (versions from 2.1), setup (`cp .env.example .env`, pods, metro), run, test (placeholder until 2.3), configuration keys, project structure tree, troubleshooting (RISK-0003 notes). Skip a repo-root `ARCHITECTURE.md` — the hub docs are the system design; this repo is not yet internally complex.
6. **Register** in `registries/repos.yml`:

   ```yaml
   - name: "quasar-disney-mobile-app"
     location: "Code/quasar-disney-mobile-app/"
     type: app
     remote: "git@github.com:raulangelj/quasar-disney-mobile-app.git"
     description: "React Native iOS + Android client: shell, auth, storefront, shared kernel, in-process API module."
   ```

   Edit only by appending; do not re-sort the file.
7. **Docs true:** In `architecture/11-interface-contracts.md` §3.1, split the obligation: STEP-2 creates the tree + README pointer + `repos.yml`; STEP-3 transcribes §7. Bump the Version Log (patch). In `architecture/03-architecture-overview.md`, note the repo now exists at that location if the "does not exist" wording is still there (patch). Glossary "after scaffold" pointers may need the same one-line fix.
8. **Thin PR to `main`** covering 2.1+2.2 in the app repo **and** the docs-hub `repos.yml` / doc pointer commits (same branch name). Merge it (or leave it ready-to-merge if the user owns merge). After merge, continue later substeps on `step-0002-app-scaffold`. Tell the user **STEP-3 may start**.

## Verification

- **No unit tests** — folders, env template, and registry. Reason: no behavior yet. Confirm iOS still boots (and Android if not dropped).
- `src/api/types/` exists and contains **no** transcribed `Card`/`Container` interfaces.
- `.env` is gitignored; `.env.example` is committed; no real secrets.
- `repos.yml` has exactly one new row; duplicate-scan N/A (YAML names). Run `Code/quasar-disney-mobile-docs/scripts/check.sh` if cheap.

## Keeping the docs true  (always)

Small clarifying change to doc 11 §3.1 and any "repo does not exist" sentences — Version Log bump. Do not invent a new ADR for a sequencing split the planning session already made.

## Definition of done

- [ ] Layout matches doc 03 §8.1; `src/api/types/` empty of wire types.
- [ ] `.env.example` has the three keys; `@env` declaration file exists.
- [ ] README filled; points at doc 11 as contract of record.
- [ ] `repos.yml` lists `quasar-disney-mobile-app`.
- [ ] Doc 11 §3.1 (and stale "repo missing" lines) updated.
- [ ] 2.1+2.2 PR opened / merged to `main`.
- [ ] Tests N/A (structure-only); app still boots.
- [ ] User told STEP-3 can start.

## Next

Mark 2.2 Done in the PLAN and index (`prompts/` `main`). **Fresh chat:** `run substep 2.3`. Andres can `run STEP-3` in parallel after this merge.
