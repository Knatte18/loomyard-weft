## Goal

`loom` stops naming one recipe and names a contract: a loom recipe takes one board task in, holds a decision record as the task's decided scope, has a build segment that turns the record into the change, and ends with Finalize landing one change in the parent.
Shed checks a recipe that declares itself a loom against that contract when it loads it, and the `lym loom` verbs act on loom runs and nowhere else.
`tapestry` and darn are the loom recipes.
What the two write twice today becomes one copy driven by the recipe, darn's record is written mechanically from the board entry, the config splits into a shared `loom.yaml` and a new `tapestry.yaml`, and the Tapestry-Preflight row and the `beam` recipe alias go.

## Scope

In: D1 to D14.
The package docs, the overview doc, recipe headers, stencils and the Code-Review rubric those decisions make stale are updated with them.
So are the pinned specs they change:

- `contracts/specs/loom-discussion-spec.md`: the history check runs on every loom record, no longer only darn's, the citation and relevant-list checks follow the record row's engine (D5), and a top-level fence opens and closes by CommonMark's rule, as D4 states it, which the section check honours too (D4).
- `contracts/specs/shed-recipe-spec.md`: the top-level `contract` and `verify_source` keys (D1, D8).
- `contracts/specs/loom-status-spec.md`: the Tapestry-Preflight row and `CheckSeed`'s verdicts (D10).

Out:

- Pawl: it joins the contract in `pawl-loom-recipe`, which depends on this task; every pawl path stays as it is (D2, D7, D8), bar D13's override move, which applies to pawl too by parent decision and is the one pawl path this task changes.
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
- a build segment, the rows the recipe owns that turn the record into the change, holding exactly one row whose engine is `Implementer` or `DarnWrite`, placed after the record row on the recipe's `on_done` path from its entry;
- a `Finalize` row, from which every `on_done` successor is a terminal;
- a `verify_source` key (D8).

The sets of record, build and Finalize engines are declared once, beside the registry in `internal/shedrecipe`, so a new loom recipe needs no change to the check.
`tapestry-recipe.yaml` and `darn-recipe.yaml` declare the contract; `pawl-recipe.yaml` and `batten-recipe.yaml` do not.
The one-task input is not a recipe property: the slug the run is seeded with is checked where `lym loom` resolves it, as today.

Rationale: the contract is what makes a recipe a loom, so the loader that builds every recipe is the one place that can hold it for every recipe, present and future.
Bound: the check reads row engines, edges and the recipe's keys only; a recipe without `contract: loom` is not checked and is no loom.

### D2 — The `lym loom` verbs serve loom runs only

Every `lym loom` verb that addresses a run reads the run's recipe from its seed, as `internal/loomcli/arm.go` does today, and refuses a recipe that does not declare the loom contract, naming the `lym shed` verbs as the way forward.
The refusal sits in the `lym loom` command path, never in `loomcli.ArmAt`, which `lym shed` shares to arm `tapestry`, darn and pawl runs (`internal/shedcli/table.go`).
An unseeded run reads as `tapestry`, as today.
Pawl is the one exception, by parent decision (orch for the operator, 2026-10-31): it keeps every `lym loom` verb and every wiring path it uses today, bar D13's override move, which applies to pawl too by parent decision, named in one place as the single non-loom recipe `lym loom` still serves, so that batten's Run-Shed, which starts a pawl run with `lym loom start`, and the orch's verbs on beams keep working until `pawl-loom-recipe` lands and removes the exception.

Rationale: the operator's rule is that `lym loom` acts on loom runs and nowhere else, and the task leaves pawl alone.
Bound: the exception admits pawl alone, and no other non-loom recipe; batten seeds no run `lym loom` addresses.

### D3 — A verb that applies to one recipe refuses on another by the recipe's rows

A `lym loom` verb that acts on something only some loom recipes have refuses on a run whose recipe lacks it, saying the verb does not apply to that recipe, and the test is the recipe's rows, never its name:

- `validate-plan`, `insert-card`, `remove-card` and `restore-renumber` refuse on a run whose recipe has no `Plan-Write` row; today they carry no recipe check and fail on the missing plan.
- `lym loom peers` reads each peer's row position against that peer's own recipe: the recipe in the peer's seed when its worktree is on this machine, else the recipe Seed-Child seeds for its board entry, pawl for a project entry and otherwise the entry's `recipe` field, empty meaning `tapestry` (`boardRecipe` in `internal/battencli/wire.go`); today it reads every peer against `tapestry`'s rows (`internal/loomcli/peers.go`).
- `decision add --amends-plan` keeps its existing refusal on a run with no plan, now keyed on the `Plan-Write` row rather than the darn recipe.
- `decision add --pattern-*` stays accepted on every loom recipe: darn's writer lands PATTERN edits in its own commits, so its stencil has it read the record's `## PATTERN changes` and apply each bullet.

