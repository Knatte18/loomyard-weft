# gatecli: fail a tier run that leaves new untracked files in the worktree

`lym gate test` now fails a tier run that leaves new untracked, non-ignored files in the worktree it ran in.
A test that writes outside its own temp directory used to leave the file behind unnoticed;
a stray `internal/hardenshed/.lock` from such a run stopped a Finalize.
The check catches that whole class, wherever the file comes from.

No hardenshed test fix is included.
The stray `.lock` did not reproduce on the base commit: every hardenshed test builds its paths under `t.TempDir()`.

## What changes

- A tier run with an invocation lists the worktree's untracked files with `git ls-files --others --exclude-standard -z` before and after it.
  A file in the second listing and not the first fails the run with exit 1; a run that already failed keeps its own code.
- The failure prints one line to stderr and the log before the summary line.
  It names at most `strayFilesListed` files, each as a quoted string, counts the rest, and gives the way forward:
  move the write under the test's temp directory, remove the files, re-run.
- The first listing is taken once per tier run, before its first attempt.
  A rerun after a shortage or a crash is held to the same listing, so a file an earlier attempt left still fails it.
- A `--copies` run is listed once around all its copies.
  The failure report lists the stray-file failure as a failed run of its own.
- A worktree git cannot list, such as a directory outside every repository, is not checked.
  A run the gate refused before running anything is not checked either.
- `internal/gatecli/doc.go` gains a "Stray files" section describing the check.

## Design decisions

- `runTier` is split: `runTierCommands` runs the commands once, and `runTierChecked` wraps it, or the copies run, with the listing check.
  `runGreenAware` takes the listing itself and passes it to every attempt through `runTierChecked`.
  `runTierCopies` calls `runTierCommands` per copy, so a copy is never checked alone.
- Bound: a file another process makes in the worktree while the tier runs fails the run as well.
  The check cannot tell it from a file a test wrote.
- The new tests are `!windows`, as they drive a real git worktree.

## To check by hand

- `internal/gatecli/copies.go`, `addFailedRuns`: a `--copies` run's stray-file failure is added to the failed runs.
  Check that a copies run with red copies and a stray file reports both, not only the red copies.
- `internal/gatecli/run.go`, `runGreenAware`, with `retryTier`: the listing is shared by every attempt.
  Check that a file the first attempt left fails the run and is not taken as already present by the rerun.
- `internal/gatecli/strayfiles_test.go` and `strayfiles_integration_test.go`: both carry `//go:build !windows`.
  Check that this, not a runtime skip, is the intended way to keep them off Windows.
- No test or gate run is recorded in this description; run `lym gate test ./internal/gatecli` before merging.

## Merge danger

**Door:** two-way
**Blast radius:** gatecli
