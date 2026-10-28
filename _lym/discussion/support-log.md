# Support log — review-setup-tuning

## Interview

### Turn 1 — board and context (autonomous)

The board entry carries the design; no `Decision for review-setup-tuning:` note exists.
Touching entries: `validator-checks` (running, machine checks for the same plan themes `repo-invariants` reads), `turn-batching` (depends on this task, edits stencil sentences), `review-inline-code` and `fixers-on-sonnet` (no overlap).
The audit report gives the measurements each lens change rests on.

### Turn 2 — where the lenses live (autonomous)

The entry says "the lens stencils under `contracts/stencils`", but every lens and fan is prose in `internal/lenses/template.yaml`, with a `csharp` layer repeating `review-code`.
A search of `contracts/` finds no changed lens name, so the change is the template, its tests and the docs that name lenses.
`goal-scope` and `board-fit` sit in `review-discussion` as well as `twill-discussion`.
`way-forward` is in `twill-plan` too, which the audit says to keep.

### Turn 3 — how a profile sets the code fan (autonomous)

`code_reviewer_fan` is a `CategoryGuard` key in loom's `ConfigSpec`.
Seed-Child writes a profile block into the pair's local layer, and configreg checks profile blocks through `checkLocalDocument`, which runs `configengine.ValidateLocal` and refuses a guard key.
So the entry's profile line cannot be met with config values alone; D5 reclassifies the one key, with its bound.

### Turn 4 — live hub files (autonomous)

The hub's `profiles.yaml` is seed-only and differs from the template (no `pawl` blocks, `docs` with `code_max_bounces: 0`); the hub's `lenses.yaml` tunes only `docs-consistency`.
`docs/dotnet-hub.md` documents a Models hub fan naming `code-pattern-fit`.
Both go to Open risks for the parent, since no landing writes a hub's Bolt config.

### Turn 5 — lens review

Every fork returned a report.

- goal-scope, MEDIUM, `board-scope` in `review-discussion` is unlicensed and the audit says keep four: fixed, `review-discussion` drops the pair with no replacement (D1).
- goal-scope, MEDIUM `decision`, whether the profile line means the live hub file too: kept as an Open risk for the parent; the record cannot write Bolt config.
- goal-scope, LOW, D5 opens a guard the entry does not name: rejected, D5 states it is forced and bounded.
- interactions, walkthrough and leanness, MEDIUM, the `PATTERN-config-layers` `changes:` bullet repeats the unchanged overview line and would trip `pattern-line-unprescribed` and the code rubric's declared-change check: fixed, section and its note dropped; D7 says the overview line is unchanged.
- interactions, LOW, D3 silently drops the "A broken invariant" and "A pattern the parent changed" names: fixed, D3 says they are dropped since no rubric carries them.
- walkthrough, MEDIUM, D5 also admits the retired darn keys moved onto `code_reviewer_fan`: fixed in D5 and its acceptance line; confirmed against `runsettings_test.go`'s key-move rows.
- walkthrough, LOW, no criterion that the light round runs solo rather than fanned: fixed in D6's acceptance line.
- leanness, MEDIUM, Auto-mode assumptions repeat the decisions: fixed, cut to the picks the entry does not license, each a pointer.
- leanness, LOW, audit figures restated in rationales: fixed, rationales cite audit sections.
- leanness, LOW, Scope's docs bullet duplicates D7: fixed.
- leanness, LOW, fixture-name note is plan detail: fixed, note dropped.
- leanness, LOW, the contracts/stencils constraint: no change, the lens kept it.
- board-fit, LOW, `repo-invariants` overlaps `validator-checks`: rejected, the Constraints line already covers it.
- existing-paths, MEDIUM, configsync's former-value strip could remove D6's `""` from the local layer: rejected after checking, `ReconcileAll` reconciles local files with a nil strip; the fact is in Constraints.
- existing-paths, LOW, comments in loom's `config.go`, `template.yaml` and `configcli/doc.go` call the fan keys closed: fixed in D7.
- existing-paths, LOW, a run in flight across the deploy may get one stale `exclude_lenses` list: rejected, one round's harmless exclusion with no design change.
- bound-breaker, MEDIUM, a hub fan naming a deleted lens fails every loom config load, not only Code-Review: fixed, Open risk states the reach and says the hub edit comes before the deploy.
- bound-breaker, MEDIUM, D5's fan-name bound holds only at load: fixed, D5 states the write is unchecked and the load refuses with the key and fix.
- bound-breaker, LOW, D5 silent on a run's own agent: fixed, D5 bounds the local-layer route against `code_max_bounces` and states `run_settings` reach a pair only at Seed-Child.