Rationale: the entry's three suspected gaps were checked against the code; the plan verbs and peers are real gaps, `--amends-plan` already refuses, and `--pattern-*` is meaningful on darn.
Keying on rows lets a future loom recipe with a plan take the plan verbs with no code change.

### D4 — A Go row writes darn's decision record from the board entry

A new Go producer, row `Record-Write` with engine `RecordWrite` (new), in `internal/loomshed`, writes the decision record with the board entry's body verbatim under `## Goal` and every other required section present and empty, and a support log with its four sections, `## Interview` saying the record was written from the board entry with no interview and `## Review rounds` seeded as the spec requires.
The body sits in one fenced `markdown` block whose backtick fence is max(3, one longer than the longest backtick run in the body) backticks long, so a body with no backticks or with inline code only still gets a fence CommonMark opens, and no line of the body, a heading such as `## Decisions` or `## Open risks` included, starts a section: the discussion spec reads no fenced line, so `decision add`, the carry-over and the PATTERN parse act only on the record's own sections.
That holds only if the parser's fence rule is CommonMark's, so `fencedLines` in `internal/discussionparser/append.go`, which today flips on every line whose right-trimmed text starts with three backticks or tildes, whatever its length, its indent or what follows the run, takes CommonMark's rule for a fenced code block at the document's top level; the parser reads no list item or blockquote, so a fence nested in one is read as a top-level line.
A fence opens on a line of up to three spaces of indent followed by a run of at least three backticks or tildes, and a backtick run whose info string holds a backtick opens none.
It closes only on a line of up to three spaces of indent, a run of the opening character at least as long as the opening run, and nothing after the run but spaces or tabs, so a line such as ```` ```js ```` inside an open fence is body text.
A line indented four spaces or more neither opens nor closes a fence.
A code block or an unclosed fence inside the body is then body text, and the outer fence closes only on its own closing line.
`missingSections` in `internal/discussionparser/validate.go`, which today matches every line whatever fence it sits in, counts a required heading only on a line outside a fence, by the same rule, so a fenced body heading never stands in for a record section the writer removed.
It commits both through the same discussion commit Discussion-Write's output takes, and writes no relevant list.
It reads the entry through the board module, never through an agent.
It writes whichever of the two files is missing and never overwrites one that exists, writing each through `fsx.AtomicWriteBytes` (`internal/fsx/fsx.go`), which writes a `.tmp-` sibling and renames it into place, so across a process crash a file that exists is whole; the helper does not fsync, so an OS crash can still leave a damaged file, which D5's guard halts on.
Before it writes, it removes every `.tmp-` file a crash left in the discussion directory, so the discussion commit, which stages the whole discussion directory, never takes one; then it commits the pair when it holds uncommitted changes, so a resume after a crash between the writes or before the commit completes the pair, and a resume or a goto back to it never overwrites `Added after Discussion` entries.
`darn-recipe.yaml` places it after Preflight and before Drift-Baseline, and darn's Drift-Baseline and Drift-Verify rows take `subject: decision-record`, as `tapestry`'s do.
The darn writer no longer writes the record or the support log; it reads the record as its scope, the entry's body under `## Goal` and the `Added after Discussion` entries over it.

A darn run in flight when this lands has no record, and its handling follows the row it is parked on:

- Preflight: it runs Record-Write next on resume, as a new run does.
- Drift-Baseline or Drift-Await: no writer has run, and the baseline capture's precondition holds HEAD at the branch point (`CheckMainCapture`, `internal/driftengine/doc.go`), so the orch moves the run with `lym loom goto --to Record-Write` before resuming it, and Drift-Baseline passes again; Record-Write's commit lands in the `_lym` weft repository, never on the task branch.
  Resumed without that goto, Drift-Baseline spawns its agent pointed at a record that does not exist, with no route to the board entry.
  The agent either returns Stuck, whose drift ways forward end in `lym loom resume` and meet the same missing record again, or records a scope from no subject, which then stands; either way the run's way forward is the goto to Record-Write this bullet names, and a run that reaches Darn-Write meets the missing-record refusal below.
