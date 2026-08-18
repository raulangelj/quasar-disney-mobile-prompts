# quasar-disney-mobile — STEP-3.4: Demo fixtures, placeholder art, test factories

> **How to run:** Tell your agent *"run substep 3.4"*. Self-contained — runnable cold.

## Context

STEP-3.3 built handlers that page and validate; this substep gives them data. Two different kinds
of data, and conflating them is the mistake this substep exists to avoid:

| Set | Lives | Used by | Rule |
|---|---|---|---|
| **Demo fixtures** | `src/api/mocks/fixtures/` | **The app**, in both build configurations | Hero + 15 containers; page sizes per doc 11 §6.3. Typed `Container[]` / `Card[]`, **never `any`** |
| **Test factories** | `src/api/mocks/`, as `*.factory.ts` | **Tests only** | `makeCard`, `makeContainer`, `makePage` — minimal objects, everything overridable |

> **Tests assert against factories, not demo fixtures.** A test asserting
> `containers[3].name === 'Series'` goes red the moment someone swaps a row for the demo — and under
> deadline pressure that is how a genuinely broken test gets deleted instead of fixed (doc 12 §4.1).

The one deliberate exception is the **fixture-invariant test**, which runs against the real demo
fixtures and asserts only structural rules — never content.

And the line the whole type-as-contract strategy rests on (doc 11 §11.2):

> **Mock fixtures are declared `Container[]` / `Card[]` — never `any`, and never untyped JSON
> imports.** A loosely imported `resources.json` would make `tsc` describe an intent nothing checks.

## Read these first

- root `.throughstone/local-user.md`
- **`Code/quasar-disney-mobile-docs/architecture/12-test-strategy.md`** §4.1 (**two fixture sets;
  factories colocated; the fixture-invariant test**), §3.5 (why demo *content* is not worth
  testing), §4.4 (fixture UUIDs are fixed committed constants, never generated), §10.1 (filenames,
  colocated tests)
- **`Code/quasar-disney-mobile-docs/architecture/11-interface-contracts.md`** §7 (payload shapes),
  §6.3 (page sizes), §11.2 (typed fixtures)
- `Code/quasar-disney-mobile-docs/architecture/04-data-model.md` §1.2 (**which artwork ratio each
  variant needs**), §1.3 (the three `progress`-only fields), §1.4 (hero + 15; CW is one `progress`
  container)
- `Code/quasar-disney-mobile-docs/architecture/07-ui-design-system.md` — the carousel variants and
  what each renders, so the fixtures carry the fields the UI will actually read
- `Code/quasar-disney-mobile-docs/architecture/assets/README.md` and
  `Code/quasar-disney-mobile-docs/architecture/assets/placeholder-art/`
- `Code/quasar-disney-mobile-docs/adr/ADR-0006-two-endpoint-home-composition.md`,
  `.../ADR-0007-container-card-shared-types.md`
- **DF10 in `Code/quasar-disney-mobile-docs/architecture/02-phasing-roadmap.md`** — trademark
  substitution: **no Disney/Marvel/Star Wars/hulu/ESPN marks or real key art**, in the codebase or
  the assets, at any phase. Also **RISK-0005** in `registries/risks.yml`.
- `Code/quasar-disney-mobile-docs/coding-standards/typescript.md`

## Scope

**Owns:** `src/api/mocks/fixtures/`, `src/shared/assets/placeholder-art/`, the `*.factory.ts`
files, and the fixture-invariant test.

