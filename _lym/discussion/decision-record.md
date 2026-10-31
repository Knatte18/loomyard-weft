## Goal

`loom` stops naming one recipe and names a contract: a loom recipe takes one board task in, holds a decision record as the task's decided scope, has a writer that writes the code, and ends with Finalize landing one change in the parent.
Shed checks a recipe that declares itself a loom against that contract when it loads it, and the `lym loom` verbs act on loom runs and nowhere else.
`tapestry` and darn are the loom recipes.
What the two write twice today becomes one copy driven by the recipe, darn's record is written mechanically from the board entry, the config splits into a shared `loom.yaml` and a new `tapestry.yaml`, and the Tapestry-Preflight row and the `beam` recipe alias go.

## Scope

In: D1 to D14.
The package docs, the overview doc, recipe headers, stencils and the Code-Review rubric those decisions make stale are updated with them.

Out:

- Pawl: it joins the contract in `pawl-loom-recipe`, which depends on this task; every pawl path stays as it is (D2, D7, D8).
- Hardener: the note `hardener` is parked, and nothing here builds it.
- `lym shed`'s verbs, which stay as they are (D14).
- Finalize's `on_reroute` edges, which the note `finalize-fix-in-place` owns.
- The shared seed's first-row name, which the note `seed-starts-at-start` owns.
- Rewriting the hub's own `profiles.yaml`: the orch does that after landing.
- `batten.yaml`'s arbiter and tackler overrides.

## Decisions

### D1 — A recipe declares the loom contract and Shed checks it at load

A recipe file declares itself a loom with a top-level `contract: loom` key, which Shed's recipe parser in `internal/shedbuild` decodes.
A recipe that declares it is checked when it is built, and refused when it breaks the contract, with a message naming the broken clause and the way forward (PATTERN-refusal-way-forward):

- exactly one record row, whose engine is `DiscussionWrite` or `RecordWrite` (D4);
- exactly one writer row, whose engine is `Implementer` or `DarnWrite`, placed after the record row on the recipe's `on_done` path from its entry;
- a `Finalize` row, from which every `on_done` successor is a terminal;
- a `verify_source` key (D8).

The sets of record, writer and Finalize engines are declared once, beside the registry in `internal/shedrecipe`, so a new loom recipe needs no change to the check.
`tapestry-recipe.yaml` and `darn-recipe.yaml` declare the contract; `pawl-recipe.yaml` and `batten-recipe.yaml` do not.
The one-task input is not a recipe property: the slug the run is seeded with is checked where `lym loom` resolves it, as today.

Rationale: the contract is what makes a recipe a loom, so the loader that builds every recipe is the one place that can hold it for every recipe, present and future.
Bound: the check reads row engines, edges and the recipe's keys only; a recipe without `contract: loom` is not checked and is no loom.

### D2 — The `lym loom` verbs serve loom runs only

Every `lym loom` verb that addresses a run reads the run's recipe from its seed, as `internal/loomcli/arm.go` does today, and refuses a recipe that does not declare the loom contract, naming the `lym shed` verbs as the way forward.
An unseeded run reads as `tapestry`, as today.
Pawl is the one exception, by parent decision (orch for the operator, 2026-10-31): it keeps every `lym loom` verb and every wiring path it uses today, named in one place as the single non-loom recipe `lym loom` still serves, so that batten's Run-Shed, which starts a pawl run with `lym loom start`, and the orch's verbs on beams keep working until `pawl-loom-recipe` lands and removes the exception.

Rationale: the operator's rule is that `lym loom` acts on loom runs and nowhere else, and the task leaves pawl alone.
Bound: the exception admits pawl alone, and no other non-loom recipe; batten seeds no run `lym loom` addresses.

### D3 — A verb that applies to one recipe refuses on another by the recipe's rows

A `lym loom` verb that acts on something only some loom recipes have refuses on a run whose recipe lacks it, saying the verb does not apply to that recipe, and the test is the recipe's rows, never its name:

