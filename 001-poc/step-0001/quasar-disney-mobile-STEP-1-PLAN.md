# quasar-disney-mobile — STEP-1 PLAN: Architecture

**Phase:** Phase 1 — POC
**Owner:**
**Status:** Done
**Date:** 2026-08-14
**Branch:** `step-0001-architecture`
**Repos (projection):** `quasar-disney-mobile-docs`, `prompts`

> STEP-1 locks the shape of the stakeholder-demo streaming app — login, carousel storefront,
> Redux + dummy API boundary, modular RN structure — as architecture docs and ADRs before any
> application code is written. This is the foundation for Phase 1 implementation and later
> migration of the real streaming product to the Throughstone method.

## Motivation

The team needs a durable, reviewable design for a Disney+-style React Native demo that
showcases Throughstone architecture to stakeholders and serves as the template for the
production streaming app. STEP-1 captures requirements, component boundaries, data/auth
contracts, UI system, and test strategy so implementation STEPs can proceed without
rearchitecting mid-build.

## Decisions already locked

- `Code/quasar-disney-mobile-docs/overview.md` — stakeholder POC; v1 = two-step login + 2
  carousel variants in 1a; fake demo auth with JWT-ready boundaries; Redux Toolkit + RTK
  Query `baseApi` + dummy API (`axios-mock-adapter`); React Native (iOS + Android);
  TypeScript; Emotion; atomic design; modular directories extractable to other repos.
- root `.throughstone/local-user.md` — Experience level 3; Communication style Normal.
- `registries/risks.yml` — review before security- or auth-sensitive sessions.
- `architecture/03-architecture-overview.md` — RN iOS+Android only; feature-based Clean Architecture inside a modular monolith (shell composition root); five in-app modules (no backend component); **RTK Query `baseApi`** + axios interceptors; **Emotion** ThemeProvider; redux-persist of the auth slice into Keychain/Keystore from day one. ADRs 0001–0003, **0019**, **0020**.
- `architecture/04-data-model.md` — User + `/me`; **Container** / **Card** (`resources`); HomeFeed and ContinueWatching as two JWT `Container[]` GETs composed under hero (`progress` variant); persist auth only; UUID + `nextCursor`; mock 7-day JWT `exp`; privacy session still Deferred (Phase 3). ADR-0006, ADR-0007.
- `architecture/05-scaling-performance.md` — Demo load &lt;10 sessions; no infra scale; on-device storefront is first bottleneck; RAM-only catalog cache; mock latency 400–600 ms.
- `architecture/06-security-threat-model.md` — Abbreviated threat model Done. Binary mock-JWT gate; demo creds in `.env`; network STRIDE / rate limit deferred to Phase 3; `npm audit` in CI deferred to Bitrise (OQ-27). ADR-0008. RISK-0008–0010. Privacy still Deferred.
- `architecture/16-identity-auth.md` — Email+password now; no IdP in Phase 1; Phase 3 **buy** behind our API (ADR-0009). Binary AuthZ, no tenancy, no s2s (ADR-0010). Mock JWT `sub`/`exp`/`iat`; `/me` = `{ id, userName }` (OQ-25 closed). Vendor/production claims remain OQ-03.
- `architecture/13-glossary.md` — Living term table. Packaging named **feature-based Clean Architecture** inside the modular monolith; I/O is RTK Query `baseApi`; theme is Emotion. Wire types stay in `src/api/types/` (features may import types + own `api.ts` hooks). Doc 03 §2 echoes the naming.

## Substeps

| # | Title | Produces | Depends on | Open questions |
|---|-------|----------|------------|----------------|
| 1.1 | System Overview, Requirements & Non-Goals | `architecture/01-system-overview.md` | overview.md | Final non-goals list from overview |
| 1.2 | Phasing & Roadmap | `architecture/02-phasing-roadmap.md` | 1.1 | Phase 2/3 sequencing for real app migration |
| 1.3 | Architecture Overview & Component Boundaries | `architecture/03-architecture-overview.md` | 1.1, 1.2 | ~~Expo vs bare RN; module/repo split~~ resolved |
| 1.3a | Native App Architecture *(conditional)* | `architecture/15-native-app-architecture.md` | 1.3 | ~~OTA / offline~~ resolved (online-only + local installs) |
| 1.4 | Data Model, Ownership & Retention | `architecture/04-data-model.md` | 1.3 | ~~Carousel/card content schema~~ resolved |
| 1.5 | Scaling & Performance | `architecture/05-scaling-performance.md` | 1.3, 1.4 | Demo-scale defaults |
| 1.6 | Security & Threat Model | `architecture/06-security-threat-model.md` | 1.4 | ~~Token storage on device~~ resolved (1.3a + 1.6); CI audit → OQ-27 |
| 1.6a | Identity & Auth *(conditional)* | `architecture/16-identity-auth.md` | 1.4, 1.6 | ~~JWT shape; dummy vs IdP path~~ resolved (OQ-25 closed; OQ-03 vendor remains) |
| 1.7 | UI / Design System | `architecture/07-ui-design-system.md` | 1.3 | Reference screens in `inputs/ui/` |
| 1.8 | Infrastructure & Deployment | `architecture/08-infrastructure-deployment.md` | 1.3 | CI for RN; no backend in v1 |
| 1.9 | Environments | `architecture/09-environments.md` | 1.8 | Dev/staging placeholders |
| 1.10 | Observability | `architecture/10-observability.md` | 1.3, 1.8 | Logging for demo scope |
| 1.11 | Interface Contracts | `architecture/11-interface-contracts.md` | 1.4, 1.6a | Dummy API endpoint naming |
| 1.12 | Test Strategy | `architecture/12-test-strategy.md` | 1.3, 1.11 | RN unit/integration/e2e tiers |
| 1.13 | Glossary | `architecture/13-glossary.md` | prior docs | Domain terms (carousel, tile, etc.) |
| 1.14 | Cross-Cutting Review | review doc | all above | — |

