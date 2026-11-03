```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I ran each changed function and branch with a concrete input and checked the callers and the added tests.

**Findings**

1. **MEDIUM** — `internal/fabricengine/originreconcile.go:640`, the new `!entry.Prunable &&` in the `registered` predicate.
   - Summary: a run-scope reconcile that does not set `PairMayBeAbsent` now errors on a prunable pair. It used to proceed.
   - Evidence: before the change a prunable entry counted as `registered`, took the `default` case, and the loop at `:663` skipped it, so no error was raised. Now `!registered && !scope.PairMayBeAbsent` returns "no pair worktree" for it.
   - Callers that pass a run scope without `PairMayBeAbsent`:
     - `internal/fabriccli/remove.go:212`
     - `internal/loomcli/landingdeps.go:662`
     - `internal/battencli/wire.go:1067`
     - `internal/loomcli/ownercheck.go:58`
     - `internal/battencli/resumereconcile.go:37`
     - `internal/fabricengine/destroy.go:1898`, which calls `originTargets` directly
   - Example: `lym fabric remove <slug>` after the task directory was deleted but not pruned would now fail at that reconcile.
   - The review text says this case "still returns today's error", which is not what the code did before.
   - The new test only covers the `PairMayBeAbsent` path and the already-unregistered no-option path, so nothing pins the prunable no-option behaviour either way.
   - The fixer should confirm that refusing here is intended, or keep the old behaviour for the no-option case.

2. **No finding** — the new stale-registration row fails with the defect in place. Without the `Prunable` check, `registered` is true and the `default` case runs. `PairAbsent` then stays false, so `entry.State != row.want || !entry.PairAbsent` trips.

3. **No finding** — the F2 change is sound.
   - `ArchiveTagName(c.slug, tip)` is identical to the old hand-formed tag.
   - The `len(tip) < ArchiveTagSHALen` guard still protects the slice.
   - `destroy.go` now calls `ArchiveTagName` with the same result as before.
   - The renamed constant covers every use in `snapshotuncommitted.go` and `replay.go`.

4. **No finding** — the F3 rows are not vacuous.
   - `PrimeRecords()` is `RecordsWorktree(h.Location)`, the prime's records worktree, which is the fork-point path `refuseEmptyOwnMarker` and `moveRecordStop` read in `add.go`.
   - With the `!opts.AdoptOnly` guard removed, each of the three rows (detached HEAD, empty marker, move record) makes Add error.

5. **No finding** — the F4 rows hold.
   - A file planted at `scratch/teardown` after `os.RemoveAll` makes `lockPairTeardown` fail with a non-`ErrPairBusy` error, so `Error` is set and `Reason` stays empty.
   - The `goneErrors` seam reaches the `refuseContinuePass` error branch.
   - `RemoveFollowUp` over a temp hub with no repository makes `PrimeName` fail, so it returns "".

Further tool calls (1):
- One `grep` over `PrimeRecords`, `RecordsWorktreePath` and `ReconcileScope{Run` callers, to confirm the F3 fork-point path and to list the callers without `PairMayBeAbsent`. My context did not hold those.
