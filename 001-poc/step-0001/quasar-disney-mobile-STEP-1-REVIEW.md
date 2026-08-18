# quasar-disney-mobile — STEP-1 REVIEW (Cross-Cutting)

**Date:** 2026-08-17
**Session:** 1.14
**Status:** Passed

A final read-through of STEP-1 architecture docs, ADRs, conditional templates, the STEP-1 PLAN, and `prompts/STEP-index.md`. This is a review, not an interview: contradictions and gaps were fixed in the living docs; remaining open questions are carried into planning.

## What was checked

1. **Conditional-session gate** — every `templates/architecture-sessions/conditional-*.md` vs the PLAN table, `overview.md`, and produced docs.
2. **Consistency** — phasing vs data model vs UI vs contracts vs CI vs stack (Emotion + RTK Query after the user stack change).
3. **Completeness** — referenced-but-unspecified items; Open Questions that would block implementation.
4. **Foreclosure** — Phase-1 shortcuts vs later-phase commitments in doc 02.
5. **Decision coverage** — contested/deferred decisions have ADRs (0019–0020 added earlier in 1.14; 0006/0008 amended here).
6. **Index accuracy** — `architecture/README.md` versions; `prompts/STEP-index.md` substeps.

## Conditional-session gate — PASS

| Template | PLAN row | Disposition | Still valid? |
|----------|----------|-------------|--------------|
| Native app | Include | `1.3a` Done → `architecture/15-native-app-architecture.md` | Yes |
| Identity & auth | Include | `1.6a` Done → `architecture/16-identity-auth.md` | Yes |
| Privacy / compliance | **Deferred** | Fake demo creds, no real PII; revisit **before Phase 3 real accounts / JWT** | Yes |

No missing PLAN row. No applicable conditional left incomplete. Review continued.

## Findings (and disposition)

### High — applied

| Finding | Fix |
|---------|-----|
| User rejected Styled Components + handwritten middleware. Wanted Emotion + RTK Query `baseApi` with axios interceptors. | ADR-0019, ADR-0020; docs 01–05, 07, 11–13, 15, 16, coding-standards, PLAN (earlier in this session). |
| Doc 02 Decision 2 / §9 vs later docs: live carousel recovered into 1a vs parked in 1b. | Live stays **1b**. Two-step auth stays **1a**. Hero **chrome** Phase 2; 1a ships a 3:4 stand-in. |
| OQ ID collisions: doc 07 reused OQ-28/29; doc 05 reused OQ-27. | **OQ-27** stays npm-audit/CI (closed). Eviction → **OQ-39**. `visibleCount` → **OQ-37** (closed: client-side). Logout → **OQ-38** (closed: expiry-only). |
| Doc 08 “no CI in Phase 1a” vs ADR-0018 GitHub Actions JS gate. | Decision 2 + coverage header: JS gate exists; **native** CI still Phase 2. |

### Medium — applied

| Finding | Fix |
|---------|-----|
| OQ-24 still open in docs 04/05/07/13 while 1.14 closed it. | Closed everywhere: 3:4 stand-in in 1a. ADR-0006 amended. |
| Doc 05 treated release build as hitch fallback vs docs 08/15 declared artifact. | Doc 05 §2 aligned; doc 08 §2 no longer cites 05 as the fallback. |
| Doc 05 §7 Card fields still “TBD” after OQ-26 closed. | Points at doc 04 §1.2 / doc 11 §7.1. |
| `visibleCount` implied as a Container field. | Client-side, keyed by variant (docs 04, 07, 11). |
| Logout affordance in 1a. | Expiry-only; Perfil remains ComingSoon. |
| Doc 01: OQ-01 still open; carousel count “8”; privacy session “1.7a”. | OQ-01 closed; 2 variants in 1a; privacy stays Deferred. |
| `overview.md` still listed Expo / card schema / pending UI as known unknowns. | Trimmed to planning/Phase-3 unknowns. |
| `registries/risks.yml`: RISK-0002 stale; RISK-0003 Saturday trigger past; RISK-0004 cited OQ-10; RISK-0013 “no CI”. | RISK-0002 **closed**. RISK-0003 retargeted to foundation STEP. RISK-0004 → OQ-34. RISK-0010/0013 distinguish JS vs native CI. |
| ADR-0008 still said “Session 1.8 must reopen OQ-27”. | Amendment: 1.8 closed it as deferred. |

