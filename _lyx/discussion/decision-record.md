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
- `lyx loom decision add --amends-plan`, the amendment record it writes, the Plan-Review settle record's round start and skip seam, the settled-segment guard's admission for it, the Plan-Burler-Review answer-key instruction, the arbiter's route through its `amendment_target` prompt input and recipe key, the orch stencil route, and Webster's plan-conflict recurrence way forward.
- The failure trace: locations from the failed verify's log, Enclose at the verified head, matching to plan cards, new fields on the Publish failure record and their rendering in the Publish failure note.
- Docs in the same commits: `loom-plan-spec.md`, `shed-recipe-spec.md` for the `amendment_target` recipe key, the package `doc.go` files the changes touch, and the glyph-conversion background file.

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
A `removes` entry whose ID a card also fences a line for is a blocking plan-gate finding naming the bullet and the card, since the plan both drops and rewrites the entry; its way forward is to drop the fenced line or the `removes` entry.
Every fenced entry line counts as a prescription; within one card the last fenced line of an ID wins, and across cards the later card in plan order wins, since the plan runs in card order.
Two cards fencing a line for one entry ID therefore make a dependency edge whatever their labels, decided by D4's pair rule, so the later card still runs later once batches run in parallel.
It then runs `pattern.Headroom` with the overview at the task branch's merge base with its parent as the base and the projection as the head, the same base the verify gate's headroom lint uses.
The Plan-Write, Plan-Burler-Fix and PR-Rework gates and the `lyx loom validate-plan` self-check all get the same merge-base reader; with no parent recorded the base is the overview at `HEAD`.
A finding is a blocking `pattern-headroom` finding naming the projected size, the base size, the line and the bytes to cut.
Its way forward: shorten the prescribed lines or move detail into background files; when that is not enough, ask the parent to record a `changes` decision that shortens other entries with `lyx loom decision add --pattern-entry … --pattern-action changes --pattern-text …`, then prescribe those lines on a card.
Once the plan is approved, the parent also passes `--amends-plan` with the pattern flags, and the change routes as D7 states.

**Rationale:** The declared bullet text is design prose, never the overview line, so only a verbatim line can be measured.
Replacing by ID makes the projection idempotent over cards already built, so the same check holds across generations.
On a run with a recorded parent, one base and `pattern.Headroom` for every gate and the self-check mean the plan gate and the run-time lint cannot disagree on a plan whose lines are built as written.
With no parent recorded, the verify gate skips its headroom lint, so the plan gate's comparison against `HEAD` is the only plan-level headroom check.

### D2 — Declared `adds` and `changes` entries must have a verbatim line

**Decision:** A new blocking plan-gate check, `pattern-line-unprescribed`: one finding per `adds` or `changes` entry of the plan's `## PATTERN changes` that no live card prescribes as a fenced verbatim line and whose bullet in the worktree `PATTERN.md` is unchanged from the merge base.
An entry an earlier generation already built is therefore satisfied without a new card.
Its way forward is to prescribe the entry's whole overview line in a fenced block on the card that edits `PATTERN.md`.
The plan-writer stencil's rule that a prose instruction to append to an entry passes is replaced: a declared entry is prescribed verbatim, and a card fences only the new line, never the old one or an example.
It runs in every mode `pattern-changes-mismatch` runs in, and only on a loom run.

**Rationale:** Without it a plan can escape D1 by describing a change in prose.
Its bound: two edits escape the plan gate.
An undeclared prose edit of `PATTERN.md` is one.
The other is a further prose change to a declared entry whose bullet already differs from the merge base and that no live card fences, since that entry is projected at its current bullet.
The card gate's `lyx loom lint-pattern` and the verify gate's headroom lint stay the guard for both.

### D3 — `Extends` is a type label: the card only adds to the ref

**Decision:** `**Extends:**` joins the type labels, with `CardTypeExtends`.
Its targets are refs that already exist (a file, a self glyph, a member glyph, or a `plan:` handle an earlier card creates).
It means the card adds to the ref (a flag, a list line, a doc paragraph, a spec entry, a verb in a wire function) and changes nothing already there.
Every reader of an `Edit` group's targets, in `internal/planparser`, `internal/planglyph` and `internal/batcher`, reads an `Extends` group alike: the plan-gate checks (`edit-test-file` included), the delete-order check, the batcher's boundary and estimate, the scope guard and the changes-files union.
The dependency rule (D4) is the one reader that tells them apart.
A ref under `Extends` and under another type label or `Uses:` of the same card is a blocking finding, as `card-field-overlap` is for targets against `Uses:`.
A re-sign arrow sub-bullet under `Extends` is a blocking plan-gate finding, since a re-sign changes what is already there; its way forward is to move that ref under `Edit`.