**Does not touch:** handlers (3.3 owns them — they take the dataset as a parameter), any render-time
asset resolution (that is STEP-5's; see task 2), any feature, any UI.

## Your task

### 1. Copy the placeholder art into the app repo

Copy all seven SVGs from `Code/quasar-disney-mobile-docs/architecture/assets/placeholder-art/` into
`Code/quasar-disney-mobile-app/src/shared/assets/placeholder-art/`:

```
tile-2x3-a.svg  tile-2x3-b.svg  tile-2x3-c.svg
tile-16x9-a.svg tile-16x9-b.svg tile-16x9-c.svg
tile-3x4-hero.svg
```

Copy them unchanged. **Do not author new art** — DF10 forbids real key art at any phase, and
RISK-0005 already flags the placeholder brand as confusingly similar to Disney.

`src/shared/assets/` is a **new folder** in doc 03 §8.1's source layout. Add it to that table and
bump the doc's Version Log (see "Keeping the docs true").

### 2. Fixtures reference art by filename string — resolution is STEP-5's

`Card.artwork` values are plain `string` on the wire, exactly as a real backend would send a URL.
Fixtures therefore hold the **exact filename** (e.g. `'tile-2x3-a.svg'`), and the render-time
lookup from that string to a bundled SVG component belongs to the storefront card component in
STEP-5.

This keeps the contract honest — no `require()` handle masquerading as a wire field — and keeps
STEP-3 free of `react-native-svg` and the Metro transformer. Say so in a docstring so it is not
rediscovered on Sunday. (Recorded as PLAN Q4.)

### 3. `src/api/mocks/fixtures/` — the demo set

- **`ids.ts`** — every UUID as a **fixed committed constant** (doc 12 §4.4). Never generate one at
  module load; a fixture whose ids change per run makes cursor and paging tests non-reproducible.
- **`homeFeed.ts`** — `Container[]`, explicitly typed. **Exactly one `variant: 'hero'` container
  plus 15 others**, in the composition order the storefront expects. **No `progress` container**
  (ADR-0006). Variants across the 15: `standardPortrait` for the bulk, with `standardLandscape`
  present so STEP-7 has something to unpark. Give each container enough `resources` that horizontal
  paging past `limit: 10` is actually exercised.
- **`continueWatching.ts`** — `Container[]`, one container with `variant: 'progress'`, whose cards
  carry `progress` (`0..1`), `remainingMinutes`, and `episodeLine` (`null` for movies). These three
  fields appear **only** here.
- **`user.ts`** — `{ id, userName }` for `/me`. A PII-shaped but fake `userName`; it is never
  logged (doc 11 §9.3).
- **`index.ts`** — assembles the `MockDataset` shape 3.3's `createMockAdapter` takes.

**Artwork ratios by variant** (doc 04 §1.2 — this is what the invariant test checks):

| Container variant | Ratio its cards need |
|---|---|
| `hero` | `'3:4'` (Phase 1a ships a portrait stand-in; full spotlight chrome is Phase 2 — OQ-24) |
| `standardPortrait` | `'2:3'` |
| `standardLandscape` | `'16:9'` |
| `progress` | `'16:9'` |

Copy is **Spanish** — the reference locale (doc 07 §9, DF8). `Container.name` is a display string
rendered verbatim, so the fixture is where that Spanish lives. No trademarked names, no real
titles.

### 4. Test factories — `*.factory.ts`

In `src/api/mocks/`, colocated with the module that owns the type (doc 12 §4.1):

- `card.factory.ts` — `makeCard(overrides?: Partial<Card>): Card`
- `container.factory.ts` — `makeContainer(overrides?: Partial<Container>): Container`
- `page.factory.ts` — `makePage<T>(overrides?: Partial<Page<T>>): Page<T>`

Minimal valid objects, **everything overridable**. Two constraints worth stating in docstrings:

- **A test-time import is not an import-rule violation.** A storefront `*.test.ts` importing
  `makeContainer` from `src/api/mocks/` looks like a features → API import; doc 03 §8.2 governs
  **runtime** paths, and test files are out of its scope. Stated so a reviewer does not flag it
  under deadline pressure.
- **Factories must not be reachable from the app entry.** They live under `src/`, so anything
  non-test importing one puts them in the demo binary. The `*.factory.ts` suffix is the convention;
  they are imported only from `*.test.ts`. `tsc` still type-checks them.

### 5. The fixture-invariant test

**One** test file, `src/api/mocks/fixtures/fixtures.test.ts`, running against the **real demo
fixtures** and asserting only the structural rules the docs state (doc 12 §4.1):

- The first HomeFeed page is **exactly one `hero` plus 15 others** (doc 04 §1.4).
- **No `progress` container appears anywhere in a HomeFeed response** (ADR-0006).
- Every card carries the artwork ratio its container's variant needs (the table above).
- Every artwork filename resolves to a file that actually exists in
  `src/shared/assets/placeholder-art/`.
- Every id is unique across the whole dataset.
- `progress` / `remainingMinutes` / `episodeLine` appear **only** inside the `progress` container,
  and `progress` values are within `0..1`.

Assert **structure, never content** — no test may reference a specific `name` or `title`.

## Verification

- **T0:** `npx tsc --noEmit`. The fixtures being declared `Container[]` / `Card[]` is what makes
  this a contract test at all (doc 11 §11.2). Grep the new files for `any` and for untyped JSON
  imports; either one fails this substep.
- **T1:** the fixture-invariant test above, plus a small `factories.test.ts` proving each factory
  returns a valid object with no overrides and that every field is overridable.
- Re-run 3.3's handler suite — it now serves the real dataset, and any mismatch between fixture
  shape and handler expectation surfaces here rather than in STEP-5.

```
npx tsc --noEmit
npx jest src/api/mocks
```

## Keeping the docs true  (always)

- **Required:** `src/shared/assets/` is a new folder in doc 03 §8.1's source-layout table — add it
  and bump that doc's Version Log.
- If the fixtures cannot satisfy doc 04 §1.4's "hero + 15" or the artwork-ratio table as written,
  the doc is what changes — amend it, bump its Version Log, and update the types, mocks, and tests
  in the same PR (doc 11 §12).
- If the copied art needs a rename to keep RISK-0005's revisit trigger honest, update that risk row
  rather than renaming silently.
- Nothing here is a secret, but the demo credentials still live only in the gitignored `.env` —
  they never appear in a fixture.

## Definition of done

- [ ] Seven placeholder-art SVGs copied unchanged into
      `src/shared/assets/placeholder-art/`; no new or trademarked art added (DF10).
- [ ] `src/api/mocks/fixtures/` ships a typed `Container[]` HomeFeed set (one `hero` + 15, no
      `progress`), a typed `progress` Continue Watching set, and the `/me` user — all with fixed
      committed UUIDs and Spanish display copy.
- [ ] Fixture artwork values are plain filename strings; no `require()` handles in wire data.
- [ ] `makeCard`, `makeContainer`, and `makePage` ship as `*.factory.ts` beside them, minimal and
      fully overridable, imported only from `*.test.ts`.
- [ ] The single fixture-invariant test passes and asserts **structure only** — no test in the repo
      asserts demo fixture *content*.
- [ ] `npx tsc --noEmit` and `npx jest src/api/mocks` are green; no `any`, no untyped JSON import.
- [ ] Doc 03 §8.1 records `src/shared/assets/` and its Version Log is bumped.
- [ ] Any accepted risk or deferred debt is in `registries/risks.yml`, or explicitly not applicable.

## Next

Update this substep's status in the STEP PLAN, then tell the user: **"run substep 3.5"**, in a fresh
chat.
