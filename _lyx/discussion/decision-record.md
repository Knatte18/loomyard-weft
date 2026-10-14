# Decision record — claimed-scope-edit

## Goal

A mid-run scope correction on a darn run reaches every later row of that run, and a darn pull request no longer waits for the orch at PR-Gate when the alignment agent can settle it.

## Scope

In:

- The decision-record format moves out of the Discussion stencil into a spec both writers point at.
- The darn writer's stencil: write the record and support log before any code; follow a parent's recorded correction.
- A `discussion` gate on the Darn-Write row of `contracts/recipes/darn-recipe.yaml`, and a Go commit of the two files at the writer's first clean validation.
- `lyx loom decision add` on a darn run: its no-record way forward.
- The Darn-Review perch's scope source: the rubric, the review row's `fasit` instructions, and a `carry_over` on Darn-Bouncer.
- Darn's Describe stencil reads the record.
- A PR-Align row in the darn recipe, wired with darn's geometry, and the alignment stencil's recipe-neutral wording.
- The orch role stencil's guidance on recording a correction and on darn's PR-Gate.
- Docs and recipe comments that describe darn as having no record or no PR-Align.

Out:

- Editing a claimed board entry's body, or appending amendments to it.
- Splitting darn into Discussion-Write, Plan-Write and Webster rows.
- Loom's own pre-record gap (`decision add` refused before Discussion-Write has written the record): unchanged.
- New `darn.yaml` keys: darn reads loom.yaml's alignment keys.
- The twill fork-spawn order, which `twill-spawn-after` owns.

## Decisions

### D1 — The darn record is loom's record, the same two files at the same paths

Darn-Write writes `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md`, at the paths `loomengine`'s accessors declare, with loom's required sections.
The support log is short: `## Interview` states that a darn run holds no interview, `## Rejected alternatives` and `## Question ledger` carry the writer's own considered options and self-picks, and `## Review rounds` is seeded as loom seeds it.
Rationale: every existing reader and writer of `_lyx/discussion/` works unchanged, and `discussionparser.Validate` requires the support log to exist.

### D2 — The record's format is a spec, and both writers point at it

The decision-record and support-log format, today in `loom-template-discussion.md`'s Step 5, moves into a new spec `contracts/specs/loom-discussion-spec.md`, registered with the other specs and read at call time.
The Discussion stencil gains a specs-directory marker for the pointer; the darn stencil already has one.
Both stencils point at the spec and restate none of it.
The Discussion stencil keeps its own Discussion-only rules (the fork lens procedure, the self-check, the gate messages).
Rationale: `PATTERN-producer-pointer-rule` forbids the darn stencil paraphrasing the format, and pointing it at the whole Discussion prompt would hand it Discussion's interview and fork procedure.

### D3 — Darn-Write writes the record before any code, and leaves a valid one alone

The darn stencil's order becomes: read the task, write the two files from the board entry and a coarse reading of the code, run `lyx loom validate-discussion` until clean, then make the change.
The record carries the scope as the writer reads it and the calls it makes before coding; a call it makes later while coding goes into the change description's design decisions, as today.
Once `validate-discussion` passes, the writer never edits the record again, in this spawn or a later one.
A later spawn of Darn-Write (a rejection or prior work) writes the two files only when they are absent or fail `validate-discussion`, which covers a writer that died mid-write.
The writer commits neither file; Go does (D4).
Rationale: the record must exist before a correction arrives for the correction to have a home, and a record the writer keeps rewriting would race the parent's appends.

### D4 — A `discussion` gate on Darn-Write, and a Go commit at its pass

The Darn-Write row's `gates` list gains the existing `discussion` gate, ahead of `verify` and `description`, with Discussion-Write's attempt count.
A handoff whose record or support log is missing or lacks a section re-prompts the same session, and an exhausted budget halts the run for the parent, as the other Darn-Write gates do.
On a darn run, the writer's first clean `lyx loom validate-discussion` commits the two files through `commitDiscussion`, the fabric commit `lyx loom decision add` uses, so the record is committed before any code is written.
A later clean run, and the gate's pass, commit again; a commit with nothing new commits nothing, and the gate's commit covers a writer that never ran the verb.
The handoff `discussion` gate stays the check, and on a darn run it, like a later clean `validate-discussion`, also refuses a record missing any `Added after Discussion` heading that any commit of the record in the weft's history has held, naming the missing entries for the writer to restore.
The reference is the record's whole commit history, never its latest committed copy, so a commit of the record from outside Go, such as `lyx fabric commit`, cannot erase it.
Rationale: a record the stencil asks for but nothing checks is skipped on the run that most needs it, in loom too it is Go, never the writer, that commits `_lyx/discussion/`, and a record committed at the writer's validation is what lets D5 check mechanically that the writer has finished it.

### D5 — `decision add` on a darn run

