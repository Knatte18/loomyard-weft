# Card 16 — batten-cleanup

**Create:**
- `internal/battencli/cleanup.go`
- `plan:internal/battencli#CleanupReport` -> `type CleanupReport struct`
- `plan:internal/battencli#CleanupSkip` -> `type CleanupSkip struct`
- `plan:internal/battencli#cleanupDeps` -> `type cleanupDeps struct`
- `plan:internal/battencli#fabricCleanupDeps` -> `func fabricCleanupDeps(location *lyxcwd.Location) cleanupDeps`
- `plan:internal/battencli#Cleanup` -> `func Cleanup(location *lyxcwd.Location, apply bool) (CleanupReport, error)`
- `plan:internal/battencli#cleanup` -> `func cleanup(location *lyxcwd.Location, apply bool, deps cleanupDeps) (CleanupReport, error)`
- `plan:internal/battencli#emptyScratch` -> `func emptyScratch(dir string, apply bool) error`
- `plan:internal/battencli#orphanScratchAge` -> `const orphanScratchAge = 24 * time.Hour`
- `plan:internal/battencli#battenCLI.cleanupCmd` -> `func (c *battenCLI) cleanupCmd() *cobra.Command`
- `internal/battencli/cleanup_test.go`
- `internal/battencli/cleanup_integration_test.go`

**Edit:**
- `internal/battencli#Command`
- `internal/battencli#battenCLI.resolvePersistentPreRun`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedrun#RunsRoot`
- `plan:internal/shedrun#ScratchRoot`
- `plan:internal/shedrun#RunDirRel`
- `plan:internal/fsx#NewestModTime`
- `internal/shedrun#RunDir`
- `internal/shedrun#ScratchDir`
- `internal/shedrun#StatusRel`
- `internal/shedrun#RunLock`
- `internal/shedrun#StatusLock`
- `internal/battencli#RefuseUnlessPrime`
- `internal/fabricengine#CommittedAnchoredFile`
- `internal/fabricengine#CommitAnchoredPaths`
- `internal/fabricengine#PushAnchored`
- `internal/lock#TryAcquireWriteLock`
- `internal/lock#AcquireWriteLock`
- `internal/shedengine#Status`

**Intent:**
Add `lyx batten cleanup [--apply]`, prime only, with `lyx board prune`'s dry-run shape: without `--apply` it lists what `--apply` removes and changes nothing (D9).
`Cleanup(location, apply)` is the function the verb and `lyx cleanup` both call: it applies `RefuseUnlessPrime`, then runs `cleanup` with `fabricCleanupDeps(location)`.
Every refusal and skip below lives in `cleanup`, so neither caller can remove more than it decides.

`cleanupDeps` carries the fabric calls as function values, as `commitStatusDeps` does, so a tier-1 test drives every branch with stubs: `Committed`, a read of an anchor-relative file at the records HEAD (`fabricengine.CommittedAnchoredFile`); `Commit`, a records commit of anchor-relative paths with a message (`fabricengine.CommitAnchoredPaths` with a throwaway `fabricengine.NewMutations("")` and `fabricengine.EnvSyncOptions()`, records only); `Push` (`fabricengine.PushAnchored` with `fabricengine.StatusPushLockWait`); and `Now`.

The slugs are the sorted union of the directory names under `shedrun.RunsRoot` and under `shedrun.ScratchRoot`; a name `shedrun.ValidateRunID` refuses or `shedrun.IsReserved` reserves is skipped and reported, and a not-exist on either root is an empty set.
For each slug, in order:

1. The run lock (`shedrun.RunLock`) is taken with `lock.TryAcquireWriteLock`; a held lock skips the slug with reason "run lock held".
   In a dry run no lock file is ever created: a lock whose file is absent counts as free and is not taken; under `--apply` the scratch directory is created first when absent, as `checkRunStatus` does for the status lock.
2. Still holding it, the status lock (`shedrun.StatusLock`) is taken with `lock.AcquireWriteLock`, before the status is read, and held until the slug's commit is made, so a `lyx batten run <slug>` started meanwhile blocks in `checkRunStatus` and can neither re-seed `status.json` before the deletion commit nor lock a new status-lock inode.
   The status is read with a plain file read and a strict JSON decode while the lock is held, never through `state.ReadJSONStrict`, which takes the same lock and would deadlock.