**Rationale:** The card does change the ref, so every guard over targets must keep seeing it.

### D4 — Two `Extends` on one ref are no dependency

**Decision:** One function in `internal/planparser` decides whether a pair of overlapping refs of two cards makes a dependency edge: a pair where both sides sit under `Extends` makes none, unless both cards fence a verbatim `PATTERN.md` line for one entry ID (D1), which makes one; `Extends` against any other type label makes one, and so does a `Uses:` entry against an `Extends` target.
`internal/websterengine`'s batch-order check and `tools/plandag` call it; `tools/plandag` draws such cells as `E`.

**Rationale:** Additions in different hunks merge cleanly and additions on the same lines are a simple conflict for a merge-back resolver; a real change and a read of added content stay ordered.
The function decides the edge for a pair its caller already found overlapping, so the pair verdict cannot drift between Webster, the tool and the batchifier `parallel-batch-parts` plans.
Overlap detection stays each caller's: Webster's `refsIntersect` matches exact strings, and plandag's `columnsOverlap` also matches a member against its type.
The bound: a wrongly labelled `Extends` drops a real dependency, which today, with batches run in card order, changes nothing, and once parallel batches exist surfaces as a merge conflict or a verify failure on the merged tree, never silently.

### D5 — The measurement comes first as a `tools/plandag` flag

