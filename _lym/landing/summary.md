# gate: report a memory limit filled by other load as an infra stop

The memory guard killed the largest test binary whenever the memory limit over the gate call was nearly spent,
and reported the kill as that package's test failure, even when other processes held most of the limit.
A change that could not affect the package got a red gate, with a way forward that pointed at the code under test.

Now, when the gate call's own processes hold less than half of the limit's use, the kill is a slice-pressure kill:

- The kill line calls it an infrastructure stop, names up to three of the largest other holders in the limit's cgroup,
  and says to wait for that load to end and re-run.
- A tier whose only stops were such kills, and that failed nothing else, is rerun once after the rerun delay.
- When the rerun stops the same way, the run does not stop a `--stop-on-fail` call.
  If no other failure judged the code, the call closes with a `lym gate test: memory pressure stop:` line,
  and the detached runner spawns no fixer.
  The call still exits non-zero.
- The loom verify and red-first gates, the landing gates, webster, weaver and `lym webster verify`
  treat that line like a runner crash: they stop instead of handing the failure to a fixer.

Closes #267
Closes #270

## Design decisions

- The guard still kills under pressure. What changes is how the kill is judged and reported.
- "Own" memory is the resident memory of the gate call's whole process tree, not only the guarded command's,
  so sibling `--copies` copies and report-step binaries never count as other load.
  When that tree cannot be sampled, the kill is an ordinary one.
- The limit's use that no process's resident size shows, such as tmpfs or kernel memory, counts as other load,
  as does a test-started process that left the call's tree.
- A killed package is told apart by `go test`'s own `signal: killed` line,
  because a test's `chdir` moves the binary's working directory.
  A failed test, or a package failure without that line, keeps the run an ordinary failure.
  So does a mix with an over-cap kill, the tier's memory cap or the process cap.
- `verifytree.Result` has `MemoryPressure` beside `RunnerCrash`, and sites branch on `Result.Unjudged()`.
  The landing fixer end `crash` is renamed `unjudged`.
- Tier run records gain an optional `memory_pressure` field.
- `proc.ResidentInCgroup` is new; it returns `errors.ErrUnsupported` on Windows.

## To check by hand

- Test coverage of the `--copies` merge of slice-pressure stops, in `mergeCopies` (`internal/gatecli/copies.go`):
  check that the `TestMergeCopies` rows cover the rule.
- Test coverage of mixed stops and the no-failing-test condition, in `runStops.slicePressureOnly`
  and `runTierCommands` (`internal/gatecli/tierrun.go`):
  check the `TestRunStopsSlicePressureOnly` and `TestRunTierMemoryGuard` cases.
- A package-level failure beside a pressure kill must keep the run an ordinary failure
  (`internal/gatecli/tierrun.go`): check `failedBeyondKills`.
- Test coverage of the call's closing line and `--stop-on-fail` (`internal/gatecli/run.go`):
  `TestUnjudgedRunsClosingLineAndStopsCall` covers `unjudgedRuns` and `stopsCall` directly,
  and no integration test forces a pressure kill through a whole gate call.
- The verify sites built on `internal/verifytree` must not start a fixer on a pressure stop:
  check that each branches on `Result.Unjudged()`.
- The memory use of `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`
  (in the `cmd/lym` test binary), which issue #267 asks about:
  one run of that test alone peaked at 2011532 KB resident,
  above the 1024 MB pressure kill floor and below the 4096 MB per-binary cap.
  The diff does not change the test.

## Merge danger

**Door:** two-way
**Blast radius:** gate