- `validate-plan`, `insert-card`, `remove-card` and `restore-renumber` refuse on a run whose recipe has no `Plan-Write` row; today they carry no recipe check and fail on the missing plan.
- `lym loom peers` reads each peer's row position against that peer's own recipe: the recipe in the peer's seed when its worktree is on this machine, else the board entry's `recipe` field, empty meaning `tapestry`; today it reads every peer against `tapestry`'s rows (`internal/loomcli/peers.go`).
- `decision add --amends-plan` keeps its existing refusal on a run with no plan, now keyed on the `Plan-Write` row rather than the darn recipe.
- `decision add --pattern-*` stays accepted on every loom recipe: darn's writer lands PATTERN edits in its own commits, so its stencil has it read the record's `## PATTERN changes` and apply each bullet.

Rationale: the entry's three suspected gaps were checked against the code; the plan verbs and peers are real gaps, `--amends-plan` already refuses, and `--pattern-*` is meaningful on darn.
Keying on rows lets a future loom recipe with a plan take the plan verbs with no code change.

### D4 — A Go row writes darn's decision record from the board entry

A new Go producer, row `Record-Write` with engine `RecordWrite` (new), in `internal/loomshed`, writes the decision record with the board entry's body verbatim under `## Goal` and every other required section present and empty, and a support log with its four sections, `## Interview` saying the record was written from the board entry with no interview and `## Review rounds` seeded as the spec requires.
It commits both through the same discussion commit Discussion-Write's output takes, and writes no relevant list.
It reads the entry through the board module, never through an agent.
When a record already exists it leaves both files untouched and returns done, so a resume or a goto back to it never overwrites `Added after Discussion` entries.
`darn-recipe.yaml` places it after Preflight and before Drift-Baseline, and darn's Drift-Baseline and Drift-Verify rows take `subject: decision-record`, as `tapestry`'s do.
The darn writer no longer writes the record or the support log; it reads the record as its scope, the entry's body under `## Goal` and the `Added after Discussion` entries over it.

A darn run in flight when this lands, parked before Darn-Write with no record, halts at Darn-Write's `discussion` gate on the missing record; the halt reason names `lym loom goto --to Record-Write` and `lym loom resume` as the way forward.
`lym loom decision add` before Record-Write has run refuses with its existing missing-record refusal (`internal/loomcli/decision.go`), which names that way forward too.

Rationale: the operator's design makes the record without Discussion mechanical, and a Go row is the only way to write it with no LLM and resume-safely.
The support log keeps the record pair `internal/discussionparser` validates whole, so `decision add`'s append, which validates both files, works on a darn record unchanged.
Bound: the row writes only when no record exists; an empty board body writes an empty `## Goal`, which the validator accepts as it accepts any empty section.

### D5 — One discussion validator and gate for every loom record

`NewDiscussionGate` and `NewDarnDiscussionGate`, and `ValidateLoomDiscussion` and `ValidateDarnDiscussion`, in `internal/loomshed/gates.go`, become one gate and one validator.
Every loom record gets the section check over the record and the support log, the decision-id and acceptance check, and the history check that keeps `Added after Discussion` entries and appended PATTERN bullets.
A record a Discussion writer made also gets the citation check and the relevant-list check; the choice follows the record row's engine in the run's recipe, never the recipe's name.
`lym loom validate-discussion` runs that same validator (Gate Self-Check Parity).
The darn writer row keeps its `discussion` gate as a validate-only guard that the writer left the record intact; it commits nothing, since D4's row committed the record.
A record whose sections bar `## Goal` are empty passes: the parser in `internal/discussionparser` has no empty-section finding, and an empty `## Decisions` has no id for the acceptance check to miss.

Rationale: the two validators differ only in which checks apply, and which apply follows from who wrote the record.
Running the history check on `tapestry` too guards its `Added after Discussion` entries against a fixer that drops one; today only darn has that guard.

