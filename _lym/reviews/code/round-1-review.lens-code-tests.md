```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests: behaviour → pinning test, or none.**

Pinned:
- `TryAcquireSpare` takes a slot only with two free, no drain and at least two slots: `TestTryAcquireSpare_TakesASlotOnlyWhileAnotherStaysFree`, one row per case.
- `Pool.Contended`: `TestContended_ReportsEveryOtherSlotHeldOrADrain`.
- Job order by summed elapsed seconds, then import path: `TestOrderJobs_TakesTheCostliestPackagesFirstThenImportPathOrder`.
- `nextRound` cap and serial/parallel split: `TestNextRound`.
- `TestFunc.Parallel` detection (top-level, only `Require` before it, renamed import): `TestScanTestFuncs`, row `ParallelOnlyAsATopLevelCallAfterNothingButRequire`.
- `--upkeep` refuses package arguments: `TestGateTestMap/Refusals/UpkeepWithAPackage`.
- `--upkeep` refuses beside a live pass: `Refusals/UpkeepBesideALivePass`.
- Only `--upkeep` writes the exit record: the `Refusals` record check, and `NamedPackageIsRebuiltWhenFreshAndLeavesTheExitRecord`.
- Side by side with separate profiles, same map as serial: `TestGateTestMapUpkeep/TestsDeclaringParallelRunSideBySideAndMapAsASerialRunDoes`.
- A round discarded on a moved linked file or HEAD, earlier rounds kept: `TestGateTestMapLinkedPackages/ALinkedFileChangedUnderARound…` and the added/removed-file rows.
- HEAD read before each job ends the pass and counts the rest queued: the same rows (`Queued: 2`) and `zmoved` in `TestGateTestMap`.
- Yield on a gate take kills the run, keeps the finished entry, releases the slot, and leaves the lock free so a hand call rebuilds whole: `TestGateTestMapPassYields`.
- Retake and end: also `TestGateTestMapPassYields`, but see F2.

Findings:

1. MEDIUM, design. `internal/gatecli/testmap.go:480`. Behaviour: a pass on a hub with fewer than two slots plans, re-stamps and ends without taking a slot, counting every job queued. Test: none. The slot-count test only covers `TryAcquireSpare` returning nil. `mapPackages` has no row with `slots: 1` that checks no slot was taken and `Queued` equals the job count. The only `slots: 1` row refuses earlier, at `NoMappedTier`. Fix: add a pass row on a one-slot hub that asserts nothing was built, the maps stand at HEAD and the exit record's `Queued` is the job count.

2. MEDIUM, design. `internal/gatecli/testmap.go:557`. Behaviour: after a retake the pass tries the build lock again without waiting, skips the package when it is held, and re-reads and re-plans the map at `H` before running another test. Test: none that fails if broken. `TestGateTestMapPassYields` only asserts exit 0 and the exit record's PID after the retake. It never checks that the hand call's map survives the pass's retake. The held-lock `continue` branch (`testmap.go:554`) has no row at all. Fix: assert in the yield test that the hand-written map is not overwritten by the retake. Add a row where a hand call holds the build lock when the pass retakes, expecting a skip and a queued count.

3. MEDIUM, design. `internal/gatecli/testmapyield.go:79`. Behaviour: on each wake of the slot wait, the pass reads HEAD and ends with `headMoved` set (the D8 acceptance criterion "slot-wait wake"). Test: none. The HEAD-moved tests hit the per-job read, not the wake read. Fix: in the yield test, commit while the pass waits for a slot, then release the gate lease and expect exit 0, the package counted queued and no map write.

4. MEDIUM, design. `internal/gatecli/testmapyield.go:74`. Behaviour: a named call waits for the package's build lock (`waitBuildLock`) while a pass holds it, and still builds whole (D9 acceptance). Test: none. `TestGateTestMapPassYields` only covers a hand call that finds the lock free. Fix: add a row where a lock is held and released a short time later by the test, and assert the named call waits, then builds. Inject the pause through `waitBase`.

5. MEDIUM, design. `internal/gatecli/testmap.go:460`. Behaviour: a call under a slot inherited through `gateslot.InheritEnv` neither takes a slot nor yields (`spare = !named && !inheritsSlot`). Test: none added in the diff. Fix: add a row that sets `gateslot.InheritEnv` to a held lease on a one-slot hub. Assert the no-package call runs to completion, takes no slot and ignores `Contended`.

6. LOW, design. `internal/gatecli/testmap.go:246`. Behaviour: a failed `--upkeep` registration (`runproc.List` or `runproc.Register` error) exits before planning. Test: only the live-pass refusal is covered. Fix: add a row with an unwritable registration directory, asserting the error text and that no map was planned.

7. LOW, design. `internal/gatecli/testmappackage.go` (`runRound`/`interruption`). Behaviour: a terminating signal kills the runs in flight and writes the rounds that finished. Test: none. Existing signal tests, if any, were not changed by this diff. Fix: add a row that cancels the call's context after round one and asserts round one's entry is on disk and the exit code is 128 plus the signal number.

8. LOW, design. `internal/gatecli/testmapaudit.go:88`. Behaviour: a yield or HEAD move ends the audit round (`errSlotYielded` returns nil, `headMoved` returns nil). Test: none. Fix: either add a row or fold this into the finding-2 yield row.

9. LOW, design. `internal/gatecli/testmap.go:117`. Behaviour: under `--upkeep` without `--max` the cap is `math.MaxInt`, with `--max` given it is honoured. Test: none. Every `--upkeep` test builds the request directly and never goes through the cobra flags. Fix: add one `RunCLIIn` row with `--upkeep` and no `--max` over more than five queued packages.

10. LOW, design. Queued-work exclusions (D8). The queue leaves out dirty-skipped, failed-at-HEAD and spawn-marked work, and `TestGateTestMap` does not assert it for the spawn-marked case. The dirty and broken packages appear in the outcome rows, but no assertion shows that a spawn-marked test alone leaves `Queued` at 0 or that a failed-at-HEAD package adds nothing to it. Fix: add one assertion each in the existing rows.

Test economy:
- No duplicates found: the new tests cover behaviours no existing test covers.
- The redundancy report lists no contained tests, so no `//testtiming:keep` finding.
- Nothing asserts static prose.

Tool calls:
1. `git diff` of the four test files. Test content was not in my context.
2. `grep` over the persisted diff to locate test headers.
3. `Read` of the persisted diff from line 60, then from line 620 to the end.
