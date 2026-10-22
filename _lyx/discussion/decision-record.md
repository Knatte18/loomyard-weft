# Decision record — plan-card-rules

## Goal

Four changes to the plan and its cards, so that faults a reviewer or the operator meets today are caught or owned inside the run:

1. The plan gate refuses a plan whose `PATTERN.md` growth crosses the headroom line, before any review round.
2. A card can declare that it only adds to a shared ref (`Extends`), so additive cards on the same ref stop being dependencies.
3. A parent or arbiter decision that amends the approved plan has an owner inside the run: the Plan-Review fix loop.
4. A verify failure at Merge-In or Publish is traced through quarry's Enclose to the plan cards whose targets hold the failing members, and the fixer's brief names those cards.

## Scope

In:

- Plan gate: a projection of the overview from the cards' verbatim entry lines, checked against the headroom line; a check that every declared `adds` or `changes` entry has a verbatim line.
- The `**Extends:**` type label in `internal/planparser`, one pair rule for dependency edges used by `internal/websterengine`'s batch-order check and by `tools/plandag`, its plan-review rubric check and its plan-writer stencil guidance.
- `tools/plandag`: the `Extends` mark and a flag that guesses additive refs.
- `lyx loom decision add --amends-plan`, the amendment record it writes, the settled-segment guard's admission for it, the Plan-Burler-Review answer-key instruction, the arbiter's route through its stop envelope, the orch stencil route, and Webster's plan-conflict recurrence way forward.
- The failure trace: locations from the failed verify's log, Enclose at the verified head, matching to plan cards, new fields on the Publish failure record and their rendering in the Publish failure note.
- Docs in the same commits: `loom-plan-spec.md`, the package `doc.go` files the changes touch, and the glyph-conversion background file.

Out:

- Parallel batches, worktrees per batch and merge-back (`parallel-batch-parts`, `weaver`): `Extends` only changes which pairs are dependencies, and the pair rule is placed so their batchifier can call it.
- The `fabriccli#addWeftVerbs` cleanup the board entry's side finding names.
- Every later Enclose use the board entry lists as its own decision: re-running one batch, mapping review findings to glyphs, scope-gap reports, grouping failures per member.
- Measuring whether fixers use the named card; that is a later audit.
- A Discussion-stage headroom check.
- Darn runs: they have no plan, so neither the plan checks nor the failure trace apply, and `--amends-plan` is refused on one.
- Non-Go verify output: location extraction reads Go test and build output only, so a C# hub's failure records an empty trace.
- `lyx webster rebaseline`'s operator wording: the verb stays the operator's for runs outside loom's auto-rebaseline.
- The hub orch rule that lets the orch edit a card by hand: it lives in `_board/_lyx/orch/rules.md`, outside this repo.

## Decisions

### D1 — The headroom check projects the overview from the cards' verbatim lines

**Decision:** The plan gate builds the projected overview from `PATTERN.md` in the worktree.
Every fenced verbatim entry line a card prescribes (the line shape `pattern-entry-line-cap` already finds) replaces the whole bullet of the same entry ID, continuation lines included, or is appended when the ID is absent; every `removes` entry of the plan's `## PATTERN changes` deletes its ID's bullet.
Every fenced entry line counts as a prescription; within one card the last fenced line of an ID wins, and across cards the later card in plan order wins, since the plan runs in card order.
It then runs `pattern.Headroom` with the overview at the task branch's merge base with its parent as the base and the projection as the head, the same base the verify gate's headroom lint uses.
The Plan-Write, Plan-Burler-Fix and PR-Rework gates and the `lyx loom validate-plan` self-check all get the same merge-base reader; with no parent recorded the base is the overview at `HEAD`.
A finding is a blocking `pattern-headroom` finding naming the projected size, the base size, the line and the bytes to cut.
Its way forward: shorten the prescribed lines or move detail into background files; when that is not enough, ask the parent to record a `changes` decision that shortens other entries with `lyx loom decision add --pattern-entry … --pattern-action changes --pattern-text …`, then prescribe those lines on a card.

**Rationale:** The declared bullet text is design prose, never the overview line, so only a verbatim line can be measured.
Replacing by ID makes the projection idempotent over cards already built, so the same check holds across generations.
One base and `pattern.Headroom` for every gate and the self-check mean the plan gate and the run-time lint cannot disagree on a plan whose lines are built as written.

### D2 — Declared `adds` and `changes` entries must have a verbatim line

