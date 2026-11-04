# Rerun a crashed test runner once; a repeat crash halts, not a fixer

A gate tier whose test runner dies on a crash signal with no failing test judged no code. For example, `dotnet test` can abort with `Internal CLR error` and exit 134. Until now that read as an ordinary failure. Finalize freed the parent lock and sent an agent fixer to a green tree, which spent one of the fixer's attempts on nothing. A second crash would have spent the last attempt and stopped the landing.

## What changes

**`lym gate test` (`internal/gatecli`, `internal/gateslot`)**
- A tier run counts as a **runner crash** when all of these hold:
  - it exits with 128 plus a crash signal: SIGABRT, SIGILL, SIGTRAP, SIGBUS, SIGSEGV or SIGFPE;
  - it reported no failing test;
  - it did not time out.
- `exitCodeOf` now reports 128 plus the signal when the shell was replaced by the command and the command died of a crash signal. Before, it reported 1. Kills and terminations still report 1.
- A crashed tier runs once more after the same 10 s delay as a resource-shortage rerun, inside the same slots. The rerun result is the tier's result. The shortage retry and the crash retry share one function, `retryTier`.
- **Both runs are recorded.** The first run's record and log are written before the rerun, and the record carries `runner_crash: <signal>`. The rerun's record carries `rerun: true`. This now also applies to the resource-shortage rerun, whose first run was not recorded before.
- If a rerun crashes again, the closing "failed runs" line names it as a runner crash. When every failed run of the call crashed this way, with no failing test, failed build or lint finding next to it, the call closes with `gateslot.RunnerCrashLine`. `gateslot.FindRunnerCrash` reads that line back.
- The Webster card fixer (`runWithFixer`) hands no such verdict to a fixer.

**`internal/verifytree`**
- `Result.RunnerCrash` is set when a failed verify log holds the runner-crash line. `Detail` then names the crashed runs.

**Landing (`internal/landingshed`)**
- `verifyGate.check` now also returns a `crashed` flag.
- On a runner crash, Parent-Align, Finalize and the merge fixer's per-turn gate halt with `LandingStopPrefix`. The halt names the crashed runs and gives `lym loom resume` as the way forward, which runs the gate again.
- No fixer is spawned, no attempt is spent and no landing failure record is written.
- The PR-Review gate's reason for a crash tells the agent to commit nothing and end its turn again.

**Row verify gate (`internal/loomshed`)**
- A runner crash fails the row's verify gate as `Terminal`, with findings that name the crashed runs. The row's `gatefixer` wrap spawns no fixer for a Terminal result, so the run stops with the crash named.

`PATTERN-gate-fixer` gains "a runner crash gets none".

## Design decisions

- **What counts as a crash is read from the exit code, not the output.** Only crash signals count, so a run killed from outside (SIGKILL, SIGTERM) or stopped by a timeout is never read as a crash. A run that reported a failing test is never a crash either, so a real failure always reaches the fixer.
- **The landing site learns of a crash from one closing line.** The line is printed only when every failed run of the call is a crash. So a call where tier0 has failing tests and tier1 crashed still goes to the fixer for tier0's failures.
- **A repeated crash stops the run instead of going to a fixer.** That is what the entry asks for. A test that aborts its own runner, such as a .NET stack overflow, is then reported as a runner crash and halts with the crash named. This is listed as a bound in the `gatecli` package doc.
- **Webster's own verify gate (`websterengine`) is unchanged.** It already reruns a failed verify once on its own. Its fixer routing was outside this entry's landing scope.

## Tests

- `TestRetryTier` replaces `TestRetryOnShortage`. It adds rows for:
  - crash then pass;
  - crash twice, which carries the signal;
  - crash then a failing test, which carries no crash;
  - a signal exit with a failing test, which is not rerun;
  - SIGTERM, which is not a crash.
  
  Each row also checks the first-run record and the rerun mark.
- `TestFailedRunLabel` adds a crash row.
- `TestWatchdogLine` adds `FindRunnerCrash` rows.
- `TestVerifyGate_CheckMapsVerifyResult` adds a crash row that checks the halt reason and the `crashed` flag.
- `TestFinalizeVerify_RunnerCrashHaltsWithoutFixer` checks that Finalize halts naming the crash, with no fixer session, no failure record and nothing landed.
- `TestVerifyFailureFindings` adds a crash row.