### D6 — The outdated-entry check stays in darn's writer, and a writer stop halts the run

The darn writer still checks, before any code, whether landed work or another entry or run overtook or contradicts the board entry, reading the board, its touching entries, `Decision for <slug>:` notes and `lym loom peers`.
It never edits the record: it asks its parent, which records the answer with `lym loom decision add`, and the writer follows the resulting `Added after Discussion` entry.
With no parent it writes the change description as a stop report and ends its turn with no commit on the task branch.
A Darn-Write handoff with no commit past the fork halts the run, with the change description as the halt's reason, in place of the discussion-gate exhaustion that halted it before.

Rationale: judging whether an entry is outdated needs an agent reading code and the board, which a mechanical row cannot do, and the writer is the agent that already reads both.
The record changes only through `lym loom decision add`, the one route the operator named for later decisions.
Bound: a darn task whose right outcome is no code change halts too, and its parent settles it; the halt lets nothing through, since it stops the run before Drift-Verify.

### D7 — The Code-Review perch measures against the decision record

On every loom recipe the Code-Review perch's answer key is the decision record, with the plan beside it where the recipe has one, set through the rows' `artifact_paths` in each recipe file as today.
A darn run no longer writes the board-entry snapshot, `_lym/discussion/board-entry.md`, and darn's Code-Review rows no longer name it; the Code-Review rubric's darn form becomes the decision-record-alone form.
The snapshot writer in `internal/darnengine/snapshot.go` stays for pawl, whose perch reads the snapshot until `pawl-loom-recipe` lands (D2).
A darn run in flight past the old Darn-Write is reviewed against the record its own writer wrote, with its `Added after Discussion` entries.

Rationale: the record is the decided scope, and on darn it holds the entry's body verbatim plus every later decision, so it carries everything the snapshot did.

### D8 — What each recipe writes twice becomes one copy driven by the recipe

- Describe: `DescribeTable` and `DarnDescribeTable` in `internal/landingshed/describe.go` become one table and one stencil, which reads the decision record, the diff and the run records that exist, rendering an absent one as `(none)`; `contracts/stencils/landing/landing-template-describe-darn.md` (removed) goes.
- The description commit becomes one.
- The writer wrappers `internal/loomshed/darnwrite.go` and `internal/loomshed/discussionwrite.go` share one wrapper for running a session and committing its outputs; the prior-work respawn stays darn's.
- The verify source is declared by each loom recipe file as a top-level `verify_source` key, `plan` or `tree_scope`, required by D1, and the wiring reads it in place of a branch on the recipe name; `tree_scope` stays darn's and reads `darn.yaml`. A pawl run keeps today's source through D2's exception.
- `Routing` and `DarnRouting`, `New` and `NewDarn`, and `TapestryRetiredRows` and `DarnRetiredRows` in `internal/loomrecipe` become one builder each, taking the recipe name, with each recipe's retired rows kept as that recipe's data.
- Every remaining branch on `RecipeDarn` or `RecipeTapestry` in `internal/loomcli` reads the recipe's rows or keys instead; pawl's branches stay, under D2's exception.

Rationale: the operator asked for each shared piece written once; driving the differences from the recipe file keeps a third loom recipe a recipe file, not new branches.
A variant keeps only the producers no other recipe uses: Plan, its perch and Webster for `tapestry`, and Record-Write, Darn-Write, tree-scoped verify and the prior-work respawn for darn.

### D9 — Drift-Verify runs before the Code-Review perch in every loom recipe

`tapestry-recipe.yaml` moves Drift-Verify to follow Parent-Align and precede Code-Bouncer, and the perch's exit leads to Describe, as darn's order already does.
Finalize's reroute to Parent-Align then passes Drift-Verify again before the perch; Drift-Verify compares whatever HEAD it meets against the same baseline and is Done on a pass (`internal/driftengine/doc.go`), so the second pass compares the merged code.
A run in flight resumes by row name along the new edges: one parked on Drift-Verify after its review runs the perch once more, and one inside the perch when this lands exits to Describe without Drift-Verify; the orch moves such a run back with `lym loom goto --to Drift-Verify` before resuming it.