**Decision:** A new blocking plan-gate check, `pattern-line-unprescribed`: one finding per `adds` or `changes` entry of the plan's `## PATTERN changes` that no live card prescribes as a fenced verbatim line and whose bullet in the worktree `PATTERN.md` is unchanged from the merge base.
An entry an earlier generation already built is therefore satisfied without a new card.
Its way forward is to prescribe the entry's whole overview line in a fenced block on the card that edits `PATTERN.md`.
The plan-writer stencil's rule that a prose instruction to append to an entry passes is replaced: a declared entry is prescribed verbatim, and a card fences only the new line, never the old one or an example.
It runs in every mode `pattern-changes-mismatch` runs in, and only on a loom run.

**Rationale:** Without it a plan can escape D1 by describing a change in prose.
Its bound: an undeclared prose edit of `PATTERN.md` still escapes the plan gate, and the per-card and verify-gate headroom lints stay the guard for it.

### D3 — `Extends` is a type label: the card only adds to the ref

**Decision:** `**Extends:**` joins the type labels, with `CardTypeExtends`.
Its targets are refs that already exist (a file, a self glyph, a member glyph, or a `plan:` handle an earlier card creates).
It means the card adds to the ref (a flag, a list line, a doc paragraph, a spec entry, a verb in a wire function) and changes nothing already there.
Every reader of an `Edit` group's targets, in `internal/planparser`, `internal/planglyph` and `internal/batcher`, reads an `Extends` group alike: the plan-gate checks (`edit-test-file` included), the delete-order check, the batcher's boundary and estimate, the scope guard and the changes-files union.
The dependency rule (D4) is the one reader that tells them apart.
A ref under `Extends` and under another type label or `Uses:` of the same card is a blocking finding, as `card-field-overlap` is for targets against `Uses:`.

**Rationale:** The card does change the ref, so every guard over targets must keep seeing it.

### D4 — Two `Extends` on one ref are no dependency

**Decision:** One function in `internal/planparser` decides whether a pair of overlapping refs of two cards makes a dependency edge: a pair where both sides sit under `Extends` makes none; `Extends` against any other type label makes one, and so does a `Uses:` entry against an `Extends` target.
`internal/websterengine`'s batch-order check and `tools/plandag` call it; `tools/plandag` draws such cells as `E`.

**Rationale:** Additions in different hunks merge cleanly and additions on the same lines are a simple conflict for a merge-back resolver; a real change and a read of added content stay ordered.
One function keeps the rule from drifting between Webster, the tool and the batchifier `parallel-batch-parts` plans.
The bound: a wrongly labelled `Extends` drops a real dependency, which today, with batches run in card order, changes nothing, and once parallel batches exist surfaces as a merge conflict or a verify failure on the merged tree, never silently.

### D5 — The measurement comes first as a `tools/plandag` flag

**Decision:** `tools/plandag` gains `-additive-guess`: on refs no card labels `Extends`, it treats change against change as no dependency where the ref is guessed additive by path and name (a `doc.go`, anything under `docs/`, `PATTERN.md`, `contracts/specs/`, and a member glyph whose name starts with `wire` or is `addWeftVerbs`); labels always win.
The report shows the critical path and best speedup with and without the guess.
This card comes first in the plan, and the build of D3 and D4 does not wait on a rerun: the orch's 2026-10-18 estimate in the board entry applied this guess with plandag's own overlap rule.

**Rationale:** The operator's scope line puts the label, its card-DAG rule and its plan-review check in this task, so the measurement orders the work and keeps the estimate repeatable on any recorded plan rather than gating the build.

### D6 — Plan review checks the label, and docs ride with code

**Decision:** The plan-review rubric gains a check: an `Extends` on a ref the card rewrites, reorders or removes content of is a finding, as a missing `Uses:` is today.
The plan-writer stencil says when to use `Extends`, and that the card writing the code also writes its docs under `Extends`, so no separate closing docs card is planned.

**Rationale:** The label's only risk is a wrong label (D4), and the plan reviewer already judges what each card does to its refs.

### D7 — A plan amendment is owned by the Plan-Review fix loop

**Decision:** `lyx loom decision add` gains `--amends-plan`, a boolean flag.
With it the verb writes the never-tracked amendment record under the run's task worktree `.lyx/loom/`, located as the verb locates the decision record, before the record's commit, and removes it again when the append or the commit fails, so a decision and its amendment record land together or not at all.
The amendment record lists the titles of every pending amending entry, a later call appending to it, and the latest write time in whole-second UTC.
The settled-segment goto guard admits a goto onto `Plan-Burler-Review` over a live Plan-Review settle record when the amendment record exists and was written after that settle; that is its only new admission.
The Plan-Burler-Review answer-key instruction gains: when the amendment record exists, each decision-record entry it names is a finding for every card that does not yet carry it, fixed by editing only cards Webster has not recorded done, or by inserting a card with `lyx loom insert-card` after the last begun batch.
The amendment record is removed by Plan-Bouncer's approved settle only when that settle judged a round run after the record's write time, and by PR-Rework's archive of a generation.
The route is then the existing one: the review round, the fix, the plan gate, Plan-Bouncer, its commit seam, and the Webster row's auto-rebaseline on entry.
Because the review round lands after the settled round, Plan-Bouncer re-enters on the round-by-round path, not as an amendment generation: it judges the fixed plan on the Plan-Review segment's already-spent bounce budget, and a judge that does not converge halts on that budget for the parent's `lyx loom resume`, as the existing Publish-failure goto onto Webster-Burler-Review does.
Webster's plan-conflict recurrence halt names this route as its way forward instead of a hand edit.
`--amends-plan` is refused on a darn run and on a run with no approved plan, naming the reason.