- Darn-Write, or any run resumed without that goto: before spawning its session, Darn-Write refuses a missing record, Stuck, naming one way forward whatever the task branch holds: `lym loom goto --to Record-Write` then `lym loom resume`, bar the committed-record case below.
  On a task branch with no commit past the branch point, Drift-Baseline passes again keeping the scope record and capture of its earlier pass.
  The capture stands since it read code at the branch point; the scope stands as the earlier agent recorded it, from the board entry on a run that passed Drift-Baseline before landing, or from no subject on one resumed without the goto, a thin scope Drift-Verify then measures against.
  On a task branch with commits past the branch point, the run passed Drift-Await, so its scope record is off or holds `awaited_at`.
  Per `internal/driftengine/doc.go`, Drift-Baseline then takes no capture whatever HEAD is, and spawns its agent only when the scope record or `baseline.md` is missing, an agent that now reads the record Record-Write wrote; Drift-Await is Done whatever HEAD is, so the resume reaches Darn-Write with no capture on changed code.
  When the `_lym` weft repository's HEAD holds a committed record, the working copy was removed after Record-Write ran, and a record written again from the board entry would drop its committed `Added after Discussion` entries, which D5's history check then refuses; the refusal then names restoring the record from that HEAD, then `lym loom resume`, in place of the goto.

`lym loom decision add` on a darn run with no record refuses with its missing-record refusal (`internal/loomcli/decision.go`), whose darn text in `decisionWayNoRecord` drops the Darn-Write session and the board-note route, says the record exists once Record-Write has run, and names the way forward by the run's current row.
At or before Record-Write it names `lym loom resume`, then re-running the verb.
On any later row, a run in flight when this lands parked on Drift-Baseline, Drift-Await or Darn-Write, it names `lym loom goto --to Record-Write` then `lym loom resume`, the route above, then re-running the verb, or, when the `_lym` weft repository's HEAD holds a committed record, restoring it from that HEAD, then re-running the verb.
Its darn refusals there for a record not committed at HEAD, a record that differs from its committed copy, and a record missing a historical entry drop `decisionWayDarnWriter`, which names a Darn-Write session that no longer writes the record:

- Not committed: only a crash inside Record-Write leaves this, with the run's current row still Record-Write, so the way forward is `lym loom resume`, which commits the pair.
- Differs from its committed copy: either `decision add` appended an entry and its commit failed, the refusal that names `lym fabric commit`, or an edit outside `decision add` changed the record.
  When the working record differs from the committed one only by the `Added after Discussion` entries or `## PATTERN changes` bullets a failed call's commit-failed refusal reported, the way forward is `lym fabric commit` in the `_lym` weft repository, which records that call's entries, then re-running the refused call alone, never the failed one, whose entries the commit already holds; the caller judges the match, so an appended entry no failed call reported takes the restore below; otherwise it is restoring the record to its committed copy in the `_lym` weft repository, then re-running the verb.
- Missing a historical entry: the committed record dropped it, so the way forward is restoring the entry and committing the record in the `_lym` weft repository, then re-running the verb.

Its post-append refusal for a committed record lacking the new entry drops the Darn-Write session from `decisionWayEntryLost` and names re-running `lym loom decision add`.

Rationale: the operator's design makes the record without Discussion mechanical, and a Go row is the only way to write it with no LLM and resume-safely.
The support log keeps the record pair `internal/discussionparser` validates whole, so `decision add`'s append, which validates both files, works on a darn record unchanged.
Bound: the row never overwrites a file that exists, and the board body sits fenced under `## Goal` in a fence of at least three backticks and one longer than the body's longest backtick run, a fence the parser closes by D4's fence rule; an empty board body writes an empty fenced block, which the validator accepts as it accepts any section.

### D5 — One discussion validator and gate for every loom record

`NewDiscussionGate` and `NewDarnDiscussionGate`, and `ValidateLoomDiscussion` and `ValidateDarnDiscussion`, in `internal/loomshed/gates.go`, become one gate and one validator.
Every loom record gets the section check over the record and the support log, the decision-id and acceptance check, and the history check that keeps `Added after Discussion` entries and appended PATTERN bullets.
A record a Discussion writer made also gets the citation check and the relevant-list check; the choice follows the record row's engine in the run's recipe, never the recipe's name.
`lym loom validate-discussion` runs that same validator (Gate Self-Check Parity).
Darn-Write keeps a validate-only guard that the writer left the record and the support log intact, running the one validator once its session returns; it commits nothing, since D4's row committed the record.
The guard never re-prompts the session: Record-Write never overwrites a file that exists (D4), so a file that fails the guard stays failing, and a re-prompt could only push the writer to edit a record D6 bars it from editing.
A guard failure halts the run, Stuck, the reason naming the findings and the way forward: restore each failing file, in the `_lym` weft repository, to its latest version that `lym loom validate-discussion` accepts, the committed copy when the damage is uncommitted, commit it with `lym fabric commit`, then `lym loom resume`.
Restoring keeps every `Added after Discussion` entry the record held before the damage, which removing the file for Record-Write to write again from the board entry would drop, and the history check would then refuse.
A stop marker D6 reads halts the run ahead of the guard.
D4's fence rule in the section check applies to every loom record: a Discussion-written record with an unclosed fence, which CommonMark runs to the end of the document, now reports every later required section missing, and the gate re-prompts its writer to close the fence.
A record whose sections bar `## Goal` are empty passes: the parser in `internal/discussionparser` has no empty-section finding, and an empty `## Decisions` has no id for the acceptance check to miss.

