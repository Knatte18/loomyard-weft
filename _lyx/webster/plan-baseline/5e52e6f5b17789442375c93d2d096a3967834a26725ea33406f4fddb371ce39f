# Card 13 — capture-teardown

**Create:**
- `plan:internal/pairteardown#Teardown.killPairCapture` -> `func (t *Teardown) killPairCapture(slug string) (bool, error)`

**Edit:**
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.EndSession`
- `internal/pairteardown#fakeSubstrates.teardown`
- `internal/pairteardown#TestTeardown_CallSequence`
- `internal/pairteardown/doc.go`

**Uses:**
- `internal/driftengine#CaptureJobName`
- `internal/proc#TerminateNamedJob`

**Intent:**
End a pair's capture at teardown on every platform (D4, Constraints).
On Linux the capture is a registered `RoleCapture` child, so the owned-process end (`runproc.EndSparing`) already ends it and the command's process group below it.
On Windows that end signals nothing, so the capture's named job must be terminated, as the loop's is.

`Teardown` gains the seam `killCapture func(slug string) (bool, error)`, wired by `New` to `killPairCapture`, which calls `proc.TerminateNamedJob(driftengine.CaptureJobName(fabricengine.WorktreePath(t.prime, slug)))`, logging what it ended; the name is computed from the pair's path, so it needs no present worktree; on Linux the call reports false and ends nothing.
`EndSession` calls it for a present and a gone task worktree alike, right after the owned-process end and before the loop seize, so an error is returned with no loop lock taken and nothing to release, as the owned-process end's own error is.
A job name lives in one machine's object namespace, so this reaches only a capture of this machine's own pair; the adopt's end of a local pair runs the same sequence.
Only the detached `capture-child` holds the named job (card 7-drift-verbs-cli); a foreground capture runs in the row's step process with no named job, in the step's `StartTied` job tree, and ends with the step.

`pairteardown`'s `doc.go` sequence step 3 names the capture's job beside the registered processes.

`fakeSubstrates.teardown` wires a recording fake for the new seam with a scriptable error, and `TestTeardown_CallSequence`'s expected call strings gain the capture kill after the owned-process end and before the seize, for a present and a gone task worktree, and a row whose scripted capture-kill error stops the sequence with no seize and no session end.

**ImpactSummary:** One new step in `EndSession` behind a seam; `lyx fabric remove`, batten's teardown and adopt reach it through the same sequence.