**Rationale:** The plan fixer is the one role that already edits an approved plan under the plan gate and the commit seam, and the Webster row already accepts what it changes; the only missing piece is a way in over a settled verdict.
A goto onto Plan-Bouncer cannot carry it, since an unchanged plan re-settles with no spawn.
The bound: the admission re-opens only Plan-Review, only for an amendment written after its last settle, and the fix still passes the plan gate (`done-card-edited` included) and a Plan-Bouncer judgment before Webster sees it.

### D8 — The arbiter and the orch take that route

**Decision:** The loom arbiter wiring puts the amendment target row, `Plan-Burler-Review`, into the stop envelope; a recipe that sets none offers no such route.
The arbiter stencil stays recipe-blind: a stop whose cause is a plan defect that the board entry and the decision record already settle is the arbiter's call when the envelope names an amendment target; it records the amendment with `lyx loom decision add --by arbiter … --amends-plan` and writes a `resume` verdict whose `goto` is that target.
Its hand-on rule is unchanged: an amendment that needs an operator decision or widens scope is handed on.
The orch stencil: a design call that amends the plan is recorded with `--amends-plan`, then `lyx loom goto --to Plan-Burler-Review` and `lyx loom resume`, never by editing a card.
Both word the decision as a concrete change to named cards, so the fixer can apply it as a finding.

**Rationale:** The arbiter already records decisions and names goto targets at or before the current row, and Plan-Burler-Review is before every row that can stop on a plan defect; the envelope keeps the stencil fit for `shed-arbiter-all`'s move of the arbiter to every recipe.

### D9 — A Merge-In or Publish verify failure is traced to plan cards

**Decision:** When Merge-In or Publish writes the Publish failure record on a loom run, Go also traces the failure:

1. It extracts `path:line` locations from the tails of the failed log's failure identities (`ParseVerifyFailures`) and places each relative to the repo root: a path that already exists at the record's `Head` as written; an absolute path under the worktree by stripping the worktree root; a bare file name or a `./`-relative one through its identity's package directory, the import path less the module path from the worktree's `go.mod`.
   Any other location is dropped.
