```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff adds or changes with a concrete input.

**Cases run**

- `settleFresh` snapshot block, `Worktree` nil: no snapshot is taken, `attempt.Snapshot` is nil and `MayEdit` is false. Correct.
- `settleFresh` snapshot block, `Snapshot()` error: refuses with a way forward before any attempt is written, so nothing spawns and no budget is spent. Correct.
- `settleFresh` snapshot block, success: the snapshot lives in the attempt that `WriteArbiterAttempt` writes. `recordStartedArbiter` rewrites the same struct, so the snapshot is kept. Correct.
- `readEdits`, seam nil or snapshot nil (an attempt written by an older binary): returns empty edits. Correct.
- `readEdits`, current `Snapshot()` error, including a detached HEAD where `gitrepo.CurrentBranch` errors: sets a problem. Correct.
- `readEdits`, branch changed: sets a problem and lists no commits. Correct.
- `readEdits`, HEAD unchanged, `Dirty` nil at spawn and `{}` after the JSON round trip: `maps.Equal` treats the two as equal. Correct.
- `readEdits`, HEAD moved, amended or reset: `IsAncestor` returns false and the problem says the history was rewritten. Correct.
- `readEdits`, HEAD moved forward: lists the commits. If `IsAncestor` or `CommitLog` fails it sets a problem and lists no commits. Correct.
- `readEdits`, a new untracked file left uncommitted: **not caught** (finding 1).
- `readEdits`, a tracked file edited but not committed: the `Dirty` hash differs and a problem is set. Correct.
- `conclude`, resume with a problem: hands on with `precondition` and keeps the commits. The problem is not repeated in the question, but it reaches the stop report through Reason and the driver's rules line. Correct.
- `conclude`, resume with commits and no goto: hands on with `precondition`. Correct.
- `conclude`, resume with commits and a goto that the goto check or guard refuses: hands on with the commits. Correct.
- `conclude`, the agent's own hand-on with commits or a problem: the question gets both appended. Correct.
- `handOn` on timeout, a died run or a missing verdict: the strand is stopped before `readEdits`, so the read cannot race a live arbiter, and the commits are named. Correct.
- `resumePrecondition`, commits plus a goto at a blocked or awaiting stop: the existing checks still run. Correct.
- `appendTo`, nothing to append: returns the question unchanged. Correct.
- `editScope` and `Prompt` with `MayEdit` false: the no-fix fragment is filled with keys it declares none of, and the no-seam test row renders it. Correct.
- `arbiterEnvelopeFields` with `Commits` nil: prints `[]`. Correct.
- `Seam.Judge` passes `Commits` through, `drive.arbitrate` puts them in the attempt note and the re-step record, and `appendSettle` gives the same log and record as before. Correct.
- `driveReport.render` with an empty `Commits`: no ", commits" segment. Correct.
- `SettleRecord.Commits` uses `omitempty`, so a record without commits keeps its old shape. Correct.

**Findings**

1. MEDIUM, design. `internal/shedarbiter/edits.go:688` (`readEdits`) misses untracked files.
   - The `Dirty` map comes from `worktreeSnapshot` (`internal/loomcli/drivewiring.go:396`), which calls `fab.DirtyPaths`. That runs `trackedDirtyPaths`, which is `git status --porcelain -z --untracked-files=no` (`internal/fabricengine/dirtiness.go:67`).
   - So an arbiter can create a file, such as a new `.go` file or test, and leave it uncommitted, and its resume is not handed on.
   - The run's gates then build and test a fix the task branch does not hold.
   - This contradicts the fix fragment's "An uncommitted change … hands the stop on" (`contracts/stencils/loom/loom-fragment-arbiter-fix.md:26`) and `doc.go`'s "must leave every change committed".
   - Fix: also compare the code checkout's untracked paths, read through fabric's read-only `CodeWorktreeChanges` (`internal/fabricengine/diff.go:120`), between spawn and conclude, and add a test row for an untracked file.
2. LOW, design. `internal/shedarbiter/edits.go:675-685`: when HEAD has moved but `IsAncestor` or `CommitLog` fails, the hand-on lists no commits even though the arbiter did commit.
   - The parent sees "could not be read" with no hint that commits exist to keep or revert.
   - Fix: say in that problem text that HEAD moved from the spawn SHA to the current one, so the parent can find the commits.

Tool calls:
1. `internal/shedarbiter/call.go` (settle, wait and stop paths) and the goto use in `consume.go`: needed to trace the timeout and attach paths into `handOn` and `readEdits`; the context held only the diff hunks.
2. `sheddriver.HandBackQuestionCause`: needed to know whether a precondition hand-on's question carries the detail.
3. `<!--` handling in `internal/stencil` and `internal/stencilstore`: needed to check whether a fragment's header comment is stripped when it is filled into `edit_scope`.
4. `stripLeadingComment` call sites in `stencil.go`: the same check, to confirm `Fill` strips the comment.
