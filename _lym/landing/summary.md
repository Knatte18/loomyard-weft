# Raddle and motif runs start and finish without the orch

Three hand steps the orch had to take around Sketches are gone.

## A raddle run finishes through Land and Worktree-Teardown

Raddle-File ends the run's own board entry, so origin's board shows the entry done while the run still has batten's Land and Worktree-Teardown ahead.
The origin reconcile exempted a run from the ended set only once its Finalize had passed the parent-side merge, and a raddle run has no Finalize.
The next batten pass therefore refused with `origin ended ...; run lym cleanup --apply`, while `lym batten status` said to run `lym batten run`.

`finalizeEntered` in `internal/fabricengine/replay.go` now also counts a pair whose status stands at Raddle-File.
The status stays there once the run is done, so the exemption lasts until batten's own status at Land or later takes over.
That one read covers every reader of the ended set: the batten resume, the loom pre-push check, a hub reconcile, which would otherwise tear the pair down, and the records replay, which would otherwise set the run's commits aside.
A Raddle-File history row alone does not count, because a return sends the run back to Raddle-Write, where an entry ended on origin is still a real end.

## A promoted Sketch starts over its motif run

Promoting a Sketch gives the entry recipe raddle under the same slug, so the raddle run met the done record of the motif run that made the Sketch, and both dobby and batten refused the slug as already completed.

`refuseAtStart` now offers a done status to `ClearMotifRun` on a run or step, and a cleared run is judged as a fresh start, so the guard's start checks still apply.
The clear happens only for a task with recipe raddle whose done run's child recipe was motif.
Dobby's read uses the same test, `startClearsMotifRun`, so it holds no start for that run.
Every other done slug keeps its refusal.

The clear sits at batten's start rather than in the promote, since that covers every way an entry gets recipe raddle and runs on the machine that holds the run directory.
`clearMotifRun`'s records commit message now reads `batten:` instead of `beam create:`, as it has two callers.

## A refused Sketch reaches the parent

When the board refuses to make a finished motif run's entry a Sketch, `sketchFinish` still leaves the entry a draft task, and now also mails the run's parent.
The line carries the board's refusal, which names the entries that depend on the entry, and both ways forward: drop or move those dependencies and make the entry a Sketch, or give the task recipe raddle and release it.
The mail goes after the draft write, and a failed mail only warns, so it never holds the run's finish.

## Risks

No test runs a whole raddle run through Land and Worktree-Teardown.
The tests pin the reconcile's verdict at Raddle-File, the start clear and dobby's read, and the rest of that path is the one a finished motif run already takes.
The clear removes only the run directory and scratch, as `lym batten cleanup --apply` does; the task branch is left alone, matching the hand step that worked twice.
