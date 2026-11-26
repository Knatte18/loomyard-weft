```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds, I name the test row that would fail if it broke, and report the behaviours that have none.

**Per behaviour, the pinning test**
- `median`, odd, even and single counts: `TestMedian`. Pinned.
- `dirList.Set` (named, bare path, two names, duplicate name, empty name or path): `TestDirList_Set`. Pinned.
- `dirList.Set` turning a relative path absolute (`filepath.Abs`): none. Every row is already absolute.
- `dirList.String`: none. It only feeds flag help text, which a human reads, so no finding.
- `parseLoadavg`, `parseCPUPressure` (the `full` line is ignored) and `parseCPUInfo` (first model, mean MHz, no frequency): their table tests. Pinned.
- `readThrottleCount`, which sums every core and returns nil when one count cannot be parsed: none.
- `compareMeasurements`, covering order, missing sides and a zero base: `TestCompareMeasurements`. Pinned.
- `writeReport` and `readReport` round trip: `TestWriteReport_ReadsBackUnchanged`. Pinned.
- `readReport` on a missing file: `TestRun_ExitCodes/CompareMissingReport`. A malformed JSON file has no row.
- `run` and `runCompareCommand` exit codes: `TestRun_ExitCodes`. Pinned.
- `runProbeCommand` registering `-dir`: none in effect. See finding 1.
- `runProbeCommand` count validation: `ProbeZeroCount` covers `-starts` only. The other three counts go through the same loop, so this is acceptable.
- `runProbeCommand` default counts and the default report file name: none. The counts are a constant survivor at `main.go:110`. Both are small; no finding.
- `runProbe` measurement names and order, the skipped lym starts, and no leftovers after success: `TestRunProbe_MeasuresEveryPartAndLeavesNoFolders`. Pinned.
- `runProbe` cleanup on a failure path, which `probe.go:869` and the summary both claim: none. See finding 2.
- `measureGitWorkload` process count: the probe test's "at least one per step" check. Pinned.
- The traced round kept out of the timings: none. See finding 4.
- `linkProbeProgram` giving each binary a new build ID: none. See finding 3.
- `measureStarts`, `measureFileOps` and `measureNewBinaries` values: only their names are pinned, through the probe test. Timing values cannot be asserted; no finding.
- `renderComparison` different-counts warning: none. See finding 5.
- `renderReport`, `renderComparison` and `renderMachine` layout: none. This is output only a human reads, so the mutation survivors at `report.go:178-180,199,201` are no finding. The same goes for the usage-text survivors at `main.go:89,106,164`.
- The `parseOptionalFloat` constant survivor at `machine.go:189`: precision only, no finding.

**Findings**

1. **MEDIUM / design**, `tools/machinecost/cli_test.go:68` (`ProbeDirNameGivenTwice`).
   This row passes even when `-dir` is never registered. In that case `flags.Parse` fails with "flag provided but not defined" and still returns `exitUsage`.
   The mutation report confirms it: dropping `flags.Var(&dirs, "dir", ...)` at `main.go:108` survives.
   No row shows that a valid `-dir` is accepted or that a duplicate is refused for the right reason.
   Fix: assert the stderr text (`folder name "a" given twice`), or add a row where a valid `-dir` gets past parsing to the next refusal. For example, `-dir a=/x -starts 0` should end with the count error.

2. **MEDIUM / design**, `tools/machinecost/probe.go:869` and `probe_test.go:944`.
   The cleanup on failure ("on failure too") has no test. `TestRunProbe` checks for leftovers only after a successful run.
   Fix: add a tier1 row where a measurement fails after folders exist. One way is a `-dir` whose path is a regular file, so `measureNewBinaries` writes the source into the temp folder and then fails at `MkdirTemp`. Then assert that `runProbe` returns an error and that no `machinecost-*` folder is left under the run directory.

3. **LOW / design**, `tools/machinecost/newbinary.go:823`.
   Nothing pins that each linked program gets a new build ID, and so a new hash. That is the point of the new-binary measurement.
   Dropping the `-ldflags=-B=0x…` argument, or reusing one ID, would pass every test.
   Fix: a tier1 test that links two programs and asserts their contents differ.

4. **LOW / design**, `tools/machinecost/gitload.go:222`.
   The summary claims the traced round is kept out of the timed rounds, but nothing would fail if the timings included it.
   With `GitRounds: 1` the test cannot tell the cases apart.
   This is acceptable only if the claim is treated as structure rather than behaviour; flagged so the handler can decide.

5. **LOW / design**, `tools/machinecost/report.go:190-194`.
   The different-counts warning, which the summary claims, has no row. `TestRun_ExitCodes/CompareTwoReports` compares a report with itself, so the branch never runs (uncovered).
   The wording is human-read, but whether the warning fires is logic.
   Fix: add a row comparing reports whose counts differ, and assert the warning line is present. Assert only its presence, not the layout.

6. **LOW / design**, `tools/machinecost/main.go:740` (`dirList.Set`).
   Turning a relative `-dir` into an absolute path has no row; every `TestDirList_Set` row is absolute.
   Fix: add a row with a relative path and assert that `Path` is absolute and `Name` is unchanged.

7. **LOW / design**, `tools/machinecost/machine.go:506` (`readThrottleCount`).
   Summing across cores, and returning nil when one core's count cannot be parsed, has no test.
   It reads a fixed `/sys` glob, so no test can drive it as written. Splitting the parse from the glob, as the other `/proc` parsers already are, would let a tier0 row pin it.

Tool calls: none. Everything came from the inherited context (the diff pieces, the mutation and reach reports, and the pattern files).