Rationale: the two validators differ only in which checks apply, and which apply follows from who wrote the record.
Running the history check on `tapestry` too guards its `Added after Discussion` entries against a fixer that drops one; today only darn has that guard.

### D6 — The outdated-entry check stays in darn's writer, and a writer stop halts the run

The darn writer still checks, before any code, whether landed work or another entry or run overtook or contradicts the board entry, reading the board, its touching entries, `Decision for <slug>:` notes and `lym loom peers`.
It never edits the record: it asks its parent, which records the answer with `lym loom decision add`, and the writer follows the resulting `Added after Discussion` entry.
With no parent, a `Decision for <slug>:` note does not settle a stop: an outdated entry with no `Added after Discussion` entry answering it stops the writer again, whatever note answers it, so every answer reaches the record.
The darn writer stencil drops its route of writing a note's answer into the record, and its stop report names `lym loom decision add` then `lym loom resume` as the way forward, never a board note.
With no parent it writes the change description as a stop report, in the form the final-summary spec requires of every change description so the `description` gate takes it as any other, writes a stop marker in the run's directory, and ends its turn with no commit.
Darn-Write removes the stop marker before it spawns each session and reads it once the session returns, a Done handoff or a gate-failed Stuck alike: a marker present halts the run, Stuck, with the change description as the halt's reason, in place of the discussion-gate exhaustion that halted it before and ahead of D5's guard.
So a stop report the `verify` gate refuses, on a base tree its command fails, or that the `description` gate refuses, still halts with the description as its reason.
The halt's way forward is the orch or the operator recording the answer with `lym loom decision add`, then `lym loom resume`, so the resumed writer and the Code-Review perch both read it as an `Added after Discussion` entry.

Rationale: judging whether an entry is outdated needs an agent reading code and the board, which a mechanical row cannot do, and the writer is the agent that already reads both.
The record changes only through `lym loom decision add`, the one route the operator named for later decisions.
Bound: a darn task whose right outcome is no code change halts too, and its parent settles it; the halt lets nothing through, since Darn-Write returns Stuck on the marker before the run reaches Drift-Verify, whatever earlier spawns of the row committed.

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
Per `CheckSeed`'s own doc in `internal/loomengine/seed.go`, four of its verdicts (missing, undecodable, invalid state, done) are pre-empted by Shed's step-1 status read before any producer runs.
What goes beyond those four, which the operator's design drops, is the stat race guard and the half-finished checks: history naming a row other than the two Preflights, a set `start_sha`, and `pause_requested` (`internal/loomengine/coherence.go`).
Its `batcher.yaml` load moves into Preflight, which runs it when the run's recipe holds an `Implementer` row, told so by the wiring.
`tapestry`'s retired-row map sends a run parked on Tapestry-Preflight to Preflight, so the `batcher.yaml` load runs again on resume.

Rationale: Preflight keeps the early failure a broken `batcher.yaml` gave, before any LLM spends tokens, while the check stays off for darn and pawl.
Bound: with the half-finished checks gone, a run whose current row is Preflight but whose status holds history past Preflight, a set `start_sha` or `pause_requested` proceeds into the row after Preflight.
A run's current row is Preflight on a fresh seed, on a resume of a run blocked at Preflight, on a resume of a run parked on Tapestry-Preflight that the retired-row map sends there, whose Tapestry-Preflight history entry the old check tolerated, or after `lym loom goto --to Preflight`, which Tapestry-Preflight already waived as a deliberate re-entry (`waivesHalfFinished` in `internal/loomshed/tapestrypreflight.go`); a repeated `lym loom start` never rewrites a seed, since `WriteSeed` refuses one that disagrees with the existing seed (`internal/loomcli/sharedbootstrap.go`).
What the dropped checks still caught is a status file edited outside those routes, or, for the stat race guard, changed between Shed's read and the row's own.

### D11 — The `beam` recipe alias goes

The legacy recipe name `beam` and its handling in seed decoding, in `internal/shedrun/seed.go`, are removed.
A seed recording `beam` then reads as the recipe `beam`: `lym shed`'s recipe lookup refuses it as unknown, naming the available recipes, and `lym loom` refuses it as a non-loom recipe (D2).

Rationale: per the board entry, no run seed records `beam`.

### D12 — `loom.yaml` holds what every loom recipe shares, `tapestry.yaml` what only `tapestry` has