2. It calls quarry's `EncloseAt` at the record's `Head` with the locations, capped at 20, through `internal/planglyph`, which owns the quarry repo handle.
3. It matches each found member glyph to every live plan card whose flat targets hold it under the overlap rule the scope guard and the batch-order check use (the glyph itself, a containing type, its unit's self glyph, or its file), with an external test unit mapped through `internal/planparser`'s existing adjustment, and reads each matched card's batch and commit from Webster's run state by the live card id.

The record gains `cards`: per matched card its number, the glyphs and locations matched, and the batch and commit (empty for a card Webster holds no record of); and `untraced`: the locations quarry rejected or no live card's targets hold, as `path:line`.
`checkedPublishFailure` shape-checks the new fields as it does the others, dropping a field that fails.
`PublishFailureNote` renders them after the failing tests: each card with its number, and its slug and first `Intent` line read from the live plan at render time, never stored in the record, and its commit, as the starting point for the fix; then the untraced locations.
A trace failure (no plan, no Webster state, a quarry error) is logged at Warn and leaves the record without the fields; it never changes the Stuck verdict or the record's other fields.

**Rationale:** The note is what the Webster-Review reviewer and fixer read after such a failure, so naming the card there needs no new channel.
Quarry v0.4.0, which `go.mod` already pins, ships `Enclose` and `EncloseAt` (`quarry/enclose.go`), so no module bump is needed.
Reading slug and intent from the plan keeps the note's rule that no free text reaches the prompt from the record.

## Constraints

- `PATTERN-glyph-conversion-chokepoint`: Enclose is a location-to-glyph conversion done by quarry; it becomes the one sanctioned such call, in `internal/planglyph`, added to the entry's background file; the overview line is unchanged.
- `PATTERN-tolerant-records`: the Publish failure record's new fields are optional, and an older record without them reads as before.
- `PATTERN-durable-vs-ephemeral-state`: the amendment record is never-tracked, under `.lyx/loom/`, declared through `internal/lyxdirs`.
- `PATTERN-leaf-packages`: the overview projection in `internal/pattern` keeps that package's import set.
- `PATTERN-told-geometry`: `landingshed` gets the trace as a told seam, wired in `internal/loomcli`; it derives no path.
- `PATTERN-refusal-way-forward`: `pattern-headroom`, `pattern-line-unprescribed`, the `--amends-plan` refusals and the guard's refusal name their way forward, and a test reaches each.
- `PATTERN-sole-parsers` and `PATTERN-ref-shape-registry`: the plan's labels, refs and fenced lines are read only through `internal/planparser`, the record's bullets only through `internal/discussionparser`.
- `PATTERN-review-round`: the amendment reaches the plan only through a review round's fixer.
- `PATTERN.md` is already past its headroom line, so this task adds no entry and no card grows the overview.

## Auto-mode assumptions

- The failure trace keeps a fixed cap of 20 locations per record.
- An `Extends` on a `_test.go` file is refused as an `Edit` on one is, since adding a test is a `Create` of its member.

## Open risks

- The trace often names no card: Go test failure lines name only the file's base name, a failure in production code that only a test's assertion reports traces to the test, and members a retired PR-Rework generation built trace to no live card; the untraced list and the failing tests stay in the note for those cases.
- An amendment late in a run judges on the Plan-Review segment's spent budget (D7), so a non-converging judgment needs the parent's `lyx loom resume`, never the operator.
- The amendment record is never-tracked, so a run adopted on another machine before the goto loses it; the goto is then refused, which stalls the route until the decision is recorded again with `--amends-plan`.
- The hub orch rule allowing a hand edit of a card now conflicts with the orch stencil (D8); the orch retires it with `lyx orch rules` after landing.
- A loom profile without the Plan-Review perch (`speed-loom-profile`) would leave the amendment with no fix loop.

## Acceptance criteria

- A plan whose cards' verbatim lines grow the projected overview past the headroom line from the merge base fails the plan gate with `pattern-headroom`, naming the bytes to cut; a plan whose lines replace existing bullets at equal or smaller size passes.
- `lyx loom validate-plan` and the row gate give the same headroom verdict on a branch that already grew `PATTERN.md`.
- A plan that declares `removes` for an entry passes with the projection smaller by that bullet.
- Two cards prescribing a line for one ID project the later card's line; two fenced lines for one ID in one card project the last.
- A rework plan whose earlier generation already wrote its lines to `PATTERN.md` projects the same size as before they were built, and its declared entries raise no `pattern-line-unprescribed`.
- A declared `adds` or `changes` entry with no verbatim line and an unchanged bullet fails with `pattern-line-unprescribed`; the same plan with the line fenced on a card passes.
- A card with an `**Extends:**` group parses with `CardTypeExtends`; an `Extends` ref that does not exist, or that also sits under another label or `Uses:` of the card, is a blocking finding; an `Extends` ref to a symbol an earlier card deletes is flagged as an `Edit` ref is.
- Two cards in reverse batch order whose only shared ref sits under `Extends` on both pass the batch-order check; an `Extends` against an `Edit` on the same ref, or against a `Uses:`, is still refused.
- `tools/plandag` draws `E` cells and drops `Extends`-against-`Extends` edges; with `-additive-guess` it reports both critical paths, and a labelled ref keeps its label.
- `lyx loom decision add --amends-plan` lands the decision and the amendment record together, a second call appends its title, and a failed commit leaves neither; on a darn run, or with no approved plan, it is refused with its way forward.
- With an amendment record written after the Plan-Review settle, `lyx loom goto --to Plan-Burler-Review` is admitted; with none, or one older than the settle, it is refused as today.
- An approved Plan-Bouncer settle that judged no round after the amendment record's write keeps the record; one that did removes it; PR-Rework's archive removes it.
- An arbiter stop envelope from a loom run names the amendment target; one from a recipe that sets none names none.
- Webster's plan-conflict recurrence halt names the amendment route.
- A loom Merge-In verify failure whose log places a build error or a test failure in a member a card targets writes a record whose `cards` names that card, its batch and its commit, and the Publish failure note shows the card's number, slug, intent and commit.
- A location quarry rejects, such as a `.md` file, or one no card's targets hold, appears in `untraced` as `path:line`.
- A quarry error during the trace leaves a record with the other fields and the same Stuck reason.
- An older Publish failure record without the new fields still reads and renders.

## Notes for the plan writer

- `PlanContext` in `internal/loomshed/plancontext.go` needs the merge-base reader, wired where the verify gate's `VerifyMergeBase` is, and in the `validate-plan` CLI.
- The projection is pure byte work and fits `internal/pattern` beside `Headroom`; the plan gate reads the files.
- Webster's `deriveEdges` compares flat `Targets`, so the pair rule needs each ref's group type.
