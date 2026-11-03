---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/battencli/cleanup.go:155
    summary: clearMotifRun returns a records commit failure after removeRunRecords already deleted the run directory, so the re-run its way forward names starts the pawl run and never finishes the removal commit
    fix: In clearMotifRun, when removeRunRecords returns a removalCommitError, keep the removal retryable. Either restore the run directory before returning the error, or make createSteps in internal/beamcli/create.go call ClearMotifRun whenever the status file is absent but the records HEAD still tracks the run (as judgeRunDirectory does through deps.Committed), and have clearMotifRun judge the committed status and child-recipe record in that case. Add a row to TestClearMotifRun_RemovesOnlyADoneMotifRun for the commit failure that asserts the retry finishes the commit.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/wire.go:1417
    summary: the child-recipe write in the Seed-Child WriteSeed seam has no test, and it is the only producer of the record clearMotifRun depends on, so deleting it passes every test while the feature goes dead
    fix: Add a row to the existing wire test of the WriteSeed seam in internal/battencli/wire_test.go (the one that exercises the seam's refusal paths) that runs the seam for the motif recipe and asserts shedrun.ReadChildRecipe on the prime location returns motif afterwards, and that a refused or disagreeing seed leaves no record.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/battencli/cleanup_test.go:546
    summary: the error versus warn split of clearMotifRun has no row for a records commit failure, a scratch-empty failure, a push-spawn failure or an untracked done motif run
    fix: Extend the table of TestClearMotifRun_RemovesOnlyADoneMotifRun with rows through the existing cleanupStubs. A commit-failing stub row wants an error and cleared false, a scratch-empty failure row and a spawn-failing stub row each want cleared true and a nil error, and a row whose status is not in stubs.committed wants cleared true with no commit and no spawn.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:456
    summary: the overview's beam entry and its batten cleanup paragraph at line 692 do not mention that lym beam create now clears a done motif run, which the package docs in the same diff do
    fix: In docs/overview.md add to the beam create description at line 456 and in the paragraph that follows it that create first removes a done run whose pair is gone and whose child ran motif, reporting motif_run_cleared, and that any other done run is still refused naming lym batten cleanup --apply. Add one clause to the cleanup sentence at line 692 that a done motif run is also cleared by lym beam create.
  - id: F5
    severity: LOW
    class: decision
    origin: handler
    location: internal/shedrun/childrecipe.go:41
    summary: a done motif run seeded before this change has no child-recipe record and keeps the old refusal, which is the very case the decision record reports, and the no-backfill choice is recorded only in the change description
    fix: Leave the code in place and defer this finding to the run's parent, who either records the no-backfill choice with lym loom decision add or asks for a legacy path such as clearing a done run whose committed seed or board type reads motif. The fixer must not change the code for it.
  - id: F6
    severity: NIT
    class: consistency
    origin: handler
    location: internal/battencli/cleanup.go:121
    summary: the clearMotifRun doc says it judges and removes as cleanup does, and the ReadChildRecipe doc carries history about runs seeded before the record existed, both against the self-contained comment rule
    fix: In internal/battencli/cleanup.go reword the second line of the clearMotifRun doc to state the locking on its own terms, for example that it judges and removes under the slug's run lock and status lock so a run started meanwhile is never removed. In internal/shedrun/childrecipe.go drop the sentence A run seeded before the record existed has none from the ReadChildRecipe doc.
---

### [MEDIUM] Commit failure leaves a half-finished removal that the way forward reports as success

**Location:** internal/battencli/cleanup.go:155
**Issue:** `removeRunRecords` deletes the run directory from disk before it commits, so a commit failure leaves the status file gone while the records HEAD still tracks the run. `clearMotifRun` returns `false, err` and `createSteps` tells the operator to run `lym beam create` again. On that run `RunStatus` finds no status file, `ClearMotifRun` is not called, and the pawl run starts with the old tracked status, seed and child-recipe deleted on disk and never committed as deleted. `cleanup` finishes this case through `judgeRunDirectory`'s committed read; the new path does not.

### [MEDIUM] The Seed-Child write of the child-recipe record is untested

**Location:** internal/battencli/wire.go:1417
**Issue:** Only `shedrun.WriteChildRecipe` is tested directly. No row reaches the `WriteSeed` closure, so removing line 1417, writing it before the child seed, or recording a different value than the child's seed recipe fails nothing, while `ClearMotifRun` would never clear in production.

### [LOW] Error and warn paths of clearMotifRun have no row

**Location:** internal/battencli/cleanup_test.go:546
**Issue:** `TestClearMotifRun_RemovesOnlyADoneMotifRun` covers the judgement rows and the happy removal. The branches that decide whether `cleared` is true, a commit failure, a scratch-empty failure, a push-spawn failure and a done motif run the records HEAD no longer tracks, have no row. The stubs mark every status as committed, so the untracked branch never runs.

### [LOW] docs/overview.md does not describe the new create step

**Location:** docs/overview.md:456
**Issue:** The overview describes what `lym beam create` does and what `lym batten cleanup` removes. The diff updates `beamcli/doc.go`, `shedrun/doc.go` and `battencli/cli.go` but leaves the overview as it was, so it no longer says all the ways a done run's records go.

### [LOW] Legacy done motif runs, the reported case, keep the refusal

**Location:** internal/shedrun/childrecipe.go:41
**Issue:** The decision record's incident is a motif run that was done before any record could exist. `ReadChildRecipe` returns empty for it, so `lym beam create` still refuses it. The change description names this as no backfill, but no `Added after Discussion` entry in the decision record carries the choice. The record's Expected line allows clearing a done motif run, so the narrowing to runs seeded after this change is an unrecorded decision for the parent.

### [NIT] Cross-symbol and history wording in two doc comments

**Location:** internal/battencli/cleanup.go:121
**Issue:** The `clearMotifRun` doc says it judges and removes "as cleanup does", which makes the comment depend on another symbol. The `ReadChildRecipe` doc has a sentence about runs seeded before the record existed, which is history the reader does not need to use the function.

## Rejected

- Lens code-tests, `motif_run_cleared` JSON key unasserted: the key is a struct tag on a field `TestCreate` already asserts, so a test pinning the tag would be a per-declaration test the repository's test economy rules discourage.
- Lens code-correctness, `clearMotifRun` takes the slug with no `ValidateRunID`, reserved or prime-name check: the only caller passes a slug of a board project entry that `lym beam create` has already resolved, and the code-quality rule is to trust an internal caller, so no path to a wrong removal exists.
- Handler, scratch directory created by `clearMotifRun` before the status read: the only caller reaches it with a done status found, so the directory already exists and the extra `MkdirAll` changes nothing.
- Handler, locks and judgement against `cleanup`: `clearMotifRun` takes the run lock with `TryAcquireWriteLock` and then the status lock in the same order as `cleanupSlug`, judges done through the same `judgeStatus`, and calls `removeRunRecords` with the same deps, so no fault was found.