### Low — applied or accepted

| Finding | Disposition |
|---------|-------------|
| Doc 02 Decision 10/15 duplicated after a partial replace. | Deduplicated; Decision 7 names RTK Query. |
| Doc 01 Decision 4 lost its row number. | Restored. |
| Doc 01 Decision 6 still says “1 senior dev”. | Accepted leftover — doc 02 Decision 13 (two seniors) is the schedule source of truth. Not blocking code. |

## Fixes applied (docs bumped)

| Doc / artifact | Version |
|----------------|---------|
| `overview.md` | known unknowns + two-step / 2-variant 1a |
| `01-system-overview.md` | v0.1.6 |
| `02-phasing-roadmap.md` | v0.4.0 |
| `04-data-model.md` | v0.2.6 |
| `05-scaling-performance.md` | v0.1.3 |
| `07-ui-design-system.md` | v0.2.1 |
| `08-infrastructure-deployment.md` | v0.1.2 |
| `11-interface-contracts.md` | v0.3.1 |
| `13-glossary.md` | v0.2.1 |
| `architecture/README.md` | index reconciled |
| ADR-0006, ADR-0008 | amendments |
| `registries/risks.yml` | RISK-0002 closed; 0003/0004/0010/0013 updated |
| STEP-1 PLAN | DoD checked; Q3/Q8 closed; stack line updated |

ADRs 0019 (Emotion) and 0020 (RTK Query `baseApi`) were written earlier in this same 1.14 pass.

## Canonical implementer layout (locked)

```
src/app/                       shell: baseApi.reducer + middleware, injectStore, Emotion ThemeProvider
src/features/auth/api.ts       injectEndpoints: login, getMe
src/features/storefront/api.ts injectEndpoints: getHomeFeed, getContinueWatching, getContainerResources
src/shared/{theme,ui,i18n,analytics}/
src/api/baseApi.ts
src/api/types/
src/api/mocks/                 axios-mock-adapter + fixtures
src/api/client/                axios instance, interceptors, axiosBaseQuery, baseQueryWithAuth
```

Features may import `src/api/types` and own `api.ts` hooks; never `axios` / `src/api/client/`.

## Open questions for planning (do not block 1.14)

**Need names / owners before or at the planning session**

| ID | Question |
|----|----------|
| OQ-12 | Who is Dev A / Dev B? |
| OQ-18 | Application repo name |
| OQ-13 | Who outlines the wordmark before 18 Aug? |
| OQ-28 | Who owns the sign-off binary, and on which two devices? |
| OQ-33 | Error-boundary fallback screen (copy + atoms + surface mode) |
| OQ-36 | Who owns a red trunk during the Sat–Mon parallel window? |

**Phase 2 / 3 — parked with triggers**

| ID | Question |
|----|----------|
| OQ-03 | Production JWT claims / IdP vendor |
| OQ-05 | Bitrise setup |
| OQ-06 | Budget |
| OQ-04 | Real streaming-app migration timeline |
| OQ-30 | Server-localized `Container.name` / locale on the request |
| OQ-31 | Native `staging` config at Phase 3 |
| OQ-32 | Bitrise secret source / CI-as-environment |
| OQ-34 | Backend team accepts the contract (doc 11 §14) |
| OQ-35 | Does Bitrise subsume the GitHub Actions JS gate? |
| OQ-39 | RTK Query cache eviction on a long paginated feed |

**Closed in 1.14 (do not reopen)**

OQ-01, OQ-17 (reversed: mocks on the real axios instance), OQ-24, OQ-37, OQ-38. OQ-27 stays closed-as-deferred (Bitrise `npm audit`).

## Next

Architecture STEP is **Done**. Start a **fresh chat** and run:

*Run planning session: Phase-1 implementation roadmap*

(`templates/planning-session.md`). Confirm with `./doctor.sh status`.