The verb works on a darn run once the record is committed (D4), including while Darn-Write runs.
Its no-record way forward drops "a darn run never has one" and says: on a darn run the record exists once Darn-Write has validated it; until then, message the running Darn-Write session, or, when no writer is live, record the call where the orch stencil's fallback says (a board note naming the task).
On a darn run the verb refuses a record that is not yet committed, whose working copy differs from its committed copy, or that lacks an `Added after Discussion` heading the record's commit history has held (D4's check), with the way forward to message the running Darn-Write session: the writer has not finished its record, or is editing it against D3.
An append that fails the verb's existing post-append discussion check is still restored and refused.
After its commit, the verb confirms the committed record holds its own entry, and refuses naming the entry when it does not, which covers a writer write landing between the verb's append and its commit.
Go's commits of `_lyx/discussion/` are serialized by the weft write lock, so the verb's commit and D4's commits need no ordering of their own.
Bound: the verb's caller rule (parent, operator shell, or the arbiter for `--by arbiter`) and its append-only, labelled, committed entry are unchanged; the one new edge is appending during Darn-Write, bounded by the committed-copy and history checks to a record the writer has validated and not touched since, by the post-commit check to an entry that reached the weft, and a later writer overwrite that drops an appended entry is refused by D4's history check even when the writer has committed the overwrite itself.
Rationale: the scope correction in the board entry arrived while Darn-Write ran, so the verb must work then.

### D6 — Darn-Write on a parent's correction

The darn stencil tells the writer: a scope correction from the parent is recorded by the parent with `lyx loom decision add`; on such a message the writer re-reads the record's `Added after Discussion` entries and follows them over the board body.
When the message names a correction the record does not yet hold, the writer asks the parent to record it, follows the message meanwhile, and names the unrecorded correction in its change description's design decisions.
Rationale: the writer may not run the verb itself (it refuses a task session), and the record, not the message, is what the later rows read.

### D7 — Darn-Review and Describe read the record; the parent's entries override the board body

The Darn-Review rubric's measuring stick becomes the board entry as amended by the record's `Added after Discussion` entries: an entry by the parent, the operator or the arbiter wins over the board body where they differ.
The writer's own sections of the record are a claim, checked against the board entry the way the rubric checks the change description.
A deviation the change description names as an unrecorded parent correction (D6) is never fixed by revert.
The rubric files it as a `decision`-class finding at MEDIUM whose fix text tells the fixer to leave the code in place and defer the finding, naming the parent as the party who records the correction with `lyx loom decision add` or refuses it.
Darn-Bouncer accepts with that entry open: a MEDIUM `decision`-class entry never makes the judge rule CONTINUE, so a judged CONVERGED round accepts with it open; in the soft-cap window, a round that another finding holds at CONTINUE or CIRCLING counts the open entry as design-wrong unless the judge tags it `mechanical`, so it can block that round's automatic accept and, open across a recorded continue, escalate with cause circling to the parent, the party who records or refuses the correction; and D8's carry-over, which takes every open entry at MEDIUM or worse, files it under the record's `## Open risks` for the alignment agent to escalate (D9).
The bound on who writes the overriding entries is the verb's caller rule: the parent, the operator's unnamed shell, and the arbiter for `--by arbiter`, with `--by` an unverified label.
The Darn-Burler-Review row's `fasit` instructions in `darn-recipe.yaml` say the same.
Darn's Describe stencil reads the record beside the board entry for the scope and decisions it describes.
This narrows the board entry's "holds the record authoritative over the board body" to the record's post-writer entries, since the darn record, unlike loom's, is reviewed by no one before coding.
Rationale: the review today reads the board body live through `lyx board get`, which is why the `webster-pack-last` correction was reverted; a writer-authored answer key would let the writer's own misreading grade itself.

### D8 — Darn-Bouncer carries the perch's open findings into the record

Darn-Bouncer gains `carry_over: Darn-Review`, so findings still open when the perch accepts land in the record's `## Open risks`, as each loom review segment does.
The recipe comment saying darn has no record to file them in goes.
Rationale: the alignment agent at `pr-gate` judges against the record's open risks, and a darn run now has a record to hold them.

### D9 — PR-Align in the darn recipe, between Publish and PR-Gate

