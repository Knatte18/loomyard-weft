---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "Made the proposed pins and two more: GOAMD64= (the finding names GOAMD64 as passing through, and the proposed list leaves it out), and readMachine now asks go env GOVERSION with GOTOOLCHAIN=local from the temp folder, so the report names the toolchain that links."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
---

# Round 1 fixer report: tools/machinecost

Commits: `6e60b0c56` (F4), `c3bd1d6bb` (the other findings), `52f9a6976` (self-check), `ba1230c4f` (gate repair).
The gate found one comment wrapped mid-sentence, on `environWithoutGitVariables`; it is now one sentence per line.
A later gate listed functions that no mapped test reaches (`.lym/gate/reach.json`); no test was added for them.
By their call sites, the tier1 `TestRunProbe_*` tests run the measuring and machine-reading ones in-process.
`main` and `renderReport` are run by no test: the first only exits the process, and the second prints text for a human.
`tools/machinecost` builds and vets, and its tier0 and tier1 tests pass after the last commit.
The added tests were not run with their defects put back.
No test sends the probe a signal, so the interrupt, SIGTERM and hangup handling is untested.

## Changes

- F1: `probeFolderPrefix` and `lymStartName` are constants, and the machine line prints `trapsPath`.
- F2: `runProbe` and the measure functions take a context from `signal.NotifyContext`.
  Children start with `exec.CommandContext`, and the file-operation loop, which starts none, checks `ctx.Err()`.
- F4: `TestRun_ExitCodes` checks stderr and has a `ProbeDirIsAccepted` row.
- F5: `TestRunProbe_FailedRunLeavesNoFolders` added.
  It is not parallel, since another probe in the same process has folders in the shared run folder while it runs.
- F6, F7: the link's environment is pinned and `GOTMPDIR` is the probe's temp folder.
- F8: the git workload drops the caller's `GIT_*` variables.
- F9: `-dir` refuses a bare path and a name holding `/`.
- F10: `newbinary_test.go` added.
- F11: `sumThrottleCounts` split out, with `TestSumThrottleCounts`.
- F12: a run without `-dir` records the skip; the existing probe test runs a second probe without dirs to check it.

## Self-check

- fix-consistency 1, LOW (`_lym/landing/summary.md` stale): deferred, see below.
- fix-consistency 2, LOW (timed `git --version` still inherits `GIT_*`): fix as proposed; the timed starts drop `GIT_*`.
- fix-consistency 3, LOW (`linkProbeProgram` doc sentence misreads): fix as proposed.
- fix-affected-set 1, LOW (group names repeated): fix as proposed; the groups are constants.
- fix-affected-set 2, LOW (`probe.txt` and the report name repeated): fix as proposed.
- fix-code-correctness 1, LOW, and fix-proposal 1, MEDIUM (hangup leaves folders), merged: fix with own change.
  `SIGHUP` is caught only when it is not already ignored, so a run under `nohup` is not ended by a hangup,
  which the proposed unconditional `signal.Notify` would have done.
- fix-code-correctness 2, LOW (a killed `go build` leaves its compiler writing into the temp folder): fix with own change.
  The fork proposed none; cancellation now interrupts the `go` command instead of killing it.

## Disputed

## Deferred

- fix-consistency 1: `_lym/landing/summary.md` does not yet mention the new tests, the no-`-dir` skip,
  the signal cleanup, the pinned build environment or the dropped `GIT_*` variables.
  It is a loop-owned record under `_lym`, outside this round's write surface.

## Rulings
