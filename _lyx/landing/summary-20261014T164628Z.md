# Darn: decision record for scope corrections, and PR-Align

A scope correction made while a darn run is in flight now reaches every later row of that run, and a darn pull request no longer waits for the orch at PR-Gate when the alignment agent can settle it.

Before this change a darn run had no decision record, so a correction the parent sent to the live writer was never read by the review (the `webster-pack-last` run reverted a correct change).
It also had no PR-Align row, so every darn PR went to the orch.

## What changes

- Darn-Write writes `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md` before any code, at the paths `loomengine` declares and with loom's required sections.
  Once `lyx loom validate-discussion` passes, the writer never edits the record again.
  A later spawn writes the two files only when they are absent or invalid.
- The record format moves out of the Discussion stencil into `loom-discussion-spec.md`, served through `stencilstore`; both the Discussion and darn stencils point at it.
- The Darn-Write row gains the `discussion` gate.
  On a darn run the writer's first clean `validate-discussion` commits the record and support log through `fabricengine`, and the gate's pass commits again.
- `lyx loom decision add` works on a darn run once the record is committed, including while Darn-Write runs.
  It refuses a record that is uncommitted, or whose working copy differs from its committed copy, with the way forward to message the Darn-Write session.
  After its commit it confirms the committed record holds its own entry.
- A record missing an `Added after Discussion` heading that any earlier commit of the record held is refused by the handoff gate, by a later clean `validate-discussion` and by `decision add`, naming the missing entries.
  `fabricengine` gains `CommittedAnchoredFileHistory` and `discussionparser` gains `CheckAddedDecisionsKept` for this.
- Darn-Review measures against the board entry as amended by the record's `Added after Discussion` entries, which win over the board body; the writer's own sections are a claim.
  A deviation the change description names as an unrecorded parent correction is filed as a MEDIUM `decision`-class finding whose fix text leaves the code in place and defers to the parent.
  Darn's Describe stencil reads the record too.
- Darn-Bouncer gains `carry_over: Darn-Review`, so findings still open at accept land under the record's `## Open risks`.
- The darn recipe gains a `PR-Align` row between Publish and PR-Gate, wired with darn's geometry (record and support log, plan directory `none`) and reading loom.yaml's alignment keys.
  The alignment stencil names no recipe row; what `pr-gate` measures against, and its scope-deviation definition, come from a told value.
  On a darn run an open risk naming an unrecorded parent correction is escalated to the parent, never approved or rejected.
- The orch role stencil records corrections through `decision add` whenever a record exists, and describes darn's PR-Gate as an alignment escalation.
- `darnengine`, `alignment` and `loomcli` docs, the darn recipe header, `docs/overview.md` and `PATTERN` follow.

## Design decisions to know

- An agent approve now lands a darn run, and batten's Land row deploys with no orch turn.
  It is bounded as in loom: `alignment_pr_approve: false` or `skip_alignment: true` in loom.yaml leaves PR-Gate to the parent, the decision-snapshot and head checks apply, and one agent reject is allowed per run.
- The darn record's own sections are the writer's unreviewed calls, so the agent and the review treat them as a claim, not as authority.
- No darn row is renamed.
  A darn run in flight when this lands must be restarted, not resumed; a resumed run halts at Darn-Bouncer's carry-over, whose way forward names the missing record.
- The writer cannot fix a record it has validated, so an alignment reject that the writer's own scope reading is wrong is fixed only by the parent's `decision add`, or reaches the parent on a second reject.

## To check by hand

- `decision-record` D7 and its acceptance line on Darn-Bouncer: a MEDIUM `decision`-class entry never makes the judge rule CONTINUE, but in the soft-cap window another finding can make the open entry count as design-wrong and escalate to the parent.
  Check that the wording in `_lyx/discussion/decision-record.md` D7 and the Bouncer stencil's behaviour agree.
- `_lyx/plan/06-validate-discussion-darn.md:25`: the weft commit `validate-discussion` now makes on a darn run is a new side effect of a verb any session in the worktree may run.
  A writer edit made after validation and then re-validated is committed, so `decision add`'s working-copy-equals-committed-copy check passes over it.
  Confirm the bound holds: darn seed only, the discussion directory pathspec only, and a record that passes the whole darn check including the history check.
- The first verify attempt failed `TestGofmt_ModuleFilesAreFormatted` and the fabric-vocabulary scan; both were fixed in `75724e18b`.
  The integration tier was run only for `cmd/lyx`, `internal/lyxcwd`, `internal/loomengine` and `internal/loomcli`; run `lyx gate test --tags integration ./...` before pushing.
- Stencil-wording tests pin the Discussion stencil's Step 5 text, which moved into the spec; expect a merge with `twill-spawn-after` in that stencil and with `orch-split` in the orch role stencil.