The darn recipe gains a `PR-Align` row with engine `PRAlign`, `on_done: PR-Gate`, outside the PR-Review segment, as in loom; Publish's `on_done` becomes `PR-Align`.
It is wired with the run's own geometry: the decision record and support log at D1's paths, and no plan directory; the alignment prompt renders an empty plan directory as `none`, as it already does for the brief path, the pull request number and the head.
It reads loom.yaml's alignment keys (`skip_alignment`, `alignment`, `alignment_timeout_min`, `alignment_pr_approve`), as darn's perch reads loom.yaml's Webster-Review model keys.
An agent reject writes the rejection record and PR-Gate bounces to Darn-Write, which renders it as rejection findings; the once-per-run reject limit holds as in loom.
On a darn run the agent at `pr-gate` measures the diff against the board entry as amended by the record's `Added after Discussion` entries, and treats the writer's own sections as a claim, as D7 does for Darn-Review; darn's wiring delivers this as a told prompt value, so the stencil names no recipe row.
On a darn run, an open risk carried over from Darn-Review that names an unrecorded parent correction is escalated to the parent at `pr-gate`, never approved and never rejected, since only the parent can record or refuse it; the same told value carries this rule.
Bound: an agent approve now lands a darn run and batten's Land row deploys with no orch turn; that is bounded as for loom, by `alignment_pr_approve`, `skip_alignment`, the decision-snapshot and head checks, and one agent reject per run, and in addition by the rule above, since the darn record's own sections are the writer's unreviewed calls and with `review_max_bounces: 0` nothing else checks them.
Rationale: the operator asked for it, and the record was the only thing darn lacked (A1).

### D10 — The alignment stencil names no recipe's rows

`loom-template-alignment.md`'s `pr-gate` judge text says "the open risks the review segments carried over" instead of naming `Merge-Review`, and takes what the run is measured against from a told value: loom's wiring tells the board entry's goal and the record's decisions, darn's tells D9's rules.
The `pr-gate` scope-deviation definition (today a change the goal does not call for, a goal point left out, or a recorded decision the change breaks) comes from the same told value: loom's wiring keeps the recorded-decision clause, and darn's names only the board entry and its `Added after Discussion` entries, with the writer's own decisions a claim.
Rationale: the stencil states that its body names no recipe row, and darn's review segment is Darn-Review.

### D11 — The orch records a correction through `decision add` whenever a record exists

The orch role stencil drops darn from the runs `decision add` refuses on.
A scope correction on a run whose record exists goes through `lyx loom decision add --by parent` even while an agent of the run is alive, and also in a message to that agent; the board-note fallback stays for a run with no record yet.
Its line that darn's PR-Gate is the operator's review point says instead that PR-Gate reaches the orch as an alignment escalation, as in loom.
Rationale: the orch's current guidance, a message to the live agent, is the path behind `webster-pack-last`; without this the correction still never reaches the record.

## Constraints

- `PATTERN-producer-pointer-rule`: the darn stencil points at the new spec and never paraphrases it (D2).
- `PATTERN-sole-parsers`: the record is read and written only through `discussionparser`; no darn-specific parser.
- `PATTERN-stencil-ownership`: the new spec is read at call time through `internal/stencilstore`, registered like the other specs.
- `PATTERN-told-geometry`: `darnengine` and `alignment` stay told their paths; the record paths come from `loomengine`'s accessors in `loomcli` wiring.
- `PATTERN-refusal-way-forward`: the changed `decision add` way forward lands its spec row and a test, and the exhausted-`discussion`-gate refusal row covers Darn-Write.
- `PATTERN-no-denied-recovery`: no way forward names a command the writer's settings deny; the writer is told to message its parent, never to run `decision add` or to commit `_lyx`.
- `PATTERN-fabric-git`: D4's commit goes through `fabricengine`, never raw git.
- `PATTERN-parent-directive`, `PATTERN-edit-directive`: the darn and Describe stencils keep rendering both directives.
- Recipe row names are durable resume identities: no darn row is renamed.
  A darn run in flight when this lands is restarted, not migrated, per the recipe's own policy for changed edges.
- Docs land in the same commit as the code they describe.

## Auto-mode assumptions

- A1: darn left PR-Align out because it had no decision record; nothing else in the alignment agent depends on loom-only state.
- A2: loom.yaml's alignment keys govern darn too, so `alignment_pr_approve: false` withdraws darn's landing as well.
- A3: the darn support log is required rather than dropped through a darn-only validator, keeping one validator and one append path.
- A4: in-flight darn runs are restarted rather than given a record-less fallback, so every darn row after Darn-Write may assume the record exists.
- A5: the Darn-Write row's `discussion` gate takes Discussion-Write's attempt count.

## Open risks