**Decision:** `tools/plandag` gains `-additive-guess`: on refs no card labels `Extends`, it treats change against change as no dependency where the ref is guessed additive by path and name (a `doc.go`, anything under `docs/`, `PATTERN.md`, `contracts/specs/`, and a member glyph whose last dotted segment starts with `wire` or is `addWeftVerbs`, so `internal/loomcli#loomCLI.wire` is guessed additive as the orch's estimate treated it); labels always win.
The report shows the critical path and best speedup with and without the guess.
This card comes first in the plan, and the build of D3 and D4 does not wait on a rerun.

**Rationale:** The orch's 2026-10-18 estimate in the board entry already applied this guess with plandag's own overlap rule.
The operator's scope line puts the label, its card-DAG rule and its plan-review check in this task, so the measurement orders the work and keeps the estimate repeatable on any recorded plan rather than gating the build.

### D6 — Plan review checks the label, and docs ride with code

**Decision:** The plan-review rubric gains a check: an `Extends` on a ref the card rewrites, reorders or removes content of is a finding, as a missing `Uses:` is today.
The plan-writer stencil says when to use `Extends`, and that the card writing the code also writes its docs under `Extends`, so no separate closing docs card is planned.

**Rationale:** The label's only risk is a wrong label (D4), and the plan reviewer already judges what each card does to its refs.

### D7 — A plan amendment is owned by the Plan-Review fix loop

**Decision:** `lyx loom decision add` gains `--amends-plan`, a boolean flag.
With it the verb writes the amendment record, `_lyx/loom/amendment.yaml` in the run's task worktree, located as the verb locates the decision record.
It is tracked content committed through the fabric like the decision record, so it survives adoption on another machine:
`commitDiscussion`'s pathspec gains the file when it exists or is tracked at HEAD, so the decision entry and the amendment record land in one commit, unless a status commit, which stages the loom durable directory `_lyx/loom` whole (`statusCommitPathspec`), lands the amendment record first.
Plan-Bouncer's plan commit pathspec and PR-Rework's commit pathspec gain it on the same condition, so each removal below is staged in that commit; the condition keeps git from refusing a pathspec that matches nothing.
The verb writes the amendment record before the append.
When the append returns an error or check findings, the verb restores the amendment record to its bytes before the call, removing it only when the call created it, the same for both.
`AppendDecision` has already restored the decision record to its pre-append bytes after its check ran; on an error before the check, such as a failed write of the record, the verb restores the decision record to its bytes before the call.
When HEAD then holds this call's amendment bytes, a status commit landed them, so the verb commits the restored amendment record before it refuses.
When the commit fails, the verb reads both files as committed at HEAD through the fabric: the amendment record, and the decision record as `committedHoldsEntry` reads it for the entry's heading.
When the amendment record equals the bytes this call wrote, whose nanosecond write time no other call shares, and the decision record holds the entry's heading, the commit landed: the verb keeps both files and reports success with the commit error noted.
When the amendment record differs, it restores both files to their bytes before the call, removing the amendment record only when the call created it, and refuses with a way forward that re-runs `lyx loom decision add … --amends-plan`, never `lyx fabric commit`.
When HEAD holds this call's amendment bytes but not the entry's heading, a status commit landed the amendment record alone: the verb keeps both files and refuses with `lyx fabric commit` as the way forward, which commits the entry.
When HEAD cannot be read, it keeps both files and refuses with `lyx fabric commit` as the way forward, since re-running the verb over a landed commit would append a duplicate entry.
A call without the flag keeps its appended entry and its `lyx fabric commit` way forward.
The amendment record lists the titles of every pending amending entry, a later call appending to it, and the latest write time in RFC 3339 UTC at nanosecond precision.
Plan-Review's settle record gains `round_started_at`, the start of the round the settle judged at nanosecond precision, which the Bouncer reads from the review half's `start` in that round's `round-<N>-usage.yaml`.
A settle that judged no round, which is every round-0 settle `approveWithoutReview` writes, records none.
A settle record with no or a zero `round_started_at`, such as a round-0 settle, an older record, a round whose usage write failed or a review half with no terminal result, has no known round start: the guard admits a goto over it whenever the amendment record exists, and its approved settle never removes the record.
Every comparison below reads both times as instants.
The settled-segment goto guard admits a goto onto `Plan-Burler-Review` over a live Plan-Review settle record when the amendment record exists and its latest write time is after that settle's round start, or that settle has no known round start; that is its only new admission.
While the amendment record exists, Plan-Review's skip seam (`loomshed.PlanReviewSkippable`) answers false, so the fixed plan of an exempt rework generation is judged and its settle records a round start.
The Plan-Burler-Review answer-key instruction gains: when the amendment record exists, each decision-record entry it names is a finding for each card that entry names while the entry's change is uncarried for that card, or, when the entry names no card, a finding until some card carries it.
An entry's change is carried for a named card when that card carries it, or, for a named card whose batch is done, when a card inserted with `lyx loom insert-card` after the last begun batch carries it.
A decision names each card by slug as well as number, because `lyx loom insert-card` renumbers every later card.
The finding is fixed by editing only cards Webster's auto-rebaseline accepts, a pending card or a card of a failed or in-flight batch, or by inserting a card with `lyx loom insert-card` after the last begun batch.
The existing plan-conflict clause, which today allows only cards whose batch has not begun, is reworded to the same rule, so the two clauses agree when both bind one round and a card of the failed batch that stopped on the defect stays editable.
The amendment record is removed by Plan-Bouncer's approved settle, inside its commit seam, only when the settle's recorded non-zero `round_started_at` is after the record's latest write time, comparing the round's start and never its end or judge time, so an amendment appended while a round runs keeps the record for one more round.
A settle with no recorded round start keeps the record, and the next approved settle with a later start removes it.
PR-Rework's archive of a generation also removes it, staged in that round's commit.
The route is then the existing one: the review round, the fix, the plan gate, Plan-Bouncer, its commit seam, and the Webster row's auto-rebaseline on entry.
Because the review round lands after the settled round, Plan-Bouncer re-enters on the round-by-round path, not as an amendment generation: it judges the fixed plan on the Plan-Review segment's already-spent bounce budget, and a judge that does not converge halts on that budget for the parent's `lyx loom resume`, as the existing Publish-failure goto onto Webster-Burler-Review does.
Webster's plan-conflict recurrence halt names this route as its way forward instead of a hand edit.
`--amends-plan` is refused on a darn run and on a run whose current generation has no approved plan, naming the reason.

**Rationale:** The plan fixer is the one role that already edits an approved plan under the plan gate and the commit seam, and the Webster row already accepts what it changes; the only missing piece is a way in over a settled verdict.
A goto onto Plan-Bouncer cannot carry it, since an unchanged plan re-settles with no spawn.
The bound: the admission re-opens only Plan-Review, only for an amendment written after its last judged round started or over a settle with no known round start, and the fix still passes the plan gate (`done-card-edited` included) and a Plan-Bouncer judgment before Webster sees it.

### D8 — The arbiter and the orch take that route

**Decision:** The arbiter prompt gains an input, `amendment_target`, rendered into the arbiter stencil beside the envelope path.
Loom's arbiter wiring in `internal/loomcli` fills it from a new top-level recipe key, `amendment_target`, which `contracts/recipes/loom-recipe.yaml` sets to `Plan-Burler-Review`.
It fills the row only once the current generation's plan is approved, the condition D7's refusal reads, and otherwise the literal `none`, as the stencil's other absent inputs are; a recipe that sets no key always gets `none`.
The generic step envelope the driver saves is unchanged.
The arbiter stencil stays recipe-blind: a stop whose cause is a plan defect that the board entry and the decision record already settle is the arbiter's call when `amendment_target` names a row; it records the amendment with `lyx loom decision add --by arbiter … --amends-plan` and writes a `resume` verdict whose `goto` is that target.
Its hand-on rule is unchanged: an amendment that needs an operator decision or widens scope is handed on.
The orch stencil: a design call that amends the plan is recorded with `--amends-plan`, then `lyx loom goto --to Plan-Burler-Review` and `lyx loom resume`, never by editing a card.
Both word the decision as a concrete change to named cards, each named by slug as well as number as D7 requires, so the fixer can apply it as a finding.
Each new `--amends-plan` call re-admits one goto, and the route's repeats are bounded as follows.
The arbiter's repeats are capped by the run's arbiter spawn budget, `arbiter_max_spawns`, counted from the attempt records on disk.
The parent's or the orch's repeats have no cap: each spent Plan-Review bounce budget pauses the judge for the parent's `lyx loom resume`, and the parent's own judgment bounds how often it records and resumes again.
A scope widening is bounded only by the hand-on rule, since the arbiter's entry joins the answer key that Plan-Review and PR-Gate's alignment agent judge against.

**Rationale:** The arbiter already records decisions and names goto targets at or before the current row, and Plan-Burler-Review is before every row after plan approval that can stop on a plan defect; a prompt input filled from a recipe key keeps the stencil fit for `shed-arbiter-all`'s move of the arbiter to every recipe.

### D9 — A Merge-In or Publish verify failure is traced to plan cards

**Decision:** When Merge-In or Publish writes the Publish failure record on a loom run, Go also traces the failure:

1. It extracts `path:line` locations from the tails of the failed log's failure identities (`ParseVerifyFailures`) and places each relative to the repo root: a path that already exists at the record's `Head` as written; an absolute path under the worktree by stripping the worktree root; a path that opens with the module path from the worktree's `go.mod` and a slash by stripping that prefix; a bare file name or a `./`-relative one through its identity's package directory, the import path less the module path.
   The module-path placement serves panic frames, which read `<module path>/<dir>/<file>.go:N` because the hub's gate tiers set `GOFLAGS` to `-trimpath` (the hub's `gate.yaml`; `trimpathLine` in `internal/gatecli/tiers.go`).
   Any other location, such as a standard-library or dependency frame, is dropped.
   An opaque identity, such as a `go vet` tier's, carries no package, so its bare or `./`-relative locations cannot be placed and go into `untraced` as written.
