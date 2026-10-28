# loom resume: Darn-Write names prior work, defers to recorded decisions

`lym loom resume` re-spawned the Darn-Write producer with a prior-work note of `(none)`, although the branch already held the first spawn's commit.
Its gate re-prompt also ordered the writer to "fix every finding", which contradicted a parent decision recorded to leave a finding unfixed.
A resumed spawn now learns that earlier work exists and where its last gate findings are, and a recorded decision wins over a finding it forbids fixing.

Closes #889
Closes #883

## Changes

- `internal/loomshed/darnwrite.go`: `renderPriorWork` renders the note whenever the run holds an earlier Darn-Write history entry.
  The note names that entry's outcome and, when it exists, the kept gate findings file, and tells the writer that `Added after Discussion` entries in the decision record win over any finding they forbid fixing.
- `internal/loomshed/darnwrite.go`: new `DarnGateReprompt` builds the gate re-prompt line, naming the findings path and the decision record.
  `internal/shedrecipe/entries_darnwrite.go` sets it on every gate entry of the Darn-Write row, in `darnWriteEntry`.
- `internal/loomcli/darnwiring.go`: `latestDarnOutcome` now takes the findings path and passes over goto and skipped entries as well as reroute ones, since they record no spawn that did work.
- `doc.go` of `loomshed` and the tests are updated to match.

## Design decisions

- The old note was keyed on the halt reason, which is always empty on a resume: the shed's resume write clears the status error and sets the state to running before the producer is called.
  `DarnOutcome.Reason` is removed and replaced by `FindingsPath`, since the reason was dead code.
  The history entry and the kept findings file both survive the resume write.
- A recorded wait decision still lets the gate fail and the verify budget run out once more.
  Only the instruction conflict is removed; the writer may leave a finding unfixed only where an `Added after Discussion` entry says so, which only the parent, operator or arbiter appends.
- Out of scope: a resume refusal or warning that tracks a "wait for task X to land" condition, and the re-prompts of the Implementer and other rows.

## To check by hand

Not run: a real `lym loom resume` after a blocked Darn-Write halt, to confirm the re-spawn prompt carries the note and the findings path.
