```yaml
lens: code-correctness
outcome: reported
```

Task: review the diff for correctness (lens code-correctness), running each changed function with a concrete input from the context I already hold.

**Per function and branch: the case I ran, and whether it is correct**

- `wire.go` `WriteSeed` seam, `WriteChildRecipe(location, slug, recipe)`: ran `recipe=motif` after a clean child seed. The record is written under the prime's run directory with the same `recipe` that `childSeedFor` used, so it is correct.
  - A write failure returns a plain error. `seedChildProducer.Call` turns it into a hard "write seed" error (`seamchild.go:104`).
  - A rerun finds the agreeing child seed (`WriteSeed` is idempotent) and writes the record again. The state converges, so this is correct.
  - A disagreeing child seed is refused before the record write, so no wrong record is ever written. Correct.
  - Not a finding: `ValidateRecipe` and `childRecipeSupported` run first, so only a supported recipe is recorded.
- `battenRunCommitPaths` with a child-recipe file on disk: the path is added after the settle records, and `removeRunRecords` commits `RunDirRel`, which covers the file, so no stray tracked file remains. Correct.
- `ReadChildRecipe` for an absent file returns `""`, nil. A present file is trimmed. Correct.
- `clearMotifRun`, run lock held: `TryAcquireWriteLock` returns `free=false`, so it returns `false, nil`. Correct.
- `clearMotifRun`, no status file: returns `false, nil`. Correct. However, this branch is the one that hides the commit-failure leftover in finding 1.
- `clearMotifRun`, status `running`: `judgeStatus` returns `done=false`, so `false, nil`. Correct.
- `clearMotifRun`, status `done` and recipe `darn` or `""`: `false, nil`. Correct.
- `clearMotifRun`, status `done` and recipe `motif`, with `removeRunRecords` returning nil: removes, commits, spawns the push and returns `true, nil`. Correct.
- `clearMotifRun`, `scratchEmptyError`: the records are gone and committed, it only warns, and `cleared=true`. Correct.
- `clearMotifRun`, spawn failure: it only warns and `cleared=true`. Correct, since the next records sync carries the removal.
- `clearMotifRun`, `removalCommitError`: **see finding 1.**
- `createSteps` guard `found && state==Done && !PairPresent`:
  - The case `cleared=true` leaves `found` and `state` stale at `found=true`, `Done`. That falls to the `else` branch and `StartRun` runs. Correct.
  - The case `cleared=false` leaves the old refusal intact, because `StartRun` refuses a done run as before. Correct.
  - `ClearMotifRun` returning an error returns the wrapped error naming the re-run. Correct.
  - A done run with `PairPresent` skips clearing and `StartRun` refuses. Correct.

**Findings**

1. MEDIUM, `design`, `internal/battencli/cleanup.go:155-160` (`removeRunRecords` at 358-371), judged against `clearMotifRun`'s handling of `removalCommitError`.
   - When the records commit fails, `removeRunRecords` has already run `os.RemoveAll` on the run directory. `clearMotifRun` returns `false, err` and the verb's way forward says "run `lym beam create` again".
   - On that rerun the status file is gone, so `clearMotifRun` returns `false, nil` at the `ErrNotExist` branch (`cleanup.go:139-141`).
   - `RunStatus` also reports `found=false`, so `ClearMotifRun` is not even called, and the pawl run starts.
   - The uncommitted deletion of the tracked old run directory (status, seed, child-recipe) is never committed or retried. `cleanup` finishes this case through `judgeRunDirectory`'s committed-status read; `clearMotifRun` does not.
   - The new run then writes into a directory whose old tracked files are deleted on disk but still in the records HEAD. Only the paths the new run commits are picked up.
   - The result is a half-finished removal that the stated way forward reports as success.
2. LOW, `consistency`, `internal/battencli/cleanup.go:116-119` (against `cleanupSlug` at 234-245).
   - `clearMotifRun` takes `slug` straight into the lock and scratch paths with no `ValidateRunID`, `IsReserved` or prime-name check, all of which `cleanupSlug` makes.
   - The slug comes from a board entry in the only caller, so I found no path to a wrong removal.
   - The removal judgment is not shared with `cleanupSlug`, so the two can drift.

**Tool calls:** none. The manifest and the diff already held every definition I needed.
