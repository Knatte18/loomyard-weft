# Fixer report — plan, round 1

## Changes

- **F1** fixed as proposed in `_lyx/plan/08-timing-spans-phases.md`.
  The Phases paragraph now puts in `land` every batten row other than `Run-Shed` after batten's first `Run-Shed` entry, and says the batten rows split at that first entry into `setup` before it and `land` after it, as the decision record's table states.
  The `TestComputeTiming` row with one row of every table entry now says the batten row between two `Run-Shed` entries counts under `land`.
- **F2** fixed as proposed in `_lyx/plan/05-history-caller-field.md`.
  `Shed.stepLocked`'s resume write now sets `PendingBy` only when the status it read is blocked, failed or awaiting, matching `RecordResumer`, so a resume from paused records nothing.
  `TestStep_ResumeFromHaltedStates` drops the paused-records-the-label claim and gains a row resuming from paused with `ResumedBy` `orch` that asserts no `by` on the appended entry and an empty `PendingBy`.
- **F3** fixed as proposed in `_lyx/plan/06-caller-label-wiring.md`, without the trailing sentence about the overlay check.
  The Create bullet for `TestBattenAutomaticMoves_LabelShed` is gone.
  `internal/battencli#TestBattenDriverDeps` is on the Edit list and `cmd/lyx/testdata/test-budget.yaml` is off it.
  The test sentence now puts the `childGotoArgv` assertion and the `deps.Goto` and `deps.Step` label assertions in `TestBattenDriverDeps`, and the budget sentence says `internal/battencli` keeps its row of 83, so the budget file is not touched.
  I left out the fix key's sentence "Checked with validate-plan --overlay: no new blocking finding.", because it reports the reviewer's own check and the card says nothing about it.
  Instead I ran `lyx loom validate-plan` after the edits: `ok: true`, with only informational `create-new-unit` findings.
  The current count is 83 top-level tests in `internal/battencli`, not counting `TestMain`, which matches its budget row.

`_lyx/plan/lens-review.md` still names `TestBattenAutomaticMoves_LabelShed` and the last-`Run-Shed` rule.
It is the record of an earlier lens review, so I left it as written.

## Disputed

None.

## Deferred

None.
