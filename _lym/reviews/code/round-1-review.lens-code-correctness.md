```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens. For each changed function and branch, one case run step by step.

## Per function, case run, result

- `runnerCrash` (`tierrun.go:181`)
  - Exit 134, `Failed` 0, no timeout: SIGABRT, correct.
  - Exit 0: `Signal(-128)` is not in the map, so "". Correct.
  - Exit 1 (a timed-out run is forced to 1): "". Correct.
  - Exit 134 with `Failed` 1 or a recorded failure: "". Correct.
  - Refusal or build failure: "". Correct.
  - Exit 130 or 143 (SIGINT, SIGTERM): not in the map, so no crash. Correct.
- `exitCodeOf` (`run.go:675`)
  - A normal exit returns its own code. Correct.
  - A non-`ExitError` returns 1. Correct.
  - A direct child killed by SIGSEGV returns 139. Correct.
  - A SIGKILL child (timeout or watchdog) returns 1. Correct.
  - One side effect: `build.go:48` shares this helper (see F1).
- `retryTier` (`tierrun.go:193`)
  - Crash, then pass: the first record is written and the rerun is marked. Correct.
  - Crash, then crash with another signal: the second signal is carried. Correct.
  - Crash, then a failing test: no crash is carried, and the first record keeps `runner_crash`. Correct.
  - Crash, then ctx cancelled during the delay: the first result is returned and `recordFirst` is not called. The returned record has `RunnerCrash` set but `crash` is empty, so it is counted as judged. Acceptable.
  - Shortage, then crash: see F2.
  - Cap: `attempt` runs at most twice, and the rerun is never re-entered. Correct.
  - Shortage marker plus signal in the first run: the marker wins and the rerun is shortage-labelled. Correct.
- `report`, `recordFirst` and `WaitStart` (`run.go:552`, `run.go:495-505`)
  - The first run's record carries the slot wait.
  - The rerun's record gets `WaitStart` = `Start`.
  - The observer's `Ended` fires once. Correct.
- `runTest` (`run.go:150-206`)
  - One tier crashed twice: `crashed` has one entry and `judged` is false, so `RunnerCrashLine` prints. Correct.
  - A crashed tier plus a failing tier: `judged` is true and no line prints. Correct.
  - A crashed tier plus a lint finding: `judged` is true and no line prints. Correct.
  - A crashed tier plus a build-blocked tier: `judged` is true and no line prints. Correct.
  - A guard run that crashes: `end.crash` is empty, so it is judged. Correct.
  - Halt, such as cancellation: it returns early with no crash line. Correct.
- `RunnerCrashLine` and `FindRunnerCrash` (`runlog.go`)
  - A crash line ending in `\r` is trimmed, and the runs are cut at the first `;`.
  - Runs contain `", "` and no `;`, so the reader and writer agree. Correct.
  - An empty `crashed` list never reaches the writer. Correct.
- `runnerCrashDetail` and `runAndRecord` (`verify.go:419`)
  - A timeout is checked first, then the crash. An unreadable log gives false. Correct.
  - A multi-call command (`a &amp;&amp; b`) stops at the first failing call, so only one line can be present. Correct.
- `verifyGate.check` (`verifygate.go:259`)
  - With named returns, `:=` reuses `err` in the same scope. Correct.
  - Crash: `recordFailure` is skipped, `verdict` returns the `LandingStopPrefix` reason, and `crashed` is true. Correct.
  - A stale landing failure record from an earlier genuine failure stays on file, which is accurate.
  - `crashed` is true even where `verdict` would first take its `ExitCode &lt; 0` branch. `RunnerCrash` is only set from a log line, so this cannot occur with a non-zero real exit. Correct.
- `Finalize.gateAndFix` (`finalize.go:718`)
  - A crash halts through `fz.halt` before `clean`, `runFixer` or any attempt write. No fixer is spawned and no `Spent` is incremented. Correct.
- `ParentAlign.Call` (`parentalign.go:201`)
  - A crash gives `stuckVerdict(..., halt=true)` before any alignment. Correct.
- Fixer `judge` and `haltReason` (`fixerrun.go:291`)
  - A crash halts with `Terminal: true` and `end` = `fixerEndCrash`. `run()` keeps `session.end`.
  - A pushed remote ref overrides it to `fixerEndPushed`, which is the right precedence.
  - `haltReason` returns `o.crash`. Correct.
  - Attempt accounting: see F3.
- `loomshed.NewVerifyGate` (`verifygate.go:141`)
  - A crash gives `Terminal: true`, and `gatefixer/rowgate.go:147` skips the fixer on a terminal result. Correct for a must-pass entry.
  - A `pass_on_cap` entry: see F4.
- `headFailureReason` (`verifygate.go:312`): see F5.
- `detachRun.endedOnRunnerCrash` (`detachfixer.go:125`)
  - It reads the whole stderr file and searches for the crash line.
  - A read error returns false, which falls through to the fixer. This is the safe default for an unreadable file.
  - The line is only printed when no judged failure exists. Correct.

## Findings

**F1: LOW, design.** `run.go:675`, `build.go:48`. `exitCodeOf` is shared with the build step, so a build killed by a crash signal now exits with 128 plus the signal instead of 1. Nothing reruns or classifies a build as a crash, so the call's first-failure exit code changes silently. The `doc.go` Exit codes paragraph only describes this for test runs.

**F2: LOW, design.** `tierrun.go:219-225`, `run.go:222-224`. `retryTier` calls `runnerCrash` on the rerun whenever it carries no shortage marker, whatever the first run was. A shortage followed by a crash is therefore labelled "a runner crash ... in its run and its rerun" and stops with no fixer. Only the rerun crashed, so the label is wrong and the "repeat crash" condition is not met. Fix: set `retried.crash` only when the first run was a crash.

**F3: LOW, design.** `fixerrun.go:134`, `fixerrun.go:291-293`. A crash after a fixer's turn halts with that turn's `Spent` charge kept. `judge` refunds the charge for an escalate (`fixerrun.go:280`) but not for a crash. Since the board entry says a crash must not spend fixer attempts, a crash after a turn still burns one of the cap. The crash halt should refund the turn the way the escalate path does.

**F4: MEDIUM, design.** `loomshed/verifygate.go:141`, `shuttleengine/wait.go:1808-1809`, `shedrecipe/entries_gate.go:114`. The loom verify gate now returns `Terminal: true` on a crash. A row's `gates` element can set `pass_on_cap: true`. For such an entry, `wait.go` rejects a terminal result with "returned a terminal result but is pass_on_cap". The crash then becomes a gate-evaluation error instead of a clean halt that names the crash. Fix: in the verify gate, don't set `Terminal` when the entry is pass-on-cap. The gate cannot see that, so the entry's contract needs to cover it. I did not check whether any shipped recipe actually sets `pass_on_cap` on a verify entry.

**F5: LOW, design.** `verifygate.go:312-313`. For the pr-review gate, a crash tells the agent to "end your turn again, which runs the gate again". It neither stops the run nor names the crash as a halt, so a persistent crash re-prompts the agent for as long as the pr-review gate allows. The board entry says a repeat crash stops the run with the crash named. I did not read `prreview.go` to see how many attempts bound this, so the severity is unconfirmed.

## Tool calls
1. Read `tierrun.go:258-327`: the `runInvocations` exit-code path, which the manifest lacked.
2. Read `shuttleengine/wait.go:1776-1855`: how `Terminal` is handled per entry kind.
3. grep for `Terminal` in `gatefixer`, `NewVerifyGate` callers and `PassOnCap` outside shuttleengine: to see whether a terminal result skips the fixer and which entries can be pass-on-cap.
4. Read `fixerrun.go:84-229`: how `fixerRun.run` maps `session.end` and charges attempts.
5. `sed` on `shedrecipe/entries_gate.go:60-125` plus a grep for `pass_on_cap`: to confirm the config key reaches the verify entry.