Rationale: Drift-Verify can change the code a good deal, and the review must see those changes.

### D10 — Tapestry-Preflight goes; Preflight loads `batcher.yaml` for a recipe with an Implementer row

The Tapestry-Preflight row and its producer, `internal/loomshed/tapestrypreflight.go` (removed), are removed, with its seed check.
Per `CheckSeed`'s own doc in `internal/loomengine/seed.go`, four of its verdicts (missing, undecodable, invalid state, done) are pre-empted by Shed's step-1 status read before any producer runs; what goes with it is the check that the seed is fresh for its first row and the stat race guard, which the operator's design drops.
Its `batcher.yaml` load moves into Preflight, which runs it when the run's recipe holds an `Implementer` row, told so by the wiring.
`tapestry`'s retired-row map sends a run parked on Tapestry-Preflight to Discussion-Write.

Rationale: Preflight keeps the early failure a broken `batcher.yaml` gave, before any LLM spends tokens, while the check stays off for darn and pawl.

### D11 — The `beam` recipe alias goes

The legacy recipe name `beam` and its handling in seed decoding, in `internal/shedrun/seed.go`, are removed.
A seed recording `beam` then reads as the recipe `beam`: `lym shed`'s recipe lookup refuses it as unknown, naming the available recipes, and `lym loom` refuses it as a non-loom recipe (D2).

Rationale: per the board entry, no run seed records `beam`.

### D12 — `loom.yaml` holds what every loom recipe shares, `tapestry.yaml` what only `tapestry` has

- `loom.yaml` keeps `discussion_*`, `code_*`, `review_*`, `friction*`, and the keys every review perch or loom run reads: `reviewer_model`, `fixer_model`, `judge`, `seed`, `driver` and `step_idle_timeout_min`.
- A new per-worktree config module `tapestry`, file `tapestry.yaml`, takes `plan`, `plan_timeout_min`, `implementer`, `plan_lenses` and every other `plan_*` key.
- `darn.yaml` keeps `writer`, `writer_timeout_min`, `tree_scope` and `verify_attempts`.
- A `tapestry` key whose empty or null value falls back today to a shared key keeps that fallback across modules: `plan_max_bounces: null` takes `loom.review_max_bounces`, and an empty `plan_reviewer_model` takes `loom.reviewer_model`.
- Each moved key is declared in `KeyMoves` in `internal/configreg/keymove.go`. The reconcile moves a tuned value in a hub's base files to `tapestry.yaml`, and a profile block or `run_settings` naming the old module is translated when it is read, with nothing rewritten; the orch rewrites `profiles.yaml` to the new module afterwards.
- Every `module.key` reference in the recipe files to a moved key names the new module.

Rationale: the operator's split; `KeyMoves` is the repo's declared route for a key changing module, so no hub loses a tuned value and the run writes no migration of its own.

### D13 — The arbiter, tackler and driver-kind keys live in `shed.yaml` alone

Parent decision (orch, 2026-10-31): `arbiter`, `arbiter_timeout_min`, `arbiter_max_spawns`, `tackler`, `tackler_timeout_min`, `tackler_max_spawns` and `driver_kind` are removed from `loom.yaml` and `darn.yaml` and live in `shed.yaml` alone; a recipe that needs other values sets them in its profile block under the `shed` module.
The removed keys are retired, not moved, since moving them into `shed.yaml` would widen a loom-only value to every recipe.
A hub base file's value is dropped by the reconcile.
A profile block or `run_settings` that names one under `loom` or `darn` is refused when the run starts, naming the key and the way forward, to set it under `shed` in the profile block; refusing rather than ignoring keeps a run pinned to a driver from silently running another.
The per-recipe override read in `internal/loomengine/config.go` that loads these keys from `loom.yaml` or `darn.yaml` goes, and every recipe, pawl included, reads them from `shed.yaml` and its profile block.
`batten.yaml`'s overrides stay, outside this decision.

