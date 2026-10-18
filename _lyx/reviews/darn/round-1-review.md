---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/configcli/hubwide.go:reconcileDefaultLayer
    summary: A failure mid-way through the apply call leaves earlier modules' files written and uncommitted, and the error neither says so nor restores them
    fix: In reconcileDefaultLayer, snapshot the board dir's config files before the apply call (as hubreconcile.snapshotConfig does in internal/hubreconcile/walk.go) and, when the apply call or the commit-less error path returns an error with results == nil, restore them (as failRestoringConfig does), so a failed apply leaves nothing written. If restoring is not wanted, make the results == nil error branch name the possibly written files and say they are uncommitted. Correct the doc comment and the change description's claim that a failing file leaves every file unwritten, which holds only for failures the dry run sees.
    origin: handler
  - id: F2
    severity: LOW
    class: design
    location: internal/configcli/hubwide.go:reconcileDefaultLayer (final switch)
    summary: When the commit succeeds and only the push fails, the error says the files were not committed
    fix: Track the commit and the push errors apart in reconcileDefaultLayer. On a commit failure keep the current message. On a push failure alone report that the files are committed in _board but not pushed, and that the next board push carries them.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: design
    location: internal/configcli/configcli.go:runReconcile (layer refusal)
    summary: From the board dir with a layer other than default, the layer refusal fires first and its way forward (run with no --layer) is itself refused in the board dir
    fix: In runReconcile, run the board-dir refusal before the layer check, so the board-dir message that names the prime worktree is the one shown there.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: design
    location: internal/configcli/configcli_integration_test.go:299
    summary: No test pins a failed apply or a failed commit or push on the new apply path, though the change description claims all-or-nothing writes
    fix: Add a row or assertion in the existing TestConfigOverRealHub reconcile scenario (or in the configcli unit tests with a fake commit seam) that makes the apply or the commit fail and asserts the reported error and that the board files match the behaviour chosen for F1 and F2.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: consistency
    location: internal/configcli/configcli.go:416
    summary: The config help still says run lyx config reconcile removes preserved keys, which no longer holds for a --set --layer default write
    fix: In the config command help at configcli.go:416-418, say that preserved keys of the board dir's files are removed by "lyx config reconcile --layer default --apply" from the prime, and those of a worktree file by "lyx config reconcile --apply".
    origin: lens:code-affected-set
  - id: F6
    severity: LOW
    class: consistency
    location: internal/configengine/config.go:285
    summary: The missingBaseError comment says a per-worktree repo default is written only by the hub reconcile at a run start, but reconcile --layer default now writes it too
    fix: In the comment on missingBaseError, add that "lyx config reconcile --layer default --apply" from the prime writes it as well, and check that the returned message names the same routes.
    origin: lens:code-affected-set
  - id: F7
    severity: LOW
    class: design
    location: internal/configengine/config.go:warnUnknownKeys
    summary: Prune is given OpenMaps but not WholeEntries, unlike every other reader of the layers, and runs on every load even for a file already warned about
    fix: In warnUnknownKeys, skip the warning for a spec with WholeEntries set (each top-level key is a unit there), or add a test showing no spec with WholeEntries reaches this loader. Optionally skip Prune when the file's bytes were already warned about in this process, so a loader in a poll loop does not re-parse every layer on each load.
    origin: lens:code-correctness
  - id: F8
    severity: NIT
    class: design
    location: internal/configcli/configcli_integration_test.go:341
    summary: The dry run is never run from a pair, so the claim that it is unguarded is unpinned
    fix: In the refusals block of the new TestConfigOverRealHub scenario, add a dry-run row from the task pair (args --layer default, no --apply) that expects exit 0 and an unchanged board loom.yaml.
    origin: lens:code-tests
---

### [MEDIUM] Failed apply leaves files written and uncommitted without saying so

**Location:** internal/configcli/hubwide.go:reconcileDefaultLayer
**Issue:** The dry run inside the lock rules out parse and migrate failures only. If the apply call fails on a mkdir, write or legacy removal after some modules were written, `CommitWritten` returns the error and commits nothing, `results` is nil, and the report is `reconcile: <err>` with no file named. Nothing restores the files, unlike the hub walk and `setHubWide`. The change description says a failing file leaves every file unwritten, and the decision record accepts uncommitted files only if the report says so; the `results == nil` branch does not.

### [LOW] Push failure reported as a commit failure

**Location:** internal/configcli/hubwide.go:reconcileDefaultLayer (final switch)
**Issue:** After a successful commit a failed `bolt.Push` produces "did not commit and push them", but the files are committed; the remedy is a push.

### [LOW] Layer refusal in the board dir names a refused way forward

**Location:** internal/configcli/configcli.go:runReconcile
**Issue:** `--layer local` from the board dir hits the layer check first, which tells the operator to run with no `--layer`; that run is refused in the board dir.

### [LOW] Failure paths of the apply are untested

**Location:** internal/configcli/configcli_integration_test.go:299
**Issue:** Only the success path of `--apply` is pinned, so the all-or-nothing and error branches of `reconcileDefaultLayer` have no coverage.

### [LOW] Config help still points preserved keys at the bare reconcile

**Location:** internal/configcli/configcli.go:416
**Issue:** The bare reconcile never reaches the board dir's files, which is the defect this task fixes, so the sentence misdirects.

### [LOW] missingBaseError comment is stale

**Location:** internal/configengine/config.go:285
**Issue:** It names the hub reconcile at a run start as the only writer of a per-worktree repo default.

### [LOW] warnUnknownKeys differs from the layer readers in WholeEntries and per-load cost

**Location:** internal/configengine/config.go:warnUnknownKeys
**Issue:** `Layers`, `Merge` and the other readers pass `spec.WholeEntries` to the yamlengine; `Prune` takes only the open maps, so a spec with `WholeEntries` could warn on entries that are units. Seed-only modules, the likely users, load through `StackLayers` and are not affected today, so this is latent. `Prune` also re-parses each layer on every load before the dedupe lookup.

### [NIT] Dry run from a pair is not pinned

**Location:** internal/configcli/configcli_integration_test.go:341
**Issue:** A guard added to the dry-run path would fail no test.

## Rejected

- Hub lock / `PATTERN-binary-change-reconcile` concern (lens:code-pattern-fit): the entry restricts the reconcile after a binary change to `hubreconcile.Ensure`; this verb is operator-run, the board entry asks for it, and the board write lock covers the writes. The restore gap is raised as F1.
- No test of the slug-bearing strand refusal through the verb (lens:code-tests): the verb calls the shared `hubWriteRefusal`, which `configcli_test.go` already tests; a second row would duplicate it.
- No test for a missing prime anchor, a non-hub checkout or `--layer worktree` (lens:code-tests): same code path as covered rows, or unreachable misuse.
- Local layer not warning (lens:code-tests): the local layer refuses such keys at load, so no state exists to test.