- `loom.yaml` keeps `discussion_*`, `code_*`, `review_*`, `friction*`, and the keys every review perch or loom run reads: `reviewer_model`, `fixer_model`, `judge`, `seed`, `driver` and `step_idle_timeout_min`.
- A new per-worktree config module `tapestry`, file `tapestry.yaml`, takes `plan`, `plan_timeout_min`, `implementer`, `plan_lenses` and every other `plan_*` key.
- `darn.yaml` keeps `writer`, `writer_timeout_min`, `tree_scope` and `verify_attempts`.
- A `tapestry` key whose empty or null value falls back today to a shared key keeps that fallback across modules: `plan_max_bounces: null` takes `loom.review_max_bounces`, and an empty `plan_reviewer_model` takes `loom.reviewer_model`.
- The plan perch's budget and hard cap are resolved and validated by the `tapestry` module's config load, which loads `loom.yaml`'s run-wide `review_max_bounces`, `review_hard_max_bounces` and `review_amend_max_bounces` beside `tapestry.yaml` and runs `reviewbudget.CheckResolved` over the plan perch, so a plan hard cap below the plan budget it resolves is refused as `LoadConfig` refuses one today; loom's own load keeps that check for the discussion and code perches.
- Each moved key is declared in `KeyMoves` in `internal/configreg/keymove.go`. The reconcile moves a tuned value in a hub's base files to `tapestry.yaml`, and a profile block or `run_settings` naming the old module is translated when it is read, with nothing rewritten; the orch rewrites `profiles.yaml` to the new module afterwards.
- Every `module.key` reference in the recipe files to a moved key names the new module.

Rationale: the operator's split; `KeyMoves` is the repo's declared route for a key changing module, so no hub loses a tuned value and the run writes no migration of its own.

### D13 — The arbiter, tackler and driver-kind keys live in `shed.yaml` alone

Parent decision (orch, 2026-10-31): `arbiter`, `arbiter_timeout_min`, `arbiter_max_spawns`, `tackler`, `tackler_timeout_min`, `tackler_max_spawns` and `driver_kind` are removed from `loom.yaml` and `darn.yaml` and live in `shed.yaml` alone; a recipe that needs other values sets them in its profile block under the `shed` module.
The removed keys are retired, not moved, since moving them into `shed.yaml` would widen a loom-only value to every recipe.
They are declared in loom's and darn's `RetiredKeys` with a category other than `CategoryNone`, never `CategoryNone`, which `ValidateLocal` admits and every load drops silently (`internal/configengine/doc.go`), so every load still drops a hub base file's value until the reconcile prunes it.
They are pruned through that `RetiredKeys` reconcile prune alone, and never added to `retiredLoomKeys` or dropped by darn's `MigrateConfig`, so no `Migrate` strips one before the check below sees it.
Refusing rather than ignoring one of these keys keeps a run pinned to a driver in a profile, run settings or a local file from silently running another.
`configengine.Spec` gains a per-key way-forward text beside `RetiredKeys`, and loom's and darn's specs give each of these keys the text to set it under the `shed` module instead: in a local file, in the pair's local `shed.yaml`; in a profile block or `run_settings`, in the `shed` object.
Local-layer files are read and judged only by `internal/configengine`: `Layers` runs `ValidateLocal`, which refuses a retired key of a category other than `CategoryNone` naming the file and the key but no way forward (`internal/configengine/layers.go`), and that refusal appends the key's way-forward text.
So a pair seeded before landing whose local `loom.yaml` or `darn.yaml` holds one is refused when its config loads, naming the file, the key and the pair's local `shed.yaml`, with no module code reading the layer.
`PrepareRunSettings` keeps its own check on each module's raw object, after `moveRunSettings` and before `module.Migrate`, so the refusal names the key before any `Migrate` warning; it refuses with the same Spec text, so both refusals have one source.
`PrepareRunSettings` reaches every place a task's settings are checked: a board `run_settings` write (`internal/boardcli/runsettings.go`), every `profiles.yaml` write, and Seed-Child (`internal/battenshed/seamchild.go`).
The per-recipe override read in `internal/loomengine/config.go` that loads these keys from `loom.yaml` or `darn.yaml` goes, and every recipe, pawl included, reads them from `shed.yaml` and its profile block.
The paths that still write the keys go with them: the three loom-to-darn `arbiter*` entries in `KeyCopies` in `internal/configreg/keymove.go`, and loom's `migrateShedOverrides` migration in `internal/loomengine/config.go`.
`batten.yaml`'s overrides stay, outside this decision.