3. The records run directory (`shedrun.RunDir`) on disk decides the path:
   - present, with a status that decodes to `done`: a candidate; under `--apply` the directory is removed with plain `os.RemoveAll` and its deletion committed with `Commit([shedrun.RunDirRel], "batten cleanup: remove the run directory of <slug>")`.
     Running, paused, blocked, awaiting and failed are skipped and reported with their state, and an undecodable status is skipped and reported with the decode error.
   - absent (a not-exist error only) but tracked: when `Committed(shedrun.StatusRel)` finds a status at the records HEAD that decodes to `done`, the slug is a candidate whose removal is already on disk, and `--apply` only makes the commit, so an apply interrupted between removal and commit is finished by the next run.
     A committed status that is not done, or does not decode, skips the slug with that reason, leaves the deletion uncommitted, and its scratch is never emptied by the orphan rule.
   - absent on disk and absent at the records HEAD (`Committed` finds neither `shedrun.StatusRel` nor `shedrun.SeedRel`): an orphan, see below.
   - any other stat error skips the slug and is reported.
4. A candidate's scratch directory (`shedrun.ScratchDir`) is emptied by `emptyScratch` after its commit, still under both locks, then both locks are released.

`emptyScratch` removes every file under the directory whose name does not end in `.lock` (`run.lock`, `status.json.lock`, `last-commit.lock`, `steps/loop.lock` and any other), then every directory left empty below it; lock files, their directories and the scratch directory itself stay, so no lock is deleted while held or recreated beside a held one.
With `apply` false it removes nothing.

An orphan is a scratch directory whose records run directory is absent on disk and at the records HEAD, whose run lock step 1 took, and whose `fsx.NewestModTime` is older than `orphanScratchAge`; it is emptied by `emptyScratch` the same way, and a fresher one is kept and reported.

After the last slug, `Push` runs once when `--apply` made at least one commit; a push error is reported as `push_error`, the envelope stays ok, and the commit stays for the next records sync, as the status-commit seam's push-warns disposition already does.
A commit error stops the run with that error after releasing the slug's locks; the next run finishes the slug through the tracked-but-deleted path.
Bound: it deletes only a done slug's records run directory and the scratch contents beside its lock files, or an aged orphan's scratch contents; it never touches a pair, a branch, a lock file or another slug, and it pushes the records side only.

The verb: `cleanupCmd` builds `cleanup` with `Use: "cleanup"`, a lower-case `Short`, audience operator, `cobra.NoArgs` and `--apply`; its `RunE` checks `clihelp.ShouldAbort`, resolves the cwd through `lyxcwd`, calls `Cleanup`, and prints `{"applied", "removed", "emptied", "skipped", "push_error"}` through `output.Ok`, or the error through `output.Err`.
`Command` adds it beside the shed verbs, and its `Long` says every verb runs from the prime and what `cleanup` does; the package comment in `cli.go` names the verb.
`resolvePersistentPreRun` returns early for `cleanup` as it does for the bare group, since the verb takes no run-id and arms no shed spec.

Tests:
`cleanup_test.go`, untagged, builds the location with `locationkit` over `t.TempDir()` and drives `cleanup` with stub deps.
One table covers: a done slug removed, committed with its `RunDirRel` and scratch emptied beside surviving lock files; each non-done state, a held run lock (the test holds it), an undecodable status and a reserved name skipped with reasons; a tracked-but-deleted done slug committed with nothing left to remove; a tracked-but-deleted not-done slug skipped with no commit and its scratch kept; an aged orphan emptied and a fresh one kept; a run directory whose stat fails with a permission error skipped (the row skips on Windows and as root); a dry run that changes nothing and lists exactly what the apply row removes; and a push failure reported with the commit made.
A second test's `Commit` stub asserts the status lock is held (`TryAcquireWriteLock` fails) and that the lock file is `os.SameFile` as before the run; after `cleanup` returns, a `battenCLI` armed for the slug reports `checkRunStatus` found false with no error, so `lyx batten run <slug>` no longer refuses on a done status.
`cleanup_integration_test.go` (`//go:build integration`) builds a prime with `hubforge`, commits and pushes a done run directory through `fabricengine`, runs `RunCLIIn(prime, out, "cleanup")` and asserts nothing changed and the dry run's lists, then `"cleanup", "--apply"` and asserts the run directory is gone on disk, absent at the records HEAD and the deletion pushed, and that the slug's lock files survive.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/battencli` row grows by the new top-level tests.
`docs/overview.md`'s batten bullet names `lyx batten cleanup [--apply]`.

**ImpactSummary:** batten gains a cleanup verb and an exported Cleanup; the pre-run skips arming for it, and no existing verb's behavior changes.