Rationale: settles the first of the entry's Open items as the parent decided it.

### D14 — `lym loom` and `lym shed` both stay

Parent decision (orch, 2026-10-31): `lym loom` keeps its run verbs for loom runs, and `lym shed` keeps its generic `run`, `step`, `status`, `pause` and `goto` for every recipe, batten and pawl included; neither is folded into the other.

Rationale: settles the last of the entry's Open items as the parent decided it.

### Added after Discussion (parent, 2026-10-31): The contract names a build segment, not a writer

Decision: The loom contract says: a record before, a build segment, and the landing rows after. The build segment is the rows a variant owns that turn the record into the change: Plan-Write, its perch and Webster in tapestry, Darn-Write in darn. The contract and the code do not call this slot 'the writer', since Discussion-Write, Plan-Write, Describe and the fixers are writers too.

Rationale: Operator, 2026-10-31: 'writer' names many agents; tapestry's plan is part of making the change.

## Constraints

- PATTERN-shed-recipe-registry: `RecordWrite` is one more `Constructor` in the one map in `internal/shedrecipe`; the contract's engine sets sit beside it.
- PATTERN-sole-parsers: `contract` and `verify_source` are decoded only by the recipe parser in `internal/shedbuild`, and the record and support log only by `internal/discussionparser`.
- PATTERN-refusal-way-forward: D1's contract refusal, D2's non-loom refusal, D3's verb refusals, D4's missing-record halt and D13's retired-key refusal each name the way forward, and a test reaches each.
- PATTERN-gate-slot and the Gate Self-Check Parity Invariant: D5's gate and `lym loom validate-discussion` call one function.
- PATTERN-config-layers: the split adds a module through the config registry and template, and moves keys only through `KeyMoves` and `hubreconcile.Ensure`.
- PATTERN-sparse-hub-base: a moved key lands in `tapestry.yaml` only where the hub had tuned it.
- PATTERN-stencil-ownership: the one Describe stencil and the darn writer stencil are read at call time; the writer stencil points at the discussion spec and restates none of it.
- PATTERN-tolerant-records: the seed keeps accepting unknown keys once the alias goes.
- PATTERN-shed-stop-chain changes with D13.
- Recipe row names are durable on-disk identities: a removed row is mapped onto its successor in the recipe's retired-row data.
- The shared seed still names Preflight as the first row until `seed-starts-at-start` lands, so both loom recipes keep Preflight as their entry row.

## Auto-mode assumptions

- D4: the row name `Record-Write`, its place after Preflight, and the support log it writes.
- D5: the history check on `tapestry` records too.
- D6: the no-commit handoff as the writer's halt.
- D8: `verify_source` as a recipe-file key.
- D10: the `batcher.yaml` load in Preflight rather than the Webster row.
- D12: which shared keys stay in `loom.yaml`.
- D13: retiring rather than moving the removed keys, refusing them in profiles and run settings, and leaving `batten.yaml` alone.

## Open risks

- D13 leaves `batten.yaml`'s overrides, while the parent's "in `shed.yaml` alone" may have meant every module; the parent should confirm.
- The note `finalize-fix-in-place` removes Finalize's `on_reroute`; D9 changes `tapestry`'s edges after Parent-Align, the reroute's target, so the later task must read the new order.
- D13 refuses a profile block or task `run_settings` that still names `arbiter*`, `tackler*` or `driver_kind` under `loom` or `darn`; the orch must move each under `shed` when it rewrites `profiles.yaml`, or such runs refuse to start.
- Runs in flight when this lands may need the orch's `goto`: a darn run with no record (D4) and a `tapestry` run inside the Code-Review perch (D9).