Rationale: settles the first of the entry's Open items as the parent decided it.
Bound: a hub base `loom.yaml` or `darn.yaml` value of one of these keys is dropped and pruned with no carry-over, since copying it into `shed.yaml` would widen it to every recipe; a run then takes `shed.yaml`'s value, and the orch moves each tuned value into the `shed` module of the profile blocks of the loom recipes before landing.

### D14 — `lym loom` and `lym shed` both stay

Parent decision (orch, 2026-10-31): `lym loom` keeps its run verbs for loom runs, and `lym shed` keeps its generic `run`, `step`, `status`, `pause` and `goto` for every recipe, batten and pawl included; neither is folded into the other.

Rationale: settles the last of the entry's Open items as the parent decided it.

### Added after Discussion (parent, 2026-10-31): The contract names a build segment, not a writer

Decision: The loom contract says: a record before, a build segment, and the landing rows after. The build segment is the rows a variant owns that turn the record into the change: Plan-Write, its perch and Webster in tapestry, Darn-Write in darn. The contract and the code do not call this slot 'the writer', since Discussion-Write, Plan-Write, Describe and the fixers are writers too.

Rationale: Operator, 2026-10-31: 'writer' names many agents; tapestry's plan is part of making the change.

### Added after Discussion (parent, 2026-10-31): A beam seed names no recipe this lym knows