2. It calls quarry's `EncloseAt` at the record's `Head` with the placed locations, capped at 20, through `internal/planglyph`, which owns the quarry repo handle.
3. It reads each flat target of every live plan card through `internal/planparser`'s ref-to-member mapping (`RefMember`), so a `plan:` handle compares as the glyph it becomes; a target `RefMember` does not map, such as a file or a self glyph, compares as written, a `plan:` handle as its body.
   It then matches each found member glyph to every live card whose targets hold it under D9's own overlap rule (the glyph itself, a containing type, its unit's self glyph, or its file), with an external test unit mapped through `internal/planparser`'s existing adjustment.
   A `not_found` answer, a location inside no member such as an unused import, matches through its `Unit`, the file's self glyph.
   For it the trace resolves each member-glyph target of the live cards at the record's `Head`, through `internal/planglyph`, which owns the quarry repo handle, and takes the file each resolves into.
   The answer then matches every live card whose targets hold that file, its file self glyph, or a member resolving into it; a member target that does not resolve at `Head`, such as one its card deletes, renames or moves out of the file, resolves at the start SHA of its card's batch, and one that resolves at neither matches only under the file and unit rule.
   With no such card it goes into `untraced`.
   It reads each matched card's batch from Webster's run state by the live card id.
   Webster commits each card but records only the batch head, in the batch's `CardSHAs` element of Webster's state (`BatchState.CardSHAs`), which the done paths set.
   A Webster row reaches Done only once every batch is terminal done (`verifyEveryBatchDone`), and Merge-In and Publish run after it, so every matched batch is done.
   The trace takes the batch's commit range from its start SHA to its recorded head; a batch whose record holds no `CardSHAs` element, such as one written before the field, gives the card an empty range.
   It also takes the later range, from the recorded head of the highest-numbered matched batch that has one to the record's `Head`, which holds every commit after that batch that can have rewritten the matched members: the verify-gate fixer's `fix:` commits, Webster-Review fixes and the merge-in.

