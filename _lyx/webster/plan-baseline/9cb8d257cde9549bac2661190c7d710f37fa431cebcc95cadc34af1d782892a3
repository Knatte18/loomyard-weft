# Card 5 — landing-hold

**Create:**
- `internal/boardengine#RunHolds`
- `internal/boardengine#holdFreeProducers`
- `internal/boardengine#BeamRunLockedError`
- `internal/boardengine#BeamRunLockedError.Error`
- `internal/boardengine#BeamRunLockedError.Is`
- `internal/boardengine_test#TestRunHolds`

**Edit:**
- `internal/boardengine#checkRunLocks`
- `internal/boardengine#checkProjectField`
- `internal/boardengine#ComputeLayers`
- `internal/boardengine_test#TestRunLock`
- `internal/boardengine_test#TestProjectRules_Membership`
- `internal/boardengine_test#TestComputeLayers`
- `internal/loomcli#runStatusTasks`
- `internal/loomcli#TestPeersVerb`
- `internal/battencli#beamForkPrecondition`
- `internal/battencli#TestBeamForkPrecondition`
- `internal/battencli#beamLanding`
- `internal/battencli#TestResolveBattenPrime`
- `internal/boardengine/doc.go`

**Intent:**
D9's landing hold, stated once: a beam's pawl run holds none from Worktree-Create through Beam-Await, a halt at any of those rows included, and holds one from Parent-Align on; every reader judges it from the producer half of the beam project entry's run status.

- `RunHolds(task)` is true for every entry whose status `IsRunStatus` accepts, except a beam project (`Kind == KindProject && Beam`) whose producer half is one of `holdFreeProducers`: `Worktree-Create`, `Beam-Up` and `Beam-Await`.
  The three names are batten's claim row and pawl's first two rows; `boardengine` imports neither recipe package, so card 16's recipe test pins `RunHolds` against `battenrecipe.NameWorktreeCreate` and the pawl recipe's rows, which `RunHolds`' doc names.
- The readers that ask whether an entry is held from change switch from `IsRunStatus` to `RunHolds`:
  membership (`checkProjectField`) accepts a task joining a beam whose run holds none; `ComputeLayers` keeps such a beam in the Project layer; members fork from the beam (`beamForkPrecondition`); every verb runs inside the beam (`beamLanding`, `ResolveBattenPrime`'s landing refusal); and `lyx loom peers` lists it as no in-flight peer (`runStatusTasks`).
  Once the run holds one, each refuses or locks as today's run hold does.
- The run lock: `checkRunLocks` keeps today's full lock for an entry `RunHolds` accepts; a beam project whose run status stands but holds none refuses only its removal (a prune, a remove or a merge, each of which leaves it absent from the result) and a change of `project`, `kind`, `recipe` or `beam`, with a `BeamRunLockedError` matching `ErrRunLocked` that names the entry, its status and `lyx batten cancel <slug>` as the way to end the run first; title, brief, body, order, labels and the other fields stay editable.
- The readers that ask whether a run is in flight keep `IsRunStatus` and see the pawl run in flight at every row: adopt's started-entry test, tender's owner-pause selection, batten's owner check and the parent dobby's driven runs.

Tests: `TestRunHolds` is the table of the predicate over a task, a grouping project and a beam project at `Worktree-Create`, `Beam-Up`, `Beam-Await` (running and blocked), `Parent-Align` and `Finalize`, and a beam with no status.
`TestRunLock` gains rows: a beam project at `running · Beam-Await` takes a title, brief, body, order and labels edit, and refuses a `project` change, a `kind` change, a `recipe` change, a `beam` change, a removal and a merge naming `lyx batten cancel <slug>`; at `running · Parent-Align` it refuses a title edit as today.
`TestProjectRules_Membership` gains a task upserted into a beam at `Beam-Await` (accepted) and at `Parent-Align` (refused, held by a run); `TestComputeLayers` a beam at `Beam-Await` in the Project layer; `TestBeamForkPrecondition` a beam at `Beam-Await` passing and one at `Parent-Align` refusing; `TestResolveBattenPrime` a verb running inside a beam at `Beam-Await`; `TestPeersVerb` a beam at `Beam-Await` not listed.
`boardengine/doc.go` states the hold and the two kinds of reader.

What it lets through, bounded: while a pawl run holds none, members join, fork and run inside the beam and its scope fields stay editable, but its identity fields and its removal stay locked, and from Parent-Align on everything locks as before.

**ImpactSummary:** one new predicate replacing `IsRunStatus` at six readers in boardengine, battencli and loomcli; the in-flight readers are untouched, and no entry other than a beam project reads differently.

**Commit:** 5: A pawl run holds its beam from change only from Parent-Align on
