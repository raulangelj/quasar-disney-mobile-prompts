# quasar-disney-mobile — STEP-3.1: Wire types — transcribe doc 11 §7

> **How to run:** Tell your agent *"run substep 3.1"*. This substep is self-contained — run it
> cold in a fresh chat.

## Context

STEP-3 (`Upcoming Prompts/quasar-disney-mobile-STEP-3-PLAN.md`) turns the interface contract into
running TypeScript. This is its first substep and the one the rest of the STEP rests on.

Right now `architecture/11-interface-contracts.md` §2.2 describes an **inversion**: the
*consumer-facing contract of record* lives in the docs hub, while the *authoring source of truth* —
the TypeScript wire types — lives in a repo that did not exist when the doc was written. §3.1 of
that doc states the transcription as an explicit **obligation** so the types get written from a
specification rather than from someone's memory of a conversation.

This substep discharges that obligation and flips the handover:

> **Doc 11's tables are normative until the TS types exist. From this substep onward the TS types
> are normative**, and doc 11 must be updated in the same PR as any wire-shape change (§12).

Nothing in this substep imports React Native, axios, or Redux. It is plain TypeScript.

## Read these first

- root `.throughstone/local-user.md` — experience level and communication style
- `Code/quasar-disney-mobile-docs/overview.md`
- **`Code/quasar-disney-mobile-docs/architecture/11-interface-contracts.md`** — §2 (authoring
  source vs contract of record), §3 (artifact locations + the scaffold obligation), §4.3–§4.4
  (compatibility; the unknown-enum rule), §5 (the five operations), §6.1–§6.4 (envelope, two
  cursors, conventions, why `expiresAt` is an ISO string), §7 (**the payload tables you are
  transcribing**), §8 (error model), §11.2 (`tsc` is the contract test), §12 (the same-PR rule)
- `Code/quasar-disney-mobile-docs/architecture/04-data-model.md` §1.2–§1.5
- `Code/quasar-disney-mobile-docs/architecture/03-architecture-overview.md` §8.1–§8.2 (source
  layout and import rules)
- `Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md` §2 (T0), §10.1 (filenames,
  colocated tests, `strict`, no `any`)
