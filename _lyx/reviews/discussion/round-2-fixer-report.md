# Round 2 fixer report — discussion

## Changes

- **F1** (fixed as proposed): D2's Bound gains a sentence deciding that `lyx loom resume` leaves the follow state as it is, and only a resume entering through `lyx loom start` re-arms follow; D2's Rationale-style sentence on resume is narrowed to `lyx loom start`.
  Acceptance criteria gain the row "A halted run woken by `lyx loom resume` with follow off leaves follow off and moves no viewer." directly after the `lyx loom start` resume row.
  The choice matches `internal/loomcli/resume.go`'s doc: it "never brings reed up and never switches a tmux client".
- **F2** (fixed as proposed): the ending-viewed-window Acceptance row now says the viewer moves as today, to the relevant window or to the new overview window when the overview strand is replaced, matching `internal/reedengine/replace.go`'s `selectStrandWindowLocked` call and D6.

## Disputed

None.

## Deferred

None.