## Rejected alternatives

- A separate profile layer between the worktree and local layers that may hold guard keys: keeps the guard but adds a config layer for one key.
- Reclassifying all three reviewer fan keys, or all six fan keys: opens guards nobody asked to open.
- Making the template's code fan solo and letting `solid` and `wide` turn the fan back on: the same guard blocks `solid` and `wide`.
- A write-time fan-name check in loom's local value check: the value check sees only the document, not `lenses.yaml`, and the load already refuses a wrong name.
- Keeping `goal-scope`, `board-fit` and `code-pattern-fit` defined but out of every fan, or a retired-name list that `Load` drops from hub fans: keeps dead prose or adds a mechanism for one hub's one fan, which a one-line hub edit fixes.
- Putting `board-scope` into `review-discussion` too: the entry names only the writer fan, and the audit keeps the reviewer's four.
- Adding `repo-invariants` to `twill-plan`: the audit keeps the writer's five and asks for a lens the writer fan lacks.
- Making `code_fixer_fan` solo in `light` and `docs` too: the entry names the reviewer only.

## Review rounds

### Round 1 — APPROVED, six findings fixed

- F1, MEDIUM: `board-scope` stays a procedure lens ending in its list, and the template header and `TestConfigTemplate`'s emphasis-line exemption widen to name it; the Constraint and D1's acceptance line follow.
  The first fix ended it with the emphasis line, which the self-check found contradicts its report-only rule.
- F2, LOW: `speed`'s loom block runs the code review solo too, listed under Auto-mode assumptions and named in Goal, Scope and Open risk 1; its `darn` and `pawl` blocks run no code round and set no fan.
- F3, LOW: D5 and its acceptance line name loom's retired `webster_reviewer_fan` and `webster_fan` as admitted routes under the same bound.
- F4, LOW: D7 drops loom's `template.yaml` and `internal/configcli/doc.go`, which hold no closed-to-local text.
- F5, LOW: the deleted-lens Open risk separates a reviewer or fixer fan key, failing at load, from `discussion_lenses` and `plan_lenses`, failing at the writer lead.
- F6, NIT: D6 says `solid` and `wide` take the hub's `code_reviewer_fan`.

### Round 2 — APPROVED, four findings fixed

- F1, MEDIUM: `way-forward` leaves `review-plan`, its one procedure fan, so D2 has its prose gain the emphasis line, as `coverage`, `row-validity` and `forced-edits` end, and D2's acceptance line checks it.
- F2, LOW: D5's third Bound bullet says the local layer can point the code review at any defined fan, not only solo, and names the whole-diff read under the rubric as the floor.
- F3, LOW: Auto-mode assumptions list D1's `review-discussion` pick, dropping `goal-scope` and `board-fit` with no replacement.
- F4, NIT: D5 says loom's `MigrateConfig` renames the retired webster keys in `PrepareRunSettings` and in a reconcile, and a hand-edited local file holding one is refused at load until a reconcile migrates it.

## Question ledger

- Q1 (auto-pick): merged lens name — `board-scope`. Short and says what it owns.
- Q2 (auto-pick): new plan lens name — `repo-invariants`. Matches the audit's wording.
- Q3 (auto-pick, revised at lens review): `review-discussion` drops the pair with no replacement; the dormant `discussion` fan takes `board-scope`.
- Q4 (auto-pick): how a profile sets a guard key — reclassify `code_reviewer_fan` to `CategoryNone` with a stated bound (D5).
- Q5 (auto-pick): fixer fan in `light` and `docs` — unchanged.
- Q6 (auto-pick): hub-side config — the live `profiles.yaml` and any hub fan naming a deleted lens go to Open risks for the parent and operator.
- Q7 (auto-pick): `code-tests` and the rubric — narrow the lens, leave the rubric's "Test economy" to the lead.