- `Code/quasar-disney-mobile-docs/adr/ADR-0016-typescript-types-as-the-contract.md`
- `Code/quasar-disney-mobile-docs/adr/ADR-0007-container-card-shared-types.md`
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`,
  `Code/quasar-disney-mobile-docs/coding-standards/api.md`
- `Code/quasar-disney-mobile-app/README.md` (the repo you are working in)

## Scope

**Owns:** `src/api/types/` in the app repo, and the doc 11 handover.

**Does not touch:** axios, interceptors, `baseApi`, mocks, fixtures, any feature, any UI, any
Redux slice. No runtime code at all beyond the page-size constants.

## Your task

### 1. Create `src/api/types/`

One file per concern, `camelCase.ts`, each type carrying a docstring that names the doc 11 section
it comes from.

**`card.ts`**

- `AspectRatio` = `'2:3' | '16:9' | '3:4'` (doc 11 §7.1; `'2:3'` portrait, `'16:9'` progress and
  landscape, `'3:4'` hero).
- `Card`:
  | Field | Type |
  |---|---|
  | `id` | `string` (UUID v4) |
  | `title` | `string` — **`title`, not `name`** (OQ-22 closed in 1.11) |
  | `artwork` | `Partial<Record<AspectRatio, string>>` |
  | `rating` | `string \| null` |
  | `releaseYear` | `number \| null` |
  | `genre` | `string \| null` |
  | `badge` | `string \| null` |
  | `tagline` | `string \| null` |
  | `progress?` | `number` — `0..1`, populated **only** inside a `progress` container |
  | `remainingMinutes?` | `number` — `progress` only; formatted through i18n by the UI, never a hardcoded phrase |
  | `episodeLine?` | `string \| null` — `progress` only; `null` for movies |

  Honour doc 12 §10.2's rule and say so in the docstrings: **`null`** for a field that always
  exists but may be empty (`rating`); **omitted** for a field that does not apply to that variant
  (`progress?`).

**`container.ts`**

- `ContainerVariant` = `'hero' | 'progress' | 'standardPortrait' | 'standardLandscape'`. Document
  that `'live'` and landscape-as-shipped are parked for STEP-7, that `'progress'` is **never** a
  member of a `/home-feed` response (ADR-0006), and that an **unrecognized value drops the row and
  emits a `console.warn` — it does not throw and does not fall back to a default layout** (doc 11
  §4.4).
- `Container`: `id: string`, `name: string` (a display string rendered **verbatim** — it is data,
  not UI copy), `variant: ContainerVariant`, `resources: Card[]` (**`resources`, not `items`** —
  ADR-0007), `nextCursor: string | null`.
- **`visibleCount` is not a field.** Tile peek is client-side, keyed by variant (OQ-37 closed).

**`envelope.ts`**

- `Page<T>` = `{ data: T[]; nextCursor: string | null }` — doc 11 §6.1. Never a bare top-level
  array. Document doc 11 §6.2's two-cursor rule: the envelope's cursor pages **containers**
  (vertical); each `Container.nextCursor` pages that row's **`resources`** (horizontal). `null`
  means exhausted; cursors are **opaque** — the client never parses, compares, or constructs one,
  and always trusts `nextCursor` over arithmetic on counts.
- `PageQuery` = `{ cursor?: string; limit?: number }`.
- Page-size defaults as named constants (doc 11 §6.3, OQ-23): `HOME_FEED_LIMIT = 16`,
  `CONTINUE_WATCHING_LIMIT = 10`, `CONTAINER_RESOURCES_LIMIT = 10`. Document that `limit` is a
  request *hint* the server may cap.

**`auth.ts`**

- `LoginRequest` = `{ email: string; password: string }` — request DTO, never stored, never logged.
- `LoginResponse` = `{ accessToken: string; expiresAt: string }` — **`expiresAt` is an ISO 8601
  UTC string on the wire, not the JWT's numeric `exp`** (doc 11 §6.4). Put the reason in the
  docstring: it kills the seconds-vs-milliseconds bug class and means no `jwt-decode` dependency
  ever enters doc 03 §7's hard-dependency list. The auth slice may store it however it likes; this
  constrains the wire only.
- `MeResponse` = `{ id: string; userName: string }` — no email, no roles, **not even a stub**
  (doc 16 §4; ADR-0010).

**`errors.ts`**

- `ErrorCode` = `'INVALID_CREDENTIALS' | 'UNAUTHORIZED' | 'VALIDATION_FAILED' | 'NOT_FOUND' | 'INTERNAL'`
  — `UPPER_SNAKE_CASE` by deliberate decision (doc 12 §10.2), unlike the domain enums.
- `ApiErrorBody` = `{ error: { code: ErrorCode; message: string; fields?: string[] } }` — the wire
  shape (doc 11 §8.2).
- `ApiError` = `{ code: ErrorCode; status: number; message: string }` — the single client-facing
  type both transports normalize into (doc 11 §8.3).
- Document the two rules that are easy to get wrong: **`code` is what the client switches on;
  `message` is developer-facing and is never rendered to a user** (all user-visible error copy is
  i18n keyed by `code`), and the **401 collision** — `INVALID_CREDENTIALS` and `UNAUTHORIZED` are
  distinct codes specifically so that the session-clearing reaction is mechanical rather than a
  path comparison (doc 11 §8.4, ADR-0017).

**`index.ts`** — a barrel re-exporting everything above. Features import from `src/api/types/`
only (doc 03 §8.2).

### 2. Discharge the doc 11 handover

In `Code/quasar-disney-mobile-docs/architecture/11-interface-contracts.md`:

- **§2.1** — the "Authoring source of truth" row now points at `src/api/types/` in the app repo,
  not at a repo that does not exist.
- **§2.2** — record that the inversion is resolved: the TS types **are** normative from this
  substep on, and doc 11's tables are the consumer-facing contract of record. Keep §12's same-PR
  rule prominent.
- **§3** — update the "Where, after the scaffold STEP" column for the TS-wire-types row to the
  actual path, and add the pointer the table promises.
- **Version Log** — add a row: date `2026-08-17`, STEP `STEP-3.1`, change = types transcribed;
  authoring source is now `src/api/types/`; **no wire-shape change** (say this explicitly if no
  field, name, enum, envelope, or error code differs from §7 — and if one *does* differ, that is a
  contract change and §4.3/§12 apply).

### 3. Add the contract-of-record line to the app repo README

Doc 11 §3.1 item 2. One line in `Code/quasar-disney-mobile-app/README.md` pointing at
`architecture/11-interface-contracts.md` as the contract of record, and noting that `src/api/types/`
is the authoring source.

### 4. Check `@env` typings exist

STEP-2 owns `src/types/env.d.ts` (declaring `API_BASE_URL`, `DEMO_EMAIL`, `DEMO_PASSWORD` from
`@env`, per ADR-0015). If it is missing, add it and say so in the substep summary — do not silently
absorb another STEP's obligation.

## Verification

This substep is **types plus constants**, so its test is the type checker plus one small unit test:

- **T0 (required):** `npx tsc --noEmit` passes with `strict: true`. This is the contract test
  (doc 11 §11.2) — the whole strategy rests on it.
- **T1:** `src/api/types/envelope.test.ts` — assert the three page-size constants are `16`, `10`,
  `10`. Small, but it is the only runtime value in this substep and doc 11 §6.3 is the only place
  those numbers are stated; a silent drift here changes what every mock handler pages.
- **No `any` anywhere.** Grep the new files to confirm.
- Compile-time checks worth writing as type-level assertions rather than runtime tests: that
  `Card.artwork` accepts a partial ratio map, that `Page<Container>` and `Page<Card>` both
  instantiate, and that an unknown string is not assignable to `ContainerVariant`.

Run before marking done:

```
npx tsc --noEmit
npx jest src/api/types
```

## Keeping the docs true  (always)

This substep **is** a doc-truth substep — §2 above is the point of it. Beyond that:

- If transcription reveals a genuine gap or contradiction in doc 11 §7 (a field the tables leave
  ambiguous, a type that cannot be expressed as written), **do not silently invent a resolution**.
  Fix doc 11's table, bump its Version Log, and note it in the substep summary. Doc 11 §4.3 makes
  Phase-1 breaking changes free until the backend team accepts the contract (OQ-34) — the cost is
  a Version Log entry, not a negotiation.
- New domain terms go in `architecture/13-glossary.md`.
- No secrets in the repo; `.env` stays gitignored.

## Definition of done

- [ ] `src/api/types/` contains `card.ts`, `container.ts`, `envelope.ts`, `auth.ts`, `errors.ts`,
      and `index.ts`, transcribing doc 11 §7 and §8 completely.
- [ ] Every exported type, enum, and constant carries a docstring naming its doc 11 section.
- [ ] `npx tsc --noEmit` is green under `strict: true`; no `any` appears in the new files.
- [ ] `npx jest src/api/types` is green.
- [ ] Doc 11 §2.1, §2.2, and §3 record that the TS types are now the authoring source, and its
      Version Log is bumped.
- [ ] The app repo README carries the contract-of-record line.
- [ ] Any accepted risk or deferred debt is recorded in `registries/risks.yml`, or explicitly
      marked not applicable.

## Next

Update this substep's status in the STEP PLAN, then tell the user the next action: **"run substep
3.2"**, in a fresh chat.
