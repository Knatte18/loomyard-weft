# Support log — shedrecipe-capabilities

## Interview

Autonomous run: no operator answered; every pick below is the writer's own, recorded in the question ledger.

### Turn 1 — scope and the board

Read the board entry, whose operator decision of 2026-10-17 fixes the two steps, and the entries that touch it: `row-entry-gate` (plans to read its gates through this task's `Setting(module, key)`), `command-producer`, `config-migrate-hub-layer` (issue #709, stale hub keys after a rename), `shed-arbiter-all` and `run-verbs-shared`.
Both prerequisites named by the entry have landed: the lens and fixer fan keys (`112cccb8f`, `4d3c8ea4f`) and the loom model key renaming (`a273d69bc`).

### Turn 2 — what the per-row maps hold

Read `shedrecipe.Env`, the loom and darn wiring and the three review entries.
The per-row maps split into two classes: values needing fallback or resolution (models, model keys, Bouncer models, budgets, hard caps) and plain single keys (reviewer fans, fixer fans, auto-accept).
Constructors receive only the row name, so a per-segment map needs the segment passed in.

### Turn 3 — design batch (auto-picked)

- Q: key `Segments` by segment or by Bouncer row? Recommended: segment, as the operator wrote; picked.
- Q: how does an entry learn its segment? Recommended: `Constructor` takes the row's name and segment; picked.
- Q: what goes through `Setting` now? Recommended: the plain single-key values, fans and auto-accept; picked.
- Q: where does the hub lookup live? Recommended: `configreg`, strict `configengine.Load`, known keys from the template; picked.
- Q: reference form? Recommended: `module.key`, matching `row-entry-gate`; picked.
- Q: cache lookups? Recommended: no; picked, for `row-entry-gate`'s mid-run reads.

### Turn 4 — lens review

Every lens returned a report.

- premises, MEDIUM, D8 status path has no `Env`/`Segments` and degrades to the template — fixed: `Routing`/`DarnRouting` take budgets only; status path keeps its own reads.
- premises, MEDIUM, loom's fix-fan refusal text is the bare key, not `<key> in loom.yaml` — fixed: D5 states loom's text gains the file name; Constraints names that one change.
- premises, LOW, fan names validated only by module loaders; scalar `""` reads as one entry — fixed: D4 bound and D5 null/empty rule.
- premises, LOW, darn auto-accept false — fixed in the first acceptance criterion.
- premises, LOW, "known" against template, not loaded doc — fixed in D4.
- scenarios, MEDIUM, status path cannot build full `SegmentSettings` — merged with premises D8, fixed.
- scenarios, MEDIUM, uncached reads can split one segment's fan value mid-build — fixed: D4 snapshot per lookup value, one per build.
- scenarios, LOW, `Setting` access path unbounded — fixed: D4 bound.
- scenarios, LOW, fix row's reference checked through the lookup — fixed in D6 and acceptance.
- scenarios, LOW, null values — fixed: null is absent/false, scenario added.
- scenarios, LOW, negative budget refusal kept — fixed in D8 and acceptance.
- scenarios, LOW, strict reads reaching lightweight verbs — fixed: D6 states reads run where the recipe builds, whose wiring already loads strictly; D8 keeps status off them.
- board-fit, MEDIUM, `Setting` type must stay passable to `shedengine` for `row-entry-gate` — fixed: D4 plain func type.
- board-fit, MEDIUM, darn edge rewrite is `row-entry-gate`'s to remove — fixed: Open risks.
- board-fit, LOW, batten fill owed by `row-entry-gate` — fixed: Open risks.
- board-fit, LOW, #709 residual silent default — fixed: Open risks.
- board-fit, LOW, `command-producer` and regression fit — no change needed.
- board-fit, LOW, guard keys bounded by local layer — fixed in D4.
- pattern-fit, MEDIUM, `reviewbudget` owns budget/cap resolution — fixed: D1 names it; added to relevant list.
- pattern-fit, MEDIUM, new PATTERN entry unenforced and too broad — fixed: text narrowed to per-row/per-segment values; Notes says review-held.
- pattern-fit, LOW, local-layer scenario needs a `CategoryNone` key — fixed in acceptance.
- pattern-fit, LOW, `PATTERN-hub-wide-config` — fixed: D4 cites it.
- pattern-fit, LOW, `PATTERN-sole-parsers` — fixed: Constraints line.
- pattern-fit, LOW, `seam_enforcement_test.go` — fixed: Notes.
- goal-scope, MEDIUM, fans moved out of `Segments` only explained in assumptions — fixed: D5 states it.
- goal-scope, LOW, "no Go change" criterion overclaims — fixed: criterion names the entry code and the key's category.
- leanness, LOW, rationales argue rejected options — fixed: trimmed D2, D4, D5.
- leanness, LOW, assumptions duplicate Scope and D5 — fixed: bullets cut.
- leanness, LOW, acceptance repeats Scope's field list — fixed: refers to Scope.
- leanness, LOW, freshness constraint duplicates D4 — fixed: dropped.
- leanness, LOW, D3 bound sentence — fixed: dropped, since D3 tightens a guard.
- leanness, LOW, Notes duplicate D2 — rejected: they carry plan detail.
- leanness, LOW, Turn 4 pending — fixed by this entry.

## Rejected alternatives

- Keep `Segments` keyed by row name: three rows of a segment would each need an entry, so a new segment would still mean several map entries.
- Pass the segment through each row's `config`: duplicates the row's `segment:` and lets the two disagree.
- Put every segment value, models included, behind `Setting` with recipe-named fallback lists: pulls model resolution, perch-budget fallback and hard-cap validation from `loomengine` into `shedrecipe`.
- Keep fans and auto-accept in `SegmentSettings` and leave `Setting` unused until `row-entry-gate`: ships an untested seam and leaves the plain values on the wiring path step 2 is meant to remove.
- A recipe-level `segments:` block in `shedbuild` holding the fan references once per segment: a new recipe shape for one agreement property a test can hold.
- No cache, a fresh read per call: two rows of one segment could read different fan values if the file changed between them.
- `Routing` taking the `SegmentSettings` map: the status path would need model resolution it does not use.
- Lookup in `loomcli`: batten and other callers would duplicate it; `configreg` already maps module names to specs.
- Degrading `LoadOrTemplate` in the lookup: would answer template defaults for a hub missing its file, where the loom and darn wiring already refuse.
- Migrate run-wide `Env` scalars (`ReviewCirclingCheckpoint`, `DarnVerifyAttempts`, ...) too: not in the operator's decision.
- Fill `Setting` in batten's wiring now: no batten row reads it yet.

## Review rounds

_No rounds yet._

## Question ledger

- Segment key for `Segments` — resolved, auto-pick: segment name.
- How an entry learns its segment — resolved, auto-pick: `Constructor` row identity with name and segment.
- Which values use `Setting` — resolved, auto-pick: reviewer fans, fixer fans, auto-accept.
- Lookup home and strictness — resolved, auto-pick: `configreg`, strict.
- Known-key source — resolved, auto-pick: the module's embedded template, so preserved stale keys (#709) are unknown.
- Reference form — resolved, auto-pick: `module.key` with `_setting` row keys.
- Caching — resolved, auto-pick: one snapshot per lookup value, one lookup per recipe build (changed from "none" at the lens review, for the split-read race).
- Status-path routing input — resolved, auto-pick: budgets keyed by segment, from the path's own degrading reads.
- Null setting values — resolved, auto-pick: absent, auto-accept false.
- Loom fix-fan refusal text — resolved, auto-pick: `<key> in loom.yaml`, a wording change only.
- Review row outside a segment — resolved, auto-pick: refused at construction.
- Batten filling `Setting` — resolved, auto-pick: not in this task.
