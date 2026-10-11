# Webster round 1 — fixer report

## Fixed

- **F1** (LOW, consistency) — commit `60b0cd015`.
  Validated against the code: `mutateGitExclude` runs `os.MkdirAll` on the exclude dir and takes `exclude.lyx.lock` before calling the rewrite closure, and `boardCriticalSection` runs `os.MkdirAll` on the board dir and takes the board write lock before the `noWrite` return.
  Cards 6 and 7 prescribe exactly these mechanisms (the dry run computes under `mutateGitExclude`'s lock, and `Prune`/`RetireLegacy` return through `noWrite` inside the critical section), so the code path stays and the docs now say what it does.
  Took the review's first fix option:
  - `unseedGitExclude` (`internal/fabricengine/junction.go`): a dry run leaves the exclude file unwritten, computes under the exclude lock, and taking that lock still creates the info directory and lock file when absent and waits behind a concurrent writer.
  - `Unwire` (`internal/fabricengine/unwire.go`): "read-only dry run" became "dry run", plus one sentence about the exclude-file lock.
  - `Board.Prune` and `Board.RetireLegacy` (`internal/boardengine/board.go`): the same change for the board write lock and directory.
  - Checks run: `lyx loom lint-comments` is green, and `lyx gate test ./internal/fabricengine ./internal/boardengine` passes.

## Disputed

None.

## Deferred

None.
