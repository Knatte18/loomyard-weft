# Add lyx batten cancel to end a batten run for good in any state

`lyx batten cancel <slug> [--remove]` ends a batten run from the hub's prime, whatever its state: before or after its pair exists, live, paused, blocked, failed or awaiting.
Until now no verb did this.
`lyx batten cleanup` skips non-done runs, `lyx batten pause` only adds a pause, and `lyx board set-status` leaves batten's records, so an operator removed run directories by hand, or fought a live run that kept coming back.

The verb ends the run's processes and strands, clears the board claim, removes batten's records and scratch, and posts one notice to the orch.
With `--remove` it also removes the pair and its task branches.
Without `--remove` the pair stays, with no session, no process and no teardown mark.

`lyx batten run` and `lyx batten step` now refuse a dirty source worktree before they write anything.
A dirty source or a left task branch met later at Worktree-Create halts with its way forward and never reaches a tackler.

Closes #775

## Design decisions

- **The run's state chooses the path, no flag does.**
  With the pair's task worktree present, the owner and origin reading is the run-scope origin reconcile in judge-only mode, so it moves no ref of a live pair.
  With it absent, the verb pulls Bolt and reads the owner from there, with no `claimPrimeOwner` write.
- **Refusals change nothing.**
  The verb refuses when the child's work has reached its parent (way forward: `lyx batten run`, then `lyx batten cleanup --apply`), when another machine's landing ended the entry, when the entry is owned elsewhere, on a stopped branch, on a live hub pause, on a reconcile or pull error, and when there is nothing left to cancel.
  On Windows a live registered process is refused at once with its `taskkill` line, because `runproc.End` signals nothing there.
- **Order, each step a no-op when already done.**
  A cancel marker is written first; the run's processes, batten's driver and run locks, and the tackler strand named by its in-flight marker are ended; the pair is removed if asked; records and scratch are removed under batten's locks; the claim is cleared; the slug's own pause entries are lifted; the marker is removed last.
  A cancel interrupted anywhere finishes on a re-run.
- **The cancel marker is a `shedrun` path.**
  Its writer's liveness is an exclusive lock beside it, never the pid, so a reused pid cannot make a dead cancel read alive.
  `armAt` reads it before the board pull and again after the pull and in `armSeed`, and the pre-drive check re-reads it in the pre-run seeding closures and in shed's `Run` and `Step` under the run lock.
  A start racing a cancel therefore writes no seed, no status and no claim.
- **An unreachable origin makes the cancel end-only.**
  It ends this machine's processes with a Warn, then removes and clears nothing and leaves the marker, so no start resumes the run.
  The cancel once origin answers finishes it.
- **`--remove` is forced with a snapshot.**
  `internal/pairteardown` now snapshots uncommitted paths in origin mode, and a refused snapshot removes nothing.
  `internal/fabricengine` archives the task branch's local and origin tips under `archive/<slug>/<tip>` tags on the warp origin, then deletes the branch locally and on origin whether or not its work is landed.
  A failed tag push removes no branch.
- **A start refuses a left local part of the pair** (task worktree, weft worktree or local weft branch), naming `lyx batten cancel <slug> --remove`, so a kept pair is never silently adopted with its old child records.
- **Precondition halt.**
  The pair add returns a typed dirty-source error.
  Worktree-Create maps it, and the existing-task-branch refusals, to a new shed precondition prefix, and the Go driver gets a rule row that hands back on it ahead of every tackler row.
- **Cleanup's per-slug removal is split** from its locks and its done judgement, so the cancel and `lyx batten cleanup` share one removal; cleanup still skips non-done runs.
- **Way-forward texts** that told a caller to abandon a run with `lyx board set-status` (`boardengine`'s run-lock refusal, `pairteardown`'s held-gone settle warning) now name `lyx batten cancel <slug>`.
- `PATTERN-pair-teardown` is updated for the cancel phase, which ends processes and strands first and waits for the run lock after.
- `internal/reedengine` carries main's pane-tail separator fix (a tab instead of `\x1f`, which tmux 3.4 prints literally), taken so the branch's reed tests pass; Merge-In should find it identical to main's.
- Docs: package docs of the touched packages, the orch role stencil's run-handling line, and `docs/overview.md`.

## To check by hand

- `batten-lock-seam-busy-path-untested`: the busy path of the lock seam batten's cancel takes (prime-side driver lock and run lock) has no test; check that a held lock refuses with the held lock's path and the cancel again as the way forward.
- `reached-parent-refusal-marker-loop`: check the refusal for a run whose child's work reached its parent, that it leaves no cancel marker behind to bar the `lyx batten run` its way forward names.
- Not run here: the cancel on Windows, including the creation-time read in `internal/proc/proc_windows.go` and the `runproc.List` re-list after the lock takes.
- Known gap: a tackler strand whose driver was ended between `Shuttle.Start` returning and the in-flight marker write is named by no marker, so the cancel does not stop it; it stays until its pane is closed by hand.
- Known gap: a raw git child of an ended process may leave an `index.lock` in a records repository, and the next commit there fails naming it.
- Known gap: processes an older binary started are not registered; the lock re-check catches the driver, and the rest stay for `lyx tender reap`.
