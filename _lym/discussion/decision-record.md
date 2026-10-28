# Decision record — resume-fixes

## Goal

A Darn-Write spawn that `lym loom resume` starts after a halt is told that earlier work exists, where its last gate findings are, and that the decision record's `Added after Discussion` entries win over those findings;
a gate re-prompt inside a Darn-Write session no longer orders the writer to fix a finding a recorded decision forbids fixing.

## Scope

- In: the Darn-Write row's prior-work note (`internal/loomshed/darnwrite.go`, `internal/loomcli/darnwiring.go`), the Darn-Write row's gate re-prompt line (`internal/shedrecipe/entries_darnwrite.go`), their tests and `doc.go` text.
- Out: a resume refusal or warning that tracks a "wait for task X to land" condition; the loom recipe's Implementer and other rows' re-prompts; the shed engine's resume write.

## Decisions

### Decision: prior work is told whenever the row has an earlier Darn-Write history entry

Decision: `renderPriorWork` renders the note whenever the run's status holds a Darn-Write history entry, not only when a halt reason is read.
The note names the earlier outcome and, when the row's kept gate findings file exists, its path.
Rationale: the shed's resume write clears the status error and sets the state to running before the producer is called, so the halt reason the note was keyed on is always empty on a resume, which is why the re-spawn read `(none)`.
The history entry and the kept findings file both survive that write.

### Decision: the halt-reason read is dropped from `latestDarnOutcome`

Decision: `DarnOutcome` loses its `Reason` field and gains `FindingsPath`, the kept gate findings file when it exists.
Rationale: the reason is never readable at call time, so keeping the field leaves dead code and a test that pins a state the driver never reaches.

### Decision: the prior-work note and every Darn-Write gate re-prompt defer to recorded decisions

Decision: the prior-work note tells the writer to follow the decision record's `Added after Discussion` entries over any finding they forbid fixing, and each Darn-Write gate entry's re-prompt line says the same, naming the decision record.
Rationale: the board entry's second option; the default re-prompt "fix every finding" contradicted a recorded wait decision.
Bound: the line lets the writer leave a finding unfixed only where an `Added after Discussion` entry, which only the parent, operator or arbiter appends, says so; the gate still fails on that finding and the verify budget still bounds the session.

## Constraints

- `PATTERN-stencil-ownership`: the re-prompt line is a Go-rendered one-line gate text, not a prompt file.
- `PATTERN-shed-recipe-registry`: the re-prompt is set inside `darnWriteEntry`, the row's one Constructor.
- `PATTERN-test-speed`: new behavior goes in as rows of the existing table tests.

## Auto-mode assumptions

- The kept gate findings file path is the one `Env.GateFindingsFile` tells the row, which `latestDarnOutcome`'s caller can compute with `shedrun.GateFindingsFile`.
- A Darn-Write history entry exists after any Darn-Write call that reached a verdict, so its presence marks committed earlier work.

## Open risks

- A recorded wait decision still lets the gate fail and exhaust its budget once more; only the instruction conflict is removed.

## Acceptance criteria

- A Darn-Write call whose run history holds an earlier Darn-Write entry and whose status reads running with an empty error tells a prior-work note, not `(none)`.
- The note names the kept findings file when it exists and omits it when it does not.
- A Darn-Write call with no earlier Darn-Write entry tells no prior work.
- Each gate entry of the built Darn-Write table carries a re-prompt line naming the findings path and the decision record.
