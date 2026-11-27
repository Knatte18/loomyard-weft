```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change, I traced who calls it, reads it or describes it, and checked whether the diff leaves any of them wrong.

**Per change, the affected set and whether it is covered**

1. **`memoryGuard.run` now returns `[]guardKill` instead of `[]string`.**
   - Callers: `runInvocations` (`tierrun.go:625`, updated) and `startMemoryGuard` (`memoryguard.go:410`, discards the result, so it is unaffected).
   - The tests use `runGuardScripted`, which is updated.
   - Covered.
2. **New `holders` field on `memoryGuard`.**
   - Production wires it only in `newMemoryGuard`. The report steps (`reports.go:90`) go through `newMemoryGuard`, so they get it.
   - Test literals: `memoryguard_test.go:63` and `tierrun_test.go:423` set it. `memoryguard_test.go:297` does not, but its `pressure` reports no limit found, so the nil `holders` is never called.
   - Covered.
3. **`kills` signature now takes the whole sample.**
   - Its only caller is `run`, which is updated. Covered.
4. **`memoryGuardLine` gains `slicePressure`, and a new line wording.**
   - Callers: `killTree`, plus the tests in `memoryguard_test.go` and `tierrun_test.go`, all updated.
   - Docs: the gatecli package doc is updated. `docs/overview.md:1024` and the `memory_quit_mb` description at `gateslot/template.yaml:3` describe only when the guard kills, and both are still accurate.
   - No other Go, markdown or stencil file reads the line's text.
   - Covered.
5. **`runStops.guardKills` changes type, and `slicePressureOnly` is added.**
   - Readers: `memoryLines` and `stopped`, both updated. Covered.
6. **New `tierResult.slicePressure` and `pressureStop`, `runEnd.pressureStop`, and the `retryTier` rerun.**
   - Readers: `mergeCopies`, `report`, `runTest`, `stopsCall` and `failedRunLabel`, all updated.
   - Docs: the gatecli package doc is updated.
   - Covered.
7. **New `gateslot.MemoryPressureLine` and `FindMemoryPressureStop`; `findClosingRuns` extracted.**
   - Readers of the closing line: `detachfixer.endedUnjudged` is updated.
   - `verifytree.runnerCrashDetail` (`verify.go:530-540`) is not updated, nor is anything downstream of it: `loomshed/verifygate.go:200,241,295`, `loomshed/redfirstgate.go`, `landingshed/verifygate.go:78,307,385,449`, `weaverengine/verify.go:151,168`, `weaverengine/strand.go:174` and `webstercli/verify.go:89`. **Not covered** (finding 1).
   - `redfirstdotnet.go:331` handles .NET red-first runs, where the Go memory guard does not run. Unaffected.
   - Docs: gateslot `doc.go` is updated, but it now over-claims (finding 1).
8. **New `proc.ResidentInCgroup` (Linux and Windows) and `withinCgroup`.**
   - Production caller: `newMemoryGuard`. The proc `doc.go` list is updated.
   - The rules for test-only code (PATTERN-test-only-code) and for stale names in code (PATTERN-code-conventions) are met: the function is reachable from production, and `/proc` was already written literally in the package.
   - Covered.
9. **Commit messages.**
   - The `Root-Cause:` trailer on aa1e875ec names the defect the diff removes.
   - No `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed.
   - No PATTERN changes are declared for this round.
   - Covered.

**Findings**

1. **The verify sites don't recognise the new memory pressure line.**
   - Severity: MEDIUM. Class: design.
   - Location: `internal/verifytree/verify.go:528-540` (`runnerCrashDetail`), and the gateslot doc claim at `internal/gateslot/doc.go:109-112`.
   - Summary: `MemoryPressureLine` is added so that "a site … tells … a memory limit other load filled from a failing test". Only the card gate's detached runner (`detachfixer.go:142-143`) reads it.
     - `verifytree` sets `RunnerCrash` only from `FindRunnerCrash` (`verify.go:535`).
     - So a verify whose tier ended on two slice-pressure stops reaches the loom verify (`loomshed/verifygate.go:200,241`), the landing gates (`landingshed/verifygate.go:78,307`), weaver (`weaverengine/verify.go:151`) and `lym webster verify` (`webstercli/verify.go:89`) as an ordinary failure. Each of them can then start a fixer that has nothing to fix.
     - Issue #267 names a loom verify step as one of the reported kills.
     - `summary.md` admits this under `## Bounds`. Even so, the gateslot package doc now states, for every site, a capability that most sites lack.
   - Evidence: `internal/verifytree/verify.go:438-440,535`, `internal/loomshed/verifygate.go:200`, `internal/landingshed/verifygate.go:78`, `internal/gateslot/doc.go` (diff hunk at 106-112).

**Tool calls**
- grep for `memoryGuard{` literals, `memoryGuardLine(` callers and `holders:`, plus a search of the docs, YAML and stencils for guard wording: the manifest had no list of the struct's other constructions or of the docs naming the guard.
- Read `memoryguard_test.go:285-320`, plus grep of the guard mentions in `template.yaml`, `overview.md` and pattern/: to check whether that guard literal reaches the nil `holders` field, and whether the docs went stale.
- Read `docs/overview.md:1015-1040` and grep of the rerun docs and PATTERN-gate-slot: to check the overview's paragraph on the guard and its rerun wording.
- grep of `RunnerCrash` and `runnerCrashDetail` readers outside gatecli: the manifest showed `verify.go`'s finder but not what the verify sites do with it.
- grep for the guard line text in other files, and the `memoryGuardFor` signature in `cli.go`: to rule out other readers asserting the changed line, and to confirm the factory's type is unchanged.
