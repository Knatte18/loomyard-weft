# Darn: decision record for mid-run scope corrections, and PR-Align

A scope correction on a running darn task now reaches every later row, and a darn pull request no longer always waits for the orch at PR-Gate.
Before this change a darn run had no decision record, so `lyx loom decision add` was refused.
A correction sent to Darn-Write reached only that session, while the Darn review kept reading the claimed board body.
That is how the `webster-pack-last` correction was reverted by its own review.

## What changes

- Darn-Write writes loom's two files, `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md`, before any code, and runs `lyx loom validate-discussion` until clean.
  It never edits the record after the first clean validation.
  A later spawn rewrites the files only when they are absent or invalid.
- The Darn-Write row gains the `discussion` gate.
  On a darn run the writer's first clean `validate-discussion` commits both files through the fabric commit that `decision add` already uses, so the record is committed before any code exists.
- `lyx loom decision add` works on a darn run once the record is committed, including while Darn-Write runs.
  It refuses a record that is uncommitted or differs from its committed copy, and it confirms after its commit that the committed record holds its own entry.
  Its no-record way forward no longer says a darn run never has one.
- On a darn run, the handoff `discussion` gate, a later clean `validate-discussion` and `decision add` refuse a record that lacks an `Added after Discussion` heading any earlier commit of the record held.
  The reference is the record's whole commit history, so a commit from outside Go, such as `lyx fabric commit`, cannot erase an entry.
  `fabricengine` gains `CommittedAnchoredFileHistory` and `discussionparser` gains `CheckAddedDecisionsKept` for this.
- The Darn-Review rubric and the review row's `fasit` instructions measure against the board entry as amended by the record's `Added after Discussion` entries, which win over the board body.
  The writer's own record sections are a claim.
  A deviation the change description names as an unrecorded parent correction is filed as a MEDIUM `decision`-class finding whose fix leaves the code and defers to the parent.
- Darn-Bouncer gains `carry_over: Darn-Review`, so findings open at accept land in the record's `## Open risks`.
- The darn recipe gains a `PR-Align` row between Publish and PR-Gate, wired with the run's own record paths and no plan directory (rendered `none`).
  It reads loom.yaml's alignment keys, so `alignment_pr_approve: false` and `skip_alignment: true` leave a darn PR-Gate to the parent.
  An agent approve now lands a darn run and batten's Land row deploys with no orch turn.
- The alignment stencil names no recipe row.
  What `pr-gate` measures the diff against, and the scope-deviation definition, come from a told fragment: loom keeps its recorded-decision clause, and darn names only the board entry and its `Added after Discussion` entries.
  On a darn run, an open risk naming an unrecorded parent correction is escalated to the parent, never approved or rejected.
- The orch role stencil records a correction through `decision add` whenever a record exists, and says PR-Gate reaches the orch as an alignment escalation on darn too.
- The record format moved from the Discussion stencil's Step 5 into `contracts/specs/loom-discussion-spec.md`, and both stencils point at it.
- Darn's Describe stencil reads the record.

## Design decisions to check

- The darn record is unreviewed before coding, so this narrows "the record is authoritative over the board body" to the record's post-writer entries.
- In-flight darn runs are restarted, not migrated, because every row after Darn-Write assumes the record exists.
  A run resumed anyway halts at Darn-Bouncer's carry-over, whose way forward names the missing record.
- Because the writer may not edit its record after validating, an alignment reject that the writer's own scope reading is wrong is fixed only by the parent's `decision add`, or reaches the parent on a second reject.
- No darn row is renamed.

## To check by hand

- Finding `d7-bouncer-premise-false` (design, MEDIUM) in the Darn-Bouncer reasoning about the unrecorded-correction finding: confirm a MEDIUM `decision`-class entry never makes the judge rule CONTINUE, and that in the soft-cap window it can still escalate with cause circling, as D7 of the record describes.
- Finding `card6-validate-discussion-commit-unbounded-intent` (scope, MEDIUM) in `validate-discussion` in `internal/loomcli/validate.go`: confirm the commit stays bounded to a darn run's record and support log.
- Stencil-wording tests pinned the Discussion stencil's Step 5 text; moving it shifted those pins.
- `twill-spawn-after` edits the Discussion stencil's fork procedure and `orch-split` restructures the orch role stencil, so expect merges there.
- Not run by hand: a live darn run through Darn-Write, `decision add` mid-run and PR-Align approve and reject.
- Two scope-guard deviations beyond the plan: `contracts/stencils/claims_test.go` and `internal/loomengine/discussiontable_test.go`, plus `internal/loomcli/decision.go`, `decision_test.go` and `validate_test.go`.
- A verify failure on gofmt spacing and a bare weft token was fixed in `75724e18b`: spacing in `internal/loomengine/prompt.go` and `discussiontable_test.go`, and "weft" now reads "fabric" in the `decision add` help text.
