# Darn-Write tells prior work on resume and lets recorded decisions win

A Darn-Write spawn that `lym loom resume` started after a halt was told `(none)` for prior work, even though the task branch held the earlier spawn's commits.
The prior-work note was keyed on the status file's halt reason, and the shed's resume write clears that error, and sets the state to running, before the row is called, so the reason was never readable.

Separately, a Darn-Write gate re-prompted with "fix every finding" even when the parent had recorded a decision telling the run not to fix those findings, so a resumed writer was ordered to work against the parent's call.

## What changes

- `loomshed.DarnOutcome` drops `Reason` and gains `FindingsPath`, the row's kept gate findings file.
  `latestDarnOutcome` in `internal/loomcli/darnwiring.go` sets it when `shedrun.GateFindingsFile` for the Darn-Write row exists.
  That file outlives a resume, a goto and a pause, and is removed only when the row's gates pass.
- `renderPriorWork` tells the note whenever the run's history holds a Darn-Write entry.
  The note names that entry's outcome, the kept findings file when there is one, and says that a decision under `Added after Discussion` in the decision record wins over a finding it forbids fixing.
- `loomshed.DarnGateReprompt` renders a one-line gate re-prompt that names the findings file and the decision record and states the same rule.
  `darnWriteEntry` sets it on every gate entry of the Darn-Write row, in place of the shuttle engine's default line.
- `internal/loomshed/doc.go` describes the note and the re-prompt.

## Design decisions

- Of the board entry's two options for the wait decision, this takes the second: the prior-work note and the re-prompt carry the recorded decisions.
  It adds no "wait for task X to land" condition and no resume refusal.
  A writer that follows such a decision still leaves the gate failing, so the gate can exhaust its budget and halt the run again; the instruction no longer contradicts the decision.
- The re-prompt applies to every Darn-Write gate entry, the discussion and description gates included, not only verify, so the rule reads the same whichever gate fails.
- Other rows keep the default re-prompt, since only the darn writer is told a decision record it follows over its findings.

## Tests

- `TestLatestDarnOutcome` gains a resumed run whose status reads running with no error and whose findings file is kept, and expects the findings path.
- `TestDarnWrite_Call` and `TestRenderPriorWork` pin the note's text with and without a kept findings file, and an empty note without an earlier entry.
- `TestDarnWriteEntry_HappyPath` asserts each gate entry's re-prompt is one line naming the findings file, the decision record and `Added after Discussion`.