Decision: A seed whose recipe is beam is refused by lym loom and lym shed alike as naming no embedded recipe, and the refusal names the re-seed way forward. This replaces the part of D2 and D11 (and D11's acceptance criterion) that had lym loom refuse it as a non-loom recipe naming the lym shed verbs.

Rationale: D11 also has lym shed refuse beam, so pointing a beam seed at the lym shed verbs would send it to another refusal (PATTERN-refusal-way-forward); a re-seed is the only way forward that works.

### Added after Discussion (parent, 2026-10-31): A record not committed names its way forward by the current row

Decision: This amends D4's Not-committed bullet and its acceptance clause. When the decision record is written but not committed, the refusal names the way forward by the run's current row: at Record-Write it names lym loom resume; at any later row it names lym loom goto --to Record-Write, then lym loom resume, then re-running the verb.

Rationale: A darn run in flight that halted at Darn-Write on its old discussion gate also holds a written, uncommitted record, and a bare lym loom resume would repeat forever on that run (PATTERN-refusal-way-forward).

## Constraints

- PATTERN-shed-recipe-registry: `RecordWrite` is one more `Constructor` in the one map in `internal/shedrecipe`; the contract's engine sets sit beside it.
- PATTERN-sole-parsers: `contract` and `verify_source` are decoded only by the recipe parser in `internal/shedbuild`, and the record and support log only by `internal/discussionparser`.
- PATTERN-refusal-way-forward: D1's contract refusal, D2's non-loom refusal, D3's verb refusals, D4's missing-record halt, D6's stop halt and D13's retired-key refusal each name the way forward, and a test reaches each.
- PATTERN-gate-slot and the Gate Self-Check Parity Invariant: D5's gate and `lym loom validate-discussion` call one function.
- PATTERN-config-layers: the split adds a module through the config registry and template, and moves keys only through `KeyMoves` and `hubreconcile.Ensure`.
- PATTERN-sparse-hub-base: a moved key lands in `tapestry.yaml` only where the hub had tuned it.
- PATTERN-stencil-ownership: the one Describe stencil and the darn writer stencil are read at call time; the writer stencil points at the discussion spec and restates none of it.
- PATTERN-tolerant-records: the seed keeps accepting unknown keys once the alias goes.
- PATTERN-shed-stop-chain changes with D13, still admitting `batten.yaml`'s arbiter and tackler overrides.
- Recipe row names are durable on-disk identities: a removed row is mapped onto its successor in the recipe's retired-row data.
- The shared seed still names Preflight as the first row until `seed-starts-at-start` lands, so both loom recipes keep Preflight as their entry row.

## Auto-mode assumptions

- D4: the row name `Record-Write`, its place after Preflight, and the support log it writes; the board body in a fenced block with the parser's fence rule made CommonMark's; in-flight runs handled by parked row, with Darn-Write refusing a missing record before its session.
- D5: the history check on `tapestry` records too.
- D6: the stop marker as the writer's halt.
- D8: `verify_source` as a recipe-file key.
- D10: the `batcher.yaml` load in Preflight rather than the Webster row, and a run parked on Tapestry-Preflight mapped to Preflight.
- D12: which shared keys stay in `loom.yaml`, and the plan perch's budget check in the `tapestry` load.
- D13: retiring rather than moving the removed keys, refusing them in profiles and run settings with a check of their own and in a pair's local files through `configengine`'s local-layer check, both with one Spec way-forward text, and leaving `batten.yaml` alone.

## Open risks

- The note `finalize-fix-in-place` removes Finalize's `on_reroute`; D9 changes `tapestry`'s edges after Parent-Align, the reroute's target, so the later task must read the new order.
- D13 refuses a profile block or task `run_settings` that still names `arbiter*`, `tackler*` or `driver_kind` under `loom` or `darn`; the orch must move each under `shed` when it rewrites `profiles.yaml`, or such runs refuse; a pair seeded before landing whose local `loom.yaml` or `darn.yaml` holds one refuses at load until the key moves to its local `shed.yaml`; a hub base `loom.yaml` or `darn.yaml` that tuned one is dropped and pruned with no carry-over, so the orch moves each such value into a loom recipe's profile block under `shed` before landing, or the hub's runs silently take `shed.yaml`'s value.
- Runs in flight when this lands may need the orch's `goto`: a darn run with no record parked on Drift-Baseline or Drift-Await, whose Drift-Baseline, resumed without the goto, points its agent at the missing record with no route to the board entry, so it halts Stuck with drift's ways forward, which loop, or records a scope from no subject, which stands, or halted at Darn-Write on the missing record (D4), each moved with `lym loom goto --to Record-Write` then `lym loom resume`, and a `tapestry` run inside the Code-Review perch (D9).

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-3-review.md; fixer report: _lym/reviews/discussion/round-3-fixer-report.md.
  - d13-retired-key-refusal-mechanism (class design, severity MEDIUM, open in rounds 1, 2, 3)
  - d4-decision-add-darn-refusals-way-forward (class consistency, severity MEDIUM, open in rounds 2, 3)
  - d4-differs-from-committed-restore-loses-answer (class design, severity MEDIUM, open in rounds 3)
  - d4-inflight-darn-halt-unreachable (class design, severity BLOCKING, open in rounds 1, 2, 3)
  - d5-darn-guard-damaged-record-no-recovery (class design, severity MEDIUM, open in rounds 3)
<!-- /lym:carry-over Discussion-Review -->

<!-- lym:carry-over Plan-Review -->
- Plan-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/plan/round-3-review.md; fixer report: _lym/reviews/plan/round-3-fixer-report.md.
  - card6-missed-moved-key-readers (class design, severity BLOCKING, open in rounds 1, 2, 3)
  - tapestry-run-wide-key-routing (class design, severity MEDIUM, open in rounds 2, 3)
<!-- /lym:carry-over Plan-Review -->

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - implementer-spec-stale-loom-implementer-sentence (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1: loading `tapestry` and darn passes the contract check; a recipe declaring `contract: loom` without a record row, without a build segment, with the build segment before the record row, without Finalize ending its `on_done` path, or without `verify_source` is refused, naming the clause and the way forward.
- D2: a `lym loom` verb on a run whose seed names a non-loom recipe other than pawl refuses, naming the `lym shed` verbs; a pawl run is served as today.
- D3: `validate-plan`, `insert-card`, `remove-card` and `restore-renumber` on a darn run refuse, saying they do not apply; `lym loom peers` reports a darn peer's row against the darn recipe, from its seed or its entry's `recipe` field, and an off-machine pawl peer's row, from a project entry with an empty `recipe` field, against the pawl recipe; `decision add --amends-plan` on a darn run refuses; `decision add --pattern-*` on a darn run appends the bullet, and the darn writer stencil applies `## PATTERN changes` bullets.
- D4: a darn run writes and commits a record with the board body verbatim in a fenced block under `## Goal` and the other sections empty, and a support log, before Drift-Baseline; a resume with an existing pair leaves both byte for byte; a resume after a crash that left the record but no support log, or an uncommitted pair, writes the missing support log and commits, leaving the record byte for byte; `decision add` appends to that record; a resume after a crash that left a torn temporary file removes it, writes the file whole and commits no temporary file; a body holding a `## Decisions` and a `## Open risks` heading, a body with no backticks and one holding inline code only, each also holding those two headings, one holding its own fenced code block with a `## Decisions` line inside, and one with an unclosed fence each yield a record that validates and whose `decision add` append lands under the record's own `## Decisions`; darn's Drift rows read the record as their subject; a darn run with no record halts at Darn-Write before spawning its session, naming `goto --to Record-Write` then `resume`, and that route reaches Darn-Write both on a task branch with no commit past the branch point and on one with commits whose scope record holds `awaited_at`; a darn run whose working record was removed while the `_lym` HEAD holds it halts there naming its restore from HEAD and `resume`; `decision add` on a darn run with no record names `lym loom resume` when the current row is Preflight or Record-Write, and `goto --to Record-Write` then `resume` when it is Drift-Baseline, Drift-Await or Darn-Write; `decision add` refuses a darn record not committed at HEAD naming `lym loom resume`, one differing from its committed copy by an appended entry or by an appended `## PATTERN changes` bullet whose commit failed naming `lym fabric commit` and re-running the refused call alone, one differing otherwise naming its restore, and one whose committed copy lacks a historical entry naming restoring and committing the entry, and none of its darn refusals names the Darn-Write session.
- D5: the one validator accepts the D4 record, and refuses a D4 record whose body holds `## Open risks` and whose own `## Open risks` heading is removed; a Discussion-written record citing a missing path or with a malformed relevant list is refused; a Discussion-written record holding a top-level fence whose body has a ```` ```js ```` line followed by a `## Decisions` line before the fence's closing line, and one holding a fence indented three spaces around a `## Decisions` line, each validate with that line read as fenced, and its `decision add` append lands under the record's own `## Decisions`; a record that drops an `Added after Discussion` entry is refused on both recipes; a darn record or support log that fails Darn-Write's guard after its session halts the run, Stuck, with no re-prompt, naming the findings and restoring the file's latest accepted version, `lym fabric commit` and `lym loom resume`, and a resume after that restore passes the guard; the gate and `validate-discussion` report the same findings.
- D6: the darn writer stencil sends an outdated-entry answer to the parent for `lym loom decision add` and never edits the record, and its no-parent stop report names `lym loom decision add` then `lym loom resume`, never a board note; the stop marker halts the run with the description as its reason, after a Done handoff and after a gate-failed Stuck alike, on a first spawn and on a respawn whose earlier spawn committed work alike; the halt names `lym loom decision add` then `lym loom resume`, and an answer recorded that way reaches the resumed writer and the Code-Review perch as an `Added after Discussion` entry; a no-parent run whose board holds an answering `Decision for <slug>:` note but whose record holds no `Added after Discussion` entry for it halts again on resume.
- D7: darn's Code-Review rows name the decision record as the answer key, and a darn run writes no `board-entry.md`; a pawl run still writes it.
- D8: one Describe table and stencil serve both recipes; the wiring holds no `tapestry` or darn recipe-name branch; the verify source follows `verify_source`.
- D9: `tapestry`'s routing runs Parent-Align, Drift-Verify, then the Code-Review perch, then Describe, and Finalize's reroute reaches Drift-Verify again.
- D10: no Tapestry-Preflight row exists; a broken `batcher.yaml` fails Preflight on a `tapestry` run and not on a darn run; a run parked on Tapestry-Preflight resumes at Preflight and fails there on a broken `batcher.yaml`; a run whose current row is Preflight and whose status, edited outside the seed, resume and goto routes, holds history past Preflight, a set `start_sha` or `pause_requested` proceeds into the row after Preflight with no seed-check refusal.
- D11: `lym shed` refuses a seed recording `beam` as an unknown recipe, and `lym loom` refuses it as a non-loom recipe.
- D12: the `plan_*` and `implementer` keys load from `tapestry.yaml`; a hub `loom.yaml` that tuned `plan_max_bounces` carries the value to `tapestry.yaml` after reconcile; a profile naming `loom.plan_max_bounces` still applies; `tapestry.plan_max_bounces: null` takes `loom.review_max_bounces`; a `tapestry.plan_hard_max_bounces` below the plan budget it inherits from `loom.review_max_bounces` is refused.
- D13: `loom.yaml` and `darn.yaml` hold no `arbiter*`, `tackler*` or `driver_kind` key; a run reads those from `shed.yaml` and its profile block; a profile or `run_settings` naming `loom.driver_kind` is refused at its write and at Seed-Child, naming the `shed` module; a seeded pair whose local `loom.yaml` holds `driver_kind` is refused at load by `configengine`'s local-layer check, naming the file, the key and the local `shed.yaml`, and the run-settings refusal carries the same Spec way-forward text; `KeyCopies` holds no `arbiter*` entry and loom has no `migrateShedOverrides`.
- D14: `lym shed` keeps its five verbs for every recipe and `lym loom` keeps its verbs.

## PATTERN changes

- `PATTERN-loom-contract` — adds: Adding a loom recipe: it declares `contract: loom`, checked by Shed's recipe loader; `lym loom` verbs serve loom runs only, bar pawl until it joins; a verb for some loom recipes refuses on others by the recipe's rows, never its name.
- `PATTERN-shed-stop-chain` — changes: A shed run's stop: rule table, tackler, arbiter, then parent, for every recipe; a stop the driver's arbiter seam marks as judged first, by row and reason, skips the tackler; arbiter keys in shed.yaml, batten's overridable in batten.yaml, a loom recipe's own via its profile block.

## Notes for the plan writer

- D12's per-worktree module needs its own template file, registry entry and config CLI, like `darn`'s in `internal/darnengine`.
