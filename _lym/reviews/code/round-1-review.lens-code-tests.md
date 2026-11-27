```yaml
lens: code-tests
outcome: reported
```

My lens was the tests of the diff: for each behaviour the diff adds or changes, which test row would fail if it broke.

**Per behaviour, the test that pins it:**
1. `memoryGuard.kills` sorts a pressure kill into slice-pressure or ordinary by the run's share: pinned by the `TestMemoryGuard` rows "a short memory limit other load fills…" and "a short memory limit the run fills…". The boundary `own*2 == Used` is not pinned. That is acceptable.
2. `otherHolders` leaves out the run's own pids and holders at 0 RSS, sorts largest first, caps at three and names each by its executable's base name: pinned by the `TestMemoryGuard` row with the literal line, which uses `otherLoad`.
3. `otherHolders` when the cgroup cannot be read: pinned by the `TestMemoryGuard` row "…whose cgroup cannot be read…".
4. `otherHolders` when no other holder is left: pinned by `TestRunTierMemoryGuard`'s `pressureLine` ("no other process under it…").
5. `otherHolders` names a holder whose `Path` is empty as "a process whose executable could not be read": no row.
6. `memoryGuardLine` with `slicePressure` set gives the infrastructure-stop outcome: pinned by the literal in `TestMemoryGuard`.
7. `guardKill.slicePressure` is carried through `run`: pinned by `wantPressureKills` in `TestMemoryGuard`.
8. `runStops.slicePressureOnly` is false for mixed stops (a pressure kill beside an over-cap kill, the memory cap or the process cap): no row.
9. `tierResult.slicePressure` needs `len(record.Failures) == 0`: no row. A pressure kill beside a failing test is not exercised.
10. `runTierCommands` wires `slicePressure` into its result: pinned by `TestRunTierMemoryGuard`/`ASlicePressureKillMarksTheRunForARerun`.
11. `stoppedOnSlicePressure` and `retryTier`'s pressure rerun, announcement and `pressureStop`: pinned by the new `TestRetryTier` rows (`SlicePressureThenPassEndsPassed`, `…OrdinaryFailure…`, `…Twice…`, `ShortageThenSlicePressure…`, `TimedOut…`).
12. `failedRunLabel` for `pressureStop`: pinned by `TestFailedRunLabel`/`PressureStopTwice`.
13. `stopsCall` ignores `pressureStop`: no row. The crash case has an integration row at `internal/gatecli/test_integration_test.go:1832`; pressure has none.
14. `runTest` closes with `MemoryPressureLine` when no failure judged the code, and leaves out `RunnerCrashLine` when a pressure stop is present: no row. The crash path has rows at `internal/gatecli/test_integration_test.go:1613-1685`; pressure has none.
15. `mergeCopies` sets `slicePressure`: no row. `TestMergeCopies` (`internal/gatecli/copies_test.go:197`) asserts only launches, exit and copies.
16. `runEnd.pressureStop` is passed through `execution.report`: no row, for the same reason as 14.
17. `detachRun.endedUnjudged` on the pressure line: pinned by `TestRunWithFixer`/`MemoryPressureLineSpawnsNone`.
18. `MemoryPressureLine` and `FindMemoryPressureStop`, and the two finders not matching each other's line: pinned by the `TestWatchdogLine` `closingTests` rows.
19. `proc.ResidentInCgroup` lists its own cgroup and the parent cgroup, and returns nothing for an empty child: pinned by `TestProcessTreeReads`.
20. `withinCgroup` with root `/`, and a sibling whose name shares the prefix (`/a` against `/ab`): no row.

**Findings:**

- **MEDIUM | design | `internal/gatecli/run.go:283-289`.** The call's pressure closing line, its precedence over `RunnerCrashLine` and the `judged` gate have no test. A regression that printed `RunnerCrashLine` beside a pressure stop, or dropped the pressure line, would pass every test. This line is what `detachfixer` reads to skip the fixer. Evidence: the crash-only rows at `internal/gatecli/test_integration_test.go:1613-1685`; no test names `MemoryPressureLine` outside `cardfixer_test.go:360` and `watchdogline_test.go:77`.
- **MEDIUM | design | `internal/gatecli/run.go:333` (`stopsCall`).** That `--stop-on-fail` does not stop at a pressure stop is uncovered. The crash case has `internal/gatecli/test_integration_test.go:1832`; no test sets `runEnd.pressureStop` there.
- **MEDIUM | design | `internal/gatecli/copies.go:110-117` (`mergeCopies`).** The new rule for a slice-pressure stop across `--copies` has no row. `TestMergeCopies` (`internal/gatecli/copies_test.go:197-215`) never sets or asserts `slicePressure`. It misses one copy under pressure with another failed (want false), all copies under pressure (want true), and one under pressure with the rest passed (want true).
- **MEDIUM | design | `internal/gatecli/tierrun.go:234` and `:338-349`.** Two exclusions in the rerun classification are untested: `slicePressureOnly`'s exclusion of mixed stops, and the `len(record.Failures) == 0` condition. `TestRunTierMemoryGuard` (`internal/gatecli/tierrun_test.go:398`) has only a pure-pressure row and a pure over-cap row. A tier with a failing test plus a pressure kill, or a pressure kill plus the memory cap or process cap, could be rerun and labelled unjudged without any test failing.
- **LOW | design | `internal/gatecli/memoryguard.go:299-302` (`otherHolders`).** The fallback name for a holder whose executable cannot be read has no row. Every holder in `otherLoad` (`internal/gatecli/memoryguard_test.go:462-471`) carries a `Path`.
- **LOW | design | `internal/proc/proc_linux.go:200-202` (`withinCgroup`).** Neither the `/` root branch nor the sibling-prefix exclusion is pinned. `TestProcessTreeReads` (`internal/proc/proc_linux_test.go:187-197`) checks only the own cgroup, its parent and a missing child, so a check that dropped the `+"/"` would pass.

Tool calls:
1. grep of the `_test.go` files for `mergeCopies`, `stopsCall`, `pressureStop`, `MemoryPressureLine` and `slicePressure`: my context had no index of the tests outside the diff.
2. grep for the runner crash closing-line tests, plus a read of `TestMergeCopies`: to find the crash rows the pressure path would mirror, and the existing merge assertions.
3. Read of `internal/gatecli/test_integration_test.go:1600-1690`: to confirm the crash rows cover the closing line only for crashes.
4. grep for the `--stop-on-fail` crash test: to confirm the crash case has a stop-on-fail row and pressure has none.