The record gains `cards`: per matched card its number, the glyphs and locations matched, and its batch and that batch's commit range (empty for a card Webster holds no record of, or whose batch record holds no `CardSHAs` element); `later_range`: the later range of step 3, empty when no matched batch has a recorded head; and `untraced`: an opaque identity's bare or `./`-relative locations, and the locations quarry rejected, answered `not_found` and matching no card, or found in a member no live card's targets hold, as `path:line`.
`checkedPublishFailure` shape-checks the new fields as it does the others, dropping a field that fails.
`PublishFailureNote` renders them after the failing tests: each card with its number, and its slug and first `Intent` line read from the live plan at render time, never stored in the record, and its batch's commit range, as the range to diff when starting the fix; then the later range, as the later commits touching the same members; then the untraced locations.
A trace failure (no plan, no Webster state, a quarry error) is logged at Warn and leaves the record without the fields; it never changes the Stuck verdict or the record's other fields.

**Rationale:** The note is what the Webster-Review reviewer and fixer read after such a failure, so naming the card there needs no new channel.
Quarry v0.4.0, which `go.mod` already pins, ships `Enclose` and `EncloseAt` (`quarry/enclose.go`), so no module bump is needed.
Reading slug and intent from the plan keeps the note's rule that no free text reaches the prompt from the record.

## Constraints

- `PATTERN-glyph-conversion-chokepoint`: Enclose is a location-to-glyph conversion done by quarry; it becomes the one sanctioned such call, in `internal/planglyph`, added to the entry's background file; the overview line is unchanged.
- `PATTERN-tolerant-records`: the Publish failure record's new fields and the settle record's `round_started_at` are optional, and an older record without them reads as before, a settle record as one with no known round start.
- `PATTERN-durable-vs-ephemeral-state`: the run's routing reads the amendment record, so it is tracked content at `_lyx/loom/amendment.yaml`, committed with the decision entry it names unless a status commit lands it first (D7).
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

