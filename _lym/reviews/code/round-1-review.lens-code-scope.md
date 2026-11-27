```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, which traces each change in the diff to the board entry's goal points.

**Decisions.** The answer key is the decision record alone. It has no `Added after Discussion` entries, so the decisions are the goal points of the board entry's `## What to do` lists:
- **G1 (#267):** tell slice-wide pressure apart from one binary's growth, and wait and retry or report the run as not judged, instead of failing the tests.
- **G2 (#267):** name the other large holders in the kill message, with a way forward that fits.
- **G3 (#267):** check the memory use of `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`.
- **G4 (#270):** report a kill under a nearly full slice as an infrastructure stop, distinct from a test failure, when the largest binary holds a small share of the limit.
- **G5 (#270):** name what holds the rest of the slice.
- **G6 (#270):** give a fitting way forward: wait and re-run, or have the run wait or re-queue the killed package.

**Changes and what they trace to**
- C1, `proc.ResidentInCgroup` and `withinCgroup` with the Windows stub, the doc and the test (`internal/proc/*`): G2 and G5.
- C2, the own-share classification in `memoryGuard.kills`, the `slicePressure` flag on `memoryKill` and `guardKill`, and `pressureOwnShareDivisor` (`memoryguard.go`): G1 and G4.
- C3, `otherHolders`, `pressureHoldersShown` and the infrastructure-stop outcome in `memoryGuardLine`: G2, G5 and G6.
- C4, `tierResult.slicePressure` and `pressureStop`, `runStops.slicePressureOnly`, `stoppedOnSlicePressure` and the rerun in `retryTier` (`tierrun.go`): G1 (wait and retry) and G6.
- C5, the `failedRunLabel` pressure label, `stopsCall` and the `pressured` closing with `MemoryPressureLine` (`run.go`): G1 (report as not judged) and G4.
- C6, `gateslot.MemoryPressureLine`, `FindMemoryPressureStop`, `findClosingRuns` and the gateslot docs: forced by C5.
- C7, `detachfixer.endedUnjudged`: G1. It stops the card gate handing a not-judged run to a fixer.
- C8, the `mergeCopies` slice-pressure fold (`copies.go`): forced by C4, since the `--copies` path also goes through `retryTier`.
- C9, the `gatecli` `doc.go` updates: forced by C2 to C5 and C7.
- C10, commit 4b4cdc423 (re-wrapping comments in `run.go` and `gateslot/doc.go`): forced. It re-wraps only comments this diff added.
- The test changes in `memoryguard_test.go`, `tierrun_test.go`, `cardfixer_test.go`, `watchdogline_test.go` and `proc_linux_test.go`: forced by C1 to C7.
- No change traces to nothing.

**Decisions and the changes that build them**
- G1: C2, C4, C5 and C7. It is built only partly: the verify sites are not covered (finding 1).
- G2: C1 and C3.
- G3: no code change. The summary reports a check by reading, with no measurement (finding 2).
- G4: C2 and C5.
- G5: C1 and C3.
- G6: C3 and C4.

**Findings**

1. **BLOCKING, scope.** Location: `internal/verifytree/verify.go:440`. G1 is not built at the verify sites the board entry names, and the summary's `## Bounds` admits it.
   - The entry reports one of the kills in "one run of the loom verify step", and asks for such a run to be reported as not judged instead of failing the tests.
   - `verifytree` marks only `gateslot.FindRunnerCrash` lines as `RunnerCrash` (`verify.go:440`, `:535`). Every verify-site reader keys its "judged no code" handling off that flag: terminal, no fixer and no retry, at `loomshed/verifygate.go:200,241,295`, `landingshed/verifygate.go:78,296,307,385`, `websterengine/verifygate.go:577,611`, `webstercli/verify.go:89`, `weaverengine/verify.go:151,168` and `loomshed/redfirstgate.go:217,228`.
   - A memory pressure stop reaches all of those sites as an ordinary failure, so they still hand it to a fixer that cannot clear it. That is the failure mode #267 reports.
   - Fix: have `verifytree` recognise `gateslot.FindMemoryPressureStop` the way it recognises runner crashes, and make its readers treat it as not judged.

2. **MEDIUM, scope.** Location: `_lym/landing/summary.md:45-48`, with no code change. G3 asks to check the memory use of `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`.
   - The summary says it "was checked by reading it, not by measuring it". It calls the memory "large but bounded by the module's size" but gives no figure.
   - The killed binaries held 1.2 to 2 GB, so the claim that the test is not a contributor is untested.
   - Fix: measure the test's peak RSS, for example its `lym.test` under `-test.run`, and record the figure against `gate.memory_quit_mb` and the pressure kill floor.

Tool calls:
- `git show` of 4b4cdc423 plus a grep of `verifytree`: the manifest did not cover that commit's content or `verifytree`'s use of the crash flag.
- A grep for readers of `.RunnerCrash` across `internal`: the verify sites that act on `verifytree`'s flag were not in the manifest.
