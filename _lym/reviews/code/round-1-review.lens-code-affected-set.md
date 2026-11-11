```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines they touch.

**Per change, the set I found, covered or not**

1. **`--upkeep` flag, and the exit record written only under `--upkeep`**
   - **Callers:** the prime dobby's `startTestMap` runs `gate testmap --max N` without `--upkeep` (`internal/beamcli/dobbydeps.go:608-631`).
     Until the dobby card lands, a dobby-started call registers through the dobby and writes no exit record.
     The record assigns the dobby side to a separate card, `testmap-upkeep-dobby`, so I don't flag it.
   - **Tests:** `internal/beamcli/dobbytestmap_unix_integration_test.go` asserts the argv and the dobby's registration, which this diff doesn't change. Covered.
   - **Constant:** `lymdirs.TestMapExitFileName` has a stale comment, see F1. Not covered.
   - **Docs:** `internal/dobbyengine/doc.go:489-497` and `internal/beamcli/doc.go:377-380` still describe the landing-count refresh. They belong to the dobby card.
   - **Readers:** `testcover.ReadExitRecord` is read only by the dobby card's side, so nothing in this diff breaks.
2. **`RoleTestMap` self-registration under `HubOwner`**
   - **Collision:** another live `RoleTestMap` record causes a refusal, but only under `--upkeep`.
     Without the flag, the dobby's own registration of its child is unchanged, so the two paths don't collide today.
   - **Role comment:** `runproc/record.go:50` says "the `lym gate testmap` refresh the prime's dobby starts". A hand `--upkeep` call now registers too, which makes that comment slightly stale. It is covered by the same F1 wording issue and I rate it NIT, so I don't file it separately.
   - **Teardown:** `orchcli/cli.go:284` ends the role through `runproc.EndRole`. It is unaffected.
3. **New `Pool.TryAcquireSpare` and `Pool.Contended`**
   - **Docs:** `gateslot/doc.go` is updated in the diff.
   - **PATTERN:** `pattern/PATTERN-gate-slot.md` has no mention of either symbol. It says "in held slots" generally, which still holds, so no finding.
4. **`takeSlot` signature**
   - **Old signature:** I found no remaining callers of the old `takeSlot(bool)`.
   - **New callers:** `testmap.go`, `testmapaudit.go` and `testmapyield.go` all pass an acquire function. Covered.
5. **`TestFunc.Parallel`**
   - **Writers:** `ParseTestFuncs` is the only writer.
   - **Docs:** `testcover/doc.go` is updated (line 18).
   - **Callers:** the one other consumer found is `gatecli`. Covered.
6. **`ExitRecord.Queued` semantics**
   - **Docs:** the field comment is updated in the diff. Covered.
7. **Waits and `PATTERN-daemon-wakeups`**
   - **Landed:** the pattern entry is added. `testmapyield.go` is added to `unboundedWaitFiles` in `cmd/lym/daemonwakeups_test.go`. The new file has no sub-second literal and no `time.NewTicker`.
   - **Poll interval:** the yield poll is one second, which meets the floor.
   - **Covered.**
8. **PATTERN declared changes and `Pattern-Departure` trailers**
   - No declared PATTERN entries other than daemon-wakeups are evident, and none of the three commits carries a trailer. Nothing to flag.

**Findings**

- **F1 — LOW, class `consistency`.** `internal/lymdirs/dirs.go:58` still says `TestMapExitFileName` is "the file under TestCoverDirName that `lym gate testmap` writes as it exits". Only a `--upkeep` call writes it now (`internal/gatecli/testmap.go:676`), so the comment should say so.
  - Fix: reword it to "that a `lym gate testmap --upkeep` pass writes as it exits".
  - Evidence: `internal/lymdirs/dirs.go:58` and `internal/gatecli/testmap.go:673-680`. I did not read the `lymdirs` line past that comment.

**Tool calls**
1. A grep for callers and doc mentions of `takeSlot`, `RoleTestMap`, the exit record and `gate testmap` outside `gatecli`. My context held only `gatecli` and `gateslot`.
2. A grep of the `pattern/` files and `testcover/doc.go` for `TryAcquire`, `Contended`, `testmap`, `upkeep` and `Parallel`. My context lacked the other pattern files.