- `twill-spawn-after` is editing `loom-template-discussion.md`'s fork procedure while this moves its Step 5 into a spec; expect a merge in that stencil.
- `orch-split` restructures the orch role stencil D11 edits.
- Stencil-wording tests pin the Discussion stencil's Step 5 text; moving it shifts those pins (`lighter-test-rules` names the cost).
- A darn run resumed despite A4 halts at Darn-Bouncer's carry-over, whose way forward names the missing record.
- The writer may not edit its record after it validates, so an alignment reject that the writer's own scope reading is wrong is fixed only by the parent's `decision add`, or reaches the parent on a second reject.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - d7-bouncer-premise-false (class design, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-1-review.md; fixer report: _lyx/reviews/plan/round-1-fixer-report.md.
  - card6-validate-discussion-commit-unbounded-intent (class scope, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- The writer's first clean `lyx loom validate-discussion` on a darn run commits the record and support log.
- A darn writer handoff lacking the record or support log, or with a section missing, is re-prompted by the `discussion` gate; one with both, valid, passes it.
- On a darn run, the handoff `discussion` gate and a later clean `validate-discussion` refuse a record missing an `Added after Discussion` heading any earlier commit of the record held, naming the missing entries.
- A darn record committed by `lyx fabric commit` without an earlier appended entry is still refused by the handoff `discussion` gate, naming the entry.
- `decision add` on a darn run refuses, naming its entry, when the committed record does not hold the entry after the verb's commit.
- A Darn-Write respawn with a valid committed record leaves it unchanged; one whose record is absent or invalid writes it.
- `lyx loom decision add --by parent` on a darn run appends and commits an entry once the record is committed, while Darn-Write runs and after, from the parent and from the operator's shell.
- On a darn run, `decision add` refuses a record not yet committed, or whose working copy differs from its committed copy, with the way forward to message the Darn-Write session.
- An append to a darn record that fails the discussion check is refused and the record restored.
- On a darn run with no record, `decision add` refuses with the new way forward, which no longer says a darn run never has one.
- A task session other than the parent or arbiter is still refused on a darn run.
- The darn recipe loads with PR-Align between Publish and PR-Gate, its reroute edges intact, and with `review_max_bounces: 0` still routing Darn-Write's Done to Merge-In.
- PR-Align on a darn run spawns the agent with the darn record and support log and `none` for the plan directory.
- A darn PR-Align prompt carries the rule that the diff is measured against the board entry as amended by `Added after Discussion` entries, with the writer's own record sections as a claim, and a scope-deviation definition that names only those, never a breach of the writer's own decisions.
- A loom PR-Align prompt keeps the scope-deviation definition's recorded-decision clause.
- A darn PR-Align whose record's `## Open risks` holds a carried-over Darn-Review finding naming an unrecorded parent correction escalates to the parent.
- The alignment stencil's `pr-gate` judge text names no recipe row and reads the open risks the review segments carried over.
- An agent reject on a darn run routes PR-Gate's bounce to Darn-Write with the rejection rendered as rejection findings; a second reject goes to the parent.
- A `decision add` recorded while the darn alignment agent runs turns its approve into an escalation through the snapshot check.
- `alignment_pr_approve: false` and `skip_alignment: true` in loom.yaml leave a darn PR-Gate to the parent.
- Darn-Bouncer accepting with open findings files them under the record's `## Open risks` with the Darn-Review segment label.
- The Darn-Review rubric and the review row's `fasit` instructions measure against the board entry as amended by `Added after Discussion` entries, and treat the writer's own record sections as a claim, and both say a deviation the change description names as an unrecorded parent correction is filed as a `decision`-class MEDIUM finding whose fix text tells the fixer to leave the code and defer to the parent; darn's Describe stencil reads the record.
- The Darn-Burler-Fix fixer handed such a finding leaves the deviating code in place and defers the finding, naming the parent.
- Darn-Bouncer, judging a round whose only open entry at MEDIUM or worse is such a finding, rules CONVERGED and accepts it without CONTINUE or a circling escalation, and its carry-over files it under the record's `## Open risks`, with the change still in the diff.
- The darn stencil tells the writer to name an unrecorded parent correction in its change description.
- The Discussion stencil and the darn stencil both point at `loom-discussion-spec.md`, served through `stencilstore` like the other specs.
- The orch role stencil no longer lists darn among the runs `decision add` refuses on, and sends a correction on a run with a record through `decision add`.

## Notes for the plan writer

- `landingAlignmentDeps` in `internal/loomcli/landingdeps.go` builds PR-Align's seams from loom geometry; check whether darn's landing deps share it.
- The `discussion` gate's resolver in `internal/shedrecipe/entries_gate.go` reads the record and support-log paths from the build env; check they are set on a darn build.
- The carry-over seam and its missing-record error are in `internal/shedadapters/bouncercarryover.go` and `discussionparser.WriteCarryOver`.
- D4's history check reads every committed version of the record in the weft; check whether `fabricengine` already exposes a file's history, since `PATTERN-fabric-git` rules out raw git.
- `validate-discussion` in `internal/loomcli/validate.go` commits nothing today and reads only the env's path fields; D4's commit needs it to learn the run is a darn run.
- Docs the change touches, as a head start: `internal/darnengine/doc.go`, `internal/alignment/doc.go`, the `loomcli` doc of the decision verb, the darn recipe header comment, and `docs/overview.md` where it describes darn's rows and review source.