Run conditional substeps **by name** (*"run the native-app session"*, *"run the identity-auth
session"*) using the matching `templates/architecture-sessions/conditional-*.md` file.

## Test plan

STEP-1 produces **no application code**. No test execution in this STEP. The Test Strategy
session (1.12) defines tiers and gates for later implementation STEPs.

| Test tier / surface | Substep(s) | Tests to create or update | Run timing | Command / gate | Notes |
|---------------------|------------|---------------------------|------------|----------------|-------|
| — | — | — | — | — | Architecture-only STEP |

## Conditional sessions considered

| Conditional session | Owning session | Decision | Substep / reason / revisit trigger |
|---------------------|----------------|----------|------------------------------------|
| Native app (mobile / desktop) | 1.3 Architecture Overview | **Include** | `1.3a` → `architecture/15-native-app-architecture.md` — React Native iOS + Android client |
| Identity & auth | 1.6 Security & Threat Model | **Include** | `1.6a` → `architecture/16-identity-auth.md` — login screen + JWT-ready dummy auth |
| Privacy, compliance & data governance | 1.4 Data Model / 1.6 Security | **Deferred** | v1 uses fake demo credentials only; revisit **before Phase 3 real accounts / real JWT** |

## Open questions

- ~~Q1: Expo vs bare React Native~~ **Resolved (1.1 + 1.3):** bare RN, no Expo.
- ~~Q2: Login and storefront UI reference images~~ **Resolved (1.2):** `inputs/ui/disney-plus-reference-screens.md`.
- ~~Q3: Carousel variant set and card metadata~~ **Resolved (1.4 + 1.5 + 1.11 + 1.14):** Container / Card / `resources` in `architecture/04-data-model.md` and ADR-0007. Extra Card fields closed as OQ-26. Hero chrome vs stand-in closed as OQ-24 (3:4 stand-in in 1a).
- Q4: Production JWT and API contract — owner: backend team — client mock claims closed in 1.6a (OQ-25); vendor + real claims remain OQ-03 → Phase 3 / **OQ-34**.
- ~~Q8: Bitrise `npm audit` / lockfile CI gates (fail vs warn)~~ **Closed (1.8) as deferred:** OQ-27; RISK-0010 revisit is Bitrise implementation. JS CI exists (ADR-0018) but does not run `npm audit`.
- ~~Q5: Persist library (`react-native-encrypted-storage` vs Keychain adapter)~~ **Resolved (1.3a):** `react-native-encrypted-storage`.
- Q6: Mock strategy — **Resolved then reversed (1.14 / ADR-0020):** `axios-mock-adapter` on the real axios instance so interceptors run in Phase 1.
- ~~Q7: Pagination wire format (cursor vs offset, page sizes)~~ **Resolved (1.4 + 1.5):** opaque `nextCursor`; both feeds `Container[]`; HomeFeed first page = hero + 15; CW `progress` separate. Envelope names → 1.11 (OQ-22); card page size → OQ-23.

## Ground rules

- **No application code in STEP-1.** Output is Markdown architecture docs + ADRs only.
- **One decision cluster at a time** in each session interview.
- **Calibrate** to root `.throughstone/local-user.md` (Level 3, Normal).
- **Significant decisions → ADRs**; living design → architecture docs with Version Log updates.
- **Update `prompts/STEP-index.md`** after each completed substep.
- **Branch:** `step-0001-architecture` in `quasar-disney-mobile-docs` and `prompts` where applicable.

## Definition of done

- [x] All core architecture sessions (1.1–1.14) completed; each output doc written and indexed.
- [x] Conditional sessions **Native app** (1.3a) and **Identity & auth** (1.6a) completed.
- [x] Privacy/compliance consciously **Deferred** with revisit trigger recorded (above).
- [x] Cross-Cutting Review passed (no missing applicable conditionals; docs coherent).
- [x] `prompts/STEP-index.md` reflects final substep statuses; STEP-1 row marked **Done**.
- [ ] User runs **planning session** next to outline Phase 1 implementation STEPs (STEP-2+).
