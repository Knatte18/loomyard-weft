# Parent-only refusals hold, not retry; halts name real files and ways

Three halt-message defects in the darn run are fixed.

## What changes

- **Red-first holds instead of retrying.**
  A bug-labelled entry whose diff adds no test, or whose check cannot judge the tests, used to fail the gate as an ordinary failure.
  Each repeat spent a verify attempt, so three attempts went in two minutes while the parent's `No test seam` decision was still committing.
  Now `redFirstGate` in `internal/loomshed/redfirstgate.go` answers pending on those two exits.
  The run holds without spending an attempt until HEAD's tree changes or the parent's record lands.
  While HEAD's tree is the one the refusal judged, the gate answers pending without running the command again.
  Other failures, a timeout or a runner crash included, still fail as before.
- **One hold line, offered until delivered.**
  The hold's pending answer sends the writer one line telling it the way forward.
  `GateResult` in `internal/shuttleengine/gate.go` gains a `Delivered` flag that the loop sets once the send lands.
  A send that failed is offered again at the next evaluation, and a delivered one is not repeated.
- **Stencil build-version refusal keeps its own way forward.**
  `preflightTableError` in `internal/loomcli/darnwiring.go` no longer appends the "fix the config or model spec" clause to a `stencilstore.NewerBoardCopyError`, since no config is at fault.
  Any other table-build error keeps the clause.
- **A halt never names a removed gate-findings file.**
  The producers no longer remove the file when a later call's gate passes, so the path in a Stuck reason survives.
  `clearGateFindings` and its call in `internal/shedadapters/burlerreview.go` are deleted.
  `latestDarnOutcome` names the findings path only for a Stuck entry, and only when the file was not last written before that Stuck's spawn began.
  A Stuck that wrote no findings of its own therefore names none.

The darn recipe comment, the darn write stencil and `docs/overview.md` describe the new red-first behaviour.

Closes #173
Closes #174
Closes #175

## Design decisions

- **Red-first now has two closures.**
  `NewRedFirstGate` returns a `RedFirstGate` with `Gate` and `Final` sharing one attempt count.
  `Gate` answers pending on the held refusal, so its `GateEntry` is `MayHold`, which `GateResult.Pending` requires outside `PassOnCap` entries.
  `Final` runs in a final evaluation and fails on that same refusal with findings naming the same way forward, so a held run that ends does not pass silently.
- **`verifytree.HeadTree` is exported** so the gate can tell whether the tree a verify judged is still HEAD's.
- **Issue #174's cause is not reproduced.**
  The decision record named the orphan scratch sweep in `internal/battencli/cleanup.go` as a suspect, and that code is untouched.
  The removal this change deletes was the producer's own clear on a passing call.
  The reviewer should judge whether that accounts for the reported missing file.

## To check by hand

None of these was run end to end against a live run.

- Confirm that the producer's clear on a passing call, not the scratch sweep in `internal/battencli/cleanup.go`, removed the file in issue #174.
- Failed hold-line send is never retried by the loop itself (`internal/loomshed/redfirstgate.go`, `internal/shuttleengine/wait.go`): a failed send is offered again only at the next pending evaluation.
- The send-failed title is a literal in the hold path (`internal/loomshed/redfirstgate.go`, `redFirstHoldSendFailed`); check it matches the way the neighbouring gate messages are named.
- A Stuck from a non-gate cause that follows a gate Stuck can still name the kept findings file when it was written after that spawn began (`latestDarnOutcome` in `internal/loomcli/darnwiring.go`).

## Merge danger

**Door:** two-way
**Blast radius:** loomshed
