```yaml
lens: code-correctness
outcome: reported
```

The code-correctness lens is done: I ran each changed function and branch with concrete inputs. Two MEDIUM and one LOW finding are below.

**Per function or branch: the case I ran, and the result**
- `memoryGuard.kills`, the cap branch. A 5000 MB tree over a 4096 MB cap is killed with its old reason, `slicePressure` false. Correct.
- `memoryGuard.kills`, the pressure branch where the run fills the limit (`own*2 >= Used`). Used 15872 MB, own 8700 MB: old reason, not slice pressure. Correct.
- `memoryGuard.kills`, the pressure branch where other load fills the limit. Used 15872 MB, own 2700 MB: slice-pressure kill, and the line is as pinned. Correct for one guarded tree.
- `memoryGuard.kills` under `--copies`, a tier run beside the report steps, and reparented daemons. Wrong: the run's own load is counted as other load (finding 1).
- `memoryGuard.kills`, headroom below zero. It is clamped to 0, and the branch is reached only with Used above 0.9·Max. Correct.
- `memoryGuard.otherHolders`, the error path. A holders error gives the "could not be read" clause and a warning in the log. Correct.
- `memoryGuard.otherHolders`, an empty or zero-RSS list, and only own pids. It gives the "no other process" clause. Correct.
- `memoryGuard.otherHolders`, five holders and an empty Path. It sorts largest first, names three, and uses the fallback name. Correct.
- `memoryGuardLine` with `slicePressure` true and false. The outcome clause switches. Correct.
- `memoryGuard.run`. A kill carries `slicePressure` into `guardKill`; the ErrUnsupported and sample-error paths are unchanged. Correct.
- `runStops.memoryLines`. No kills and no memory cap give nil; with a cap, the guard lines come first. Correct.
- `runStops.slicePressureOnly`. No kills is false; a memory cap or process cap is false; one non-pressure kill is false; all pressure kills is true. Correct.
- `runTierCommands`, `slicePressure`. A pressure kill in package A plus a build or vet failure (a package-level failure) in package B in the same command: `record.Failures` drops the package failure, so `slicePressure` is true. Wrong (finding 2).
- `stoppedOnSlicePressure`. A timed-out, refused or build-failed run is false. Correct.
- `retryTier`: pressured then pass, pressured twice, pressured then an ordinary failure, shortage then pressured, ctx cancelled in the delay. The rerun and `pressureStop` are as documented. Correct.
- `retryTier`: crash then pressured, or pressured then crash. Neither gets a flag, so the call counts as judged; this matches the shortage-then-crash precedent. Correct as documented.
- `retryTier`: the first run's record for a pressure stop. Nothing on the record marks it, unlike `RunnerCrash` (finding 3).
- `mergeCopies`. One pressured copy and the rest passing is pressure; one pressured copy and one failing otherwise is not; a refused copy goes the refusal path. Correct, given each copy's own flag; see finding 1 for the flag itself.
- `runTest` closing: crash and pressure mixed with no judged failure. Only the pressure line prints, and it names only the pressured runs. Consistent with the line's text.
- `runTest` closing: pressure plus a failed lint. `judged` is set, so no pressure line prints. Correct.
- `stopsCall`, `failedRunLabel`, `report`. `pressureStop` is threaded through, and the cases cannot overlap. Correct.
- `detachRun.endedUnjudged`. A pressure line in stderr returns true; an unreadable file returns false. Correct.
- `findClosingRuns`, `FindMemoryPressureStop`, `MemoryPressureLine`. A CR-terminated line works; the runs stop at the first semicolon. Correct.
- `ResidentInCgroup`, `withinCgroup`. "/" matches everything; "/a" matches "/a/b" but not "/ab"; a process that has gone or has a short stat is skipped. The path form is the same `0::` path that `MemoryLimit.Cgroup` comes from. Correct.

**Findings**

1. MEDIUM · design · `internal/gatecli/memoryguard.go:264-275`, in `memoryGuard.kills` (with `otherHolders` at 280-306)
   - Summary: the own share sums only the guard's root tree. Memory this same call holds outside that tree counts as "other load", so the kill can be called an infrastructure stop wrongly, and the line names the call's own processes as the other holders. Each case below sits outside the guard's root tree:
     - **`--copies`.** Each copy's guard is rooted at that copy's own shell (`tierrun.go:625`, `copies.go:94`). Four copies at 3.5 GB each in a 16 GB slice give own = 3.5 GB, under half of Used. Every copy becomes slice pressure, `mergeCopies` marks the tier as pressure, and on a repeat it is `pressureStop`: not judged and no fixer. The sibling copies' `.test` binaries are then named as other holders.
     - **Report steps.** These test binaries run beside the last tier as children of the lym verb, not of the tier's shell (`reports.go:99`, `run.go:199-200`).
     - **Reparented daemons.** Processes a test starts that double-fork, such as tmux servers, leave the tree.
     - **tmpfs and shmem.** These are charged to `Used`, since `fileCacheBytes` subtracts only `active_file` and `inactive_file` (`memorylimit.go:78-99`), but no RSS shows them.

     The line then states "no change to the code under test frees it", which is false in each of these cases.
   - Evidence: `memoryguard.go:265-269`, `tierrun.go:625`, `copies.go:88-96`, `reports.go:89-99`, `proc/memorylimit.go:19,78`.

2. MEDIUM · design · `internal/gatecli/tierrun.go:234`, in `runTierCommands`
   - Summary: `slicePressure` checks only `len(record.Failures) == 0`, and `testFailures` drops every package-level failure. A run that failed for a cause in the code under test can therefore be classed as judging no code:
     - a build or vet failure in another package;
     - a TestMain `os.Exit`;
     - an init panic.

     If both runs end that way, `pressureStop` follows:
     - the label says "not a test failure";
     - `--stop-on-fail` is skipped;
     - the call closes with `MemoryPressureLine`;
     - no fixer is spawned.
   - The check should take into account the package failures of packages whose binary the guard did not kill (`result.failures` entries with `test == ""`).
   - Evidence: `tierrun.go:229-234`, `tierrun.go:399-404`, `counters.go:343-348`, `run.go:221-227,333`, `detachfixer.go:139-145`.

3. LOW · design · `internal/gatecli/tierrun.go:498-500`, in `retryTier`
   - Summary: a first run that was a pressure stop gets no marker on its run record. A crash gets `record.RunnerCrash` at line 496. The rerun's record carries only `Rerun`, and `pressureStop` lives only in `tierResult` and `runEnd`. So the records on file cannot tell a pressure stop from an ordinary guard kill, and the doc's "first run's record carries `runner_crash`" has no counterpart for pressure.
   - Evidence: `tierrun.go:494-497,511-519`, `run.go:771`, `doc.go` hunk at lines 132-133.

Tool calls:
1. grep for `runTierChecked`, the counter's package failures, and the `memoryGuard{` literals in tests. The context lacked the copies dispatch and how package failures are recorded.
2. Read `counters.go:300-354`, to confirm that package-level and build failures carry `test == ""` and so are dropped from `record.Failures`.