- The trace often names no card: Go test failure lines name only the file's base name, a failure in production code that only a test's assertion reports traces to the test, members a retired PR-Rework generation built trace to no live card, and an opaque identity, such as a `go vet` tier's, carries no package, so its bare file names stay untraced; the untraced list and the failing tests stay in the note for those cases.
- The hub orch rule allowing a hand edit of a card now conflicts with the orch stencil (D8); the orch retires it with `lyx orch rules` after landing.
- A loom profile without the Plan-Review perch (`speed-loom-profile`) would leave the amendment with no fix loop.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - d7-answer-key-carry-test-done-and-renumbered-cards (class decision, severity MEDIUM, open in rounds 3)
  - d9-plan-handle-targets-never-match (class decision, severity MEDIUM, open in rounds 2, 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A plan whose cards' verbatim lines grow the projected overview past the headroom line from the merge base fails the plan gate with `pattern-headroom`, naming the bytes to cut; a plan whose lines replace existing bullets at equal or smaller size passes.
- `lyx loom validate-plan` and the row gate give the same headroom verdict on a branch that already grew `PATTERN.md`.
- A plan that declares `removes` for an entry passes with the projection smaller by that bullet; the same plan with a card fencing a line for that ID fails, naming the bullet and the card.
- Two cards prescribing a line for one ID project the later card's line; two fenced lines for one ID in one card project the last.
- A rework plan whose earlier generation already wrote its lines to `PATTERN.md` projects the same size as before they were built, and its declared entries raise no `pattern-line-unprescribed`.
- A declared `adds` or `changes` entry with no verbatim line and an unchanged bullet fails with `pattern-line-unprescribed`; the same plan with the line fenced on a card passes.
- A card with an `**Extends:**` group parses with `CardTypeExtends`; an `Extends` ref that does not exist, or that also sits under another label or `Uses:` of the card, is a blocking finding; an `Extends` ref to a symbol an earlier card deletes is flagged as an `Edit` ref is; a re-sign arrow sub-bullet under `Extends` is a blocking finding whose way forward moves the ref under `Edit`.
- Two cards in reverse batch order whose only shared ref sits under `Extends` on both pass the batch-order check; an `Extends` against an `Edit` on the same ref, or against a `Uses:`, is still refused, and so are two `Extends` on `PATTERN.md` that fence a line for one entry ID.
- `tools/plandag` draws `E` cells and drops `Extends`-against-`Extends` edges; with `-additive-guess` it reports both critical paths, and a labelled ref keeps its label.
- The plan-review rubric names the `Extends` check, and the plan-writer stencil says when to use `Extends`.
- `lyx loom decision add --amends-plan` lands the decision and the amendment record in the verb's own commit, and a second call appends its title.
  A commit that did not land, and an append refused with check findings, leave the decision record holding no new entry and the amendment record as before the call, so a failed second call keeps the first call's title, with a way forward that re-runs the verb; a commit error after the commit landed keeps both files and reports success with the error noted; a commit error after a status commit landed the amendment record without the entry keeps both files and refuses with `lyx fabric commit` as the way forward.
  On a darn run, or with no approved plan, it is refused with its way forward.
- With an amendment record written after the start of the round the Plan-Review settle judged, including one written while that round ran, `lyx loom goto --to Plan-Burler-Review` is admitted; with none, or one written before that round started, it is refused as today.
- On a rework-exempt generation settled at round 0, an amendment written after that settle is admitted, the re-opened round's Plan-Bouncer judges the fixed plan instead of skipping, and its approved settle removes the record; a round-0 settle never removes the record.
- An approved Plan-Bouncer settle that judged no round started after the amendment record's latest write keeps the record; one that did removes it; an amendment appended while a round runs keeps the record through that round's approved settle; an approved settle with no recorded round start keeps it; PR-Rework's archive removes it.
- The Plan-Burler-Review instruction names the amendment record and the same editable-card rule as the plan-conflict clause.
- An amending entry naming a done card is satisfied by an inserted card that carries the change, and the next round raises no finding for the done card.
- An arbiter prompt for a stop after plan approval on a loom run names `Plan-Burler-Review` as `amendment_target`; one for a stop before plan approval, or from a recipe that sets no `amendment_target` key, names `none`.
- The orch stencil names `--amends-plan`, then `lyx loom goto --to Plan-Burler-Review` and `lyx loom resume`, as the route for a design call that amends the plan.
- Webster's plan-conflict recurrence halt names the amendment route.
- A loom Merge-In verify failure whose log places a build error or a test failure in a member a card targets writes a record whose `cards` names that card, its batch and that batch's commit range, and the Publish failure note shows the card's number, slug, intent and batch commit range.
- An unused-import build error in a file a card targets names that card, and so does one in a file where a card targets only a member glyph; a failing test a card creates under a `plan:` handle names that card.
- A matched card of a batch whose record holds no `CardSHAs` element takes an empty range; a matched card of a done batch followed by a verify-gate fixer's `fix:` commit makes the note show the later range from that batch's head to the record's `Head`.
- A panic frame under `-trimpath`, reading the module path then a repo-relative file, is placed and traced as a repo-relative location.
- A location quarry rejects, such as a `.md` file, one no card's targets hold, or a bare file name of an opaque identity, appears in `untraced` as `path:line`.
- A quarry error during the trace leaves a record with the other fields and the same Stuck reason.
- An older Publish failure record without the new fields still reads and renders.

## Notes for the plan writer

- `PlanContext` in `internal/loomshed/plancontext.go` needs the merge-base reader, wired where the verify gate's `VerifyMergeBase` is, and in the `validate-plan` CLI.
- The projection is pure byte work and fits `internal/pattern` beside `Headroom`; the plan gate reads the files.
- Webster's `deriveEdges` compares flat `Targets`, so the pair rule needs each ref's group type and each card's fenced `PATTERN.md` entry IDs.
