# gate: report a crashed test runner apart and rerun it once

A tier whose test runner dies on a signal with no failing test, such as a .NET `Internal CLR error` ending in exit 134, no longer counts as a failing gate.
Finalize used to read that as a code failure, free the parent lock and spend one of the fixer's two attempts on a green tree.
Now the gate runs the tier once more, and a crash that repeats stops the run with the crash named and spawns no fixer.

## What changes

- `gatecli`: a tier run that fails without timing out, reports no failing test and exits 128 plus a crash signal (SIGABRT, SIGILL, SIGTRAP, SIGBUS, SIGSEGV, SIGFPE) runs once more after the same delay as the resource-shortage rerun.
  `defaultShortageRetryDelay` becomes `defaultRerunDelay`, shared by both reruns.
  A command the shell replaced and a signal ended takes the same exit code, while a build a signal ended keeps exit 1.
  The stderr line names the exit code and the signal, and the rerun's log opens with it.
- `gateslot`: the first run's record and log are written before the rerun starts.
  `RunRecord` gains `RunnerCrash` (the signal) and `Rerun`, and the rerun's record counts its slot wait as none.
  When every failed run crashed this way, with no failing test, failed build or lint finding beside them, the call closes with `RunnerCrashLine`, which `FindRunnerCrash` reads back.
- `verifytree`: `Result` gains `RunnerCrash`, set when a failed command's log holds that line and holds no timeout line.
- `landingshed`: the Finalize and parent-align fixer paths, and the PR-Review gate, halt the row on a runner crash with a `LandingStopPrefix` reason naming the crashed runs and the way forward (`lym loom resume`), instead of spawning a fixer or mailing the agent.
  A crash writes no landing failure record.
- `websterengine`, `webstercli` and `loomshed`: the verify gate fails `Terminal` on a runner crash, on the first run or the rerun, with no fixer, no spent attempt and no report.
  Its stuck reason ends in reentry alone, since no fixer commit exists to reset.
- `gatecli` fixer detach: a red verdict whose stderr holds `RunnerCrashLine` ends with the first run's result and spawns no fixer.
- Docs: `PATTERN.md` and the package docs of `gatecli`, `gateslot`, `landingshed`, `verifytree` and `websterengine` describe the crash rerun, the new exit code and the halt.

## Design decisions

- The crash test is the exit code (128 plus a crash signal) together with no reported failing test, so a failing test keeps its verdict and goes to a fixer as before.
- One rerun per tier run, bounded like the shortage rerun.
  A test that aborts its own runner, such as a .NET stack overflow, is therefore reported as a crash, not as a failing test; the package doc says so.
- A repeated crash is a halt, not a failure, because no change to the code under test mends a runtime fault.
  The reason points the reader to the runs' logs.
- A run's exit code is its own: a runner crash exits with 128 plus the signal, an exit the callers' failing-run line and `RunnerCrashLine` both report.

## To check by hand

No integration run on a machine whose runner really crashes was made.
Check these findings, each listed as open after review:

- The verify gate of Webster (`websterengine`, `webstercli`) must not treat a runner crash as a failing test; confirm a crash in the first run and in the rerun both end `Terminal` without a fixer.
- `gatecli`: the closing line, the exit code and the fixer detach for a crash have tests only partly covering them; confirm `RunnerCrashLine` reaches `runWithFixer` and suppresses the fixer.
- `landingshed`: the parent-align and Finalize fixer ends on a crash are thinly tested; confirm the halt reason and that the lock is not freed for a fixer.
- `loomshed`: confirm the verify gate ends terminal on a crash.
- `verifytree`: confirm a log with a timeout line and a crash line reads as a timeout, and that a log with only the crash line reads as `RunnerCrash`.