## Acceptance criteria

- D1: loading `tapestry` and darn passes the contract check; a recipe declaring `contract: loom` without a record row, without a writer row, with the writer before the record row, without Finalize ending its `on_done` path, or without `verify_source` is refused, naming the clause and the way forward.
- D2: a `lym loom` verb on a run whose seed names a non-loom recipe other than pawl refuses, naming the `lym shed` verbs; a pawl run is served as today.
- D3: `validate-plan`, `insert-card`, `remove-card` and `restore-renumber` on a darn run refuse, saying they do not apply; `lym loom peers` reports a darn peer's row against the darn recipe, from its seed or its entry's `recipe` field; `decision add --amends-plan` on a darn run refuses; `decision add --pattern-*` on a darn run appends the bullet, and the darn writer stencil applies `## PATTERN changes` bullets.
- D4: a darn run writes and commits a record with the board body verbatim under `## Goal` and the other sections empty, and a support log, before Drift-Baseline; a resume with an existing record leaves both byte for byte; `decision add` appends to that record; darn's Drift rows read the record as their subject; a darn run with no record halts at Darn-Write naming `goto --to Record-Write`.
- D5: the one validator accepts the D4 record; a Discussion-written record citing a missing path or with a malformed relevant list is refused; a record that drops an `Added after Discussion` entry is refused on both recipes; the gate and `validate-discussion` report the same findings.
- D6: the darn writer stencil sends an outdated-entry answer to the parent for `lym loom decision add` and never edits the record; a Darn-Write handoff with no commit past the fork halts the run with the description as its reason.
- D7: darn's Code-Review rows name the decision record as the answer key, and a darn run writes no `board-entry.md`; a pawl run still writes it.
- D8: one Describe table and stencil serve both recipes; the wiring holds no `tapestry` or darn recipe-name branch; the verify source follows `verify_source`.
- D9: `tapestry`'s routing runs Parent-Align, Drift-Verify, then the Code-Review perch, then Describe, and Finalize's reroute reaches Drift-Verify again.
- D10: no Tapestry-Preflight row exists; a broken `batcher.yaml` fails Preflight on a `tapestry` run and not on a darn run; a run parked on Tapestry-Preflight resumes at Discussion-Write.
- D11: `lym shed` refuses a seed recording `beam` as an unknown recipe, and `lym loom` refuses it as a non-loom recipe.
- D12: the `plan_*` and `implementer` keys load from `tapestry.yaml`; a hub `loom.yaml` that tuned `plan_max_bounces` carries the value to `tapestry.yaml` after reconcile; a profile naming `loom.plan_max_bounces` still applies; `tapestry.plan_max_bounces: null` takes `loom.review_max_bounces`.
- D13: `loom.yaml` and `darn.yaml` hold no `arbiter*`, `tackler*` or `driver_kind` key; a run reads those from `shed.yaml` and its profile block; a profile naming `loom.driver_kind` refuses the run, naming the `shed` module.
- D14: `lym shed` keeps its five verbs for every recipe and `lym loom` keeps its verbs.

## PATTERN changes

- `PATTERN-loom-contract` — adds: Adding a loom recipe: it declares `contract: loom`, checked by Shed's recipe loader; `lym loom` verbs serve loom runs only, bar pawl until it joins; a verb for some loom recipes refuses on others by the recipe's rows, never its name.
- `PATTERN-shed-stop-chain` — changes: A shed run's stop: rule table, tackler, arbiter, then parent, for every recipe; a blocked stop the driver's arbiter seam marks as judged first, by its row and reason, goes rule table, arbiter, then parent; arbiter keys in shed.yaml, a loom recipe's own through its profile block.

## Notes for the plan writer

- D12's per-worktree module needs its own template file, registry entry and config CLI, like `darn`'s in `internal/darnengine`.
