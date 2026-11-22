```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I ran each function and branch the diff adds or changes against concrete inputs.

**Functions and branches, with the case run and the verdict**

- `reportStepsOf`: reports `nil`, which gives all three steps false. Reports `[reach, mutation]`, which gives reach and mutation. Correct.
- `reportSteps.any`: the zero value gives false; one step set gives true. Correct.
- `lastTierStart`:
  - An empty plan gives -1.
  - A single run gives 0.
  - Tiers `[0, 1, 2, 2]` give 2, the first run of the highest tier.
  - Tiers `[2, 0]` give 0.
  - Correct.
- `newJudgedReports`: it reads every field of `e` on the caller's goroutine. `memoryGuarding` and `commands` are set before the plan loop starts the chain. Correct.
- `judgedReports.judge`:
  - With the guard off, `startMemoryGuard` returns no-ops. With it on, `foreign` is `tierCommands.holds`. A nil `tierCommands` gives `holds` false. Correct.
  - Each step gets the chain's context and `guardKilled` before it judges. Correct.
  - Red or not in a hub returns early with the note. Correct.
- `judgedReports.write`:
  - The guard lines flush first, then each step's held lines, then its report line. `stderr` is set back before `writeReport`. Correct.
  - Redundancy with `kept` set prints the kept line and writes nothing. Correct.
- `flushLines`: a nil writer prints nothing and keeps the buffer; a write error logs at Warn. Correct.
- `startReports`, `wait` and `stop`:
  - `wait` blocks on `done` and then cancels.
  - `stop` after `wait` returns at once, since `done` is closed.
  - Calling `stop` twice is safe: `cancel` is idempotent and a closed channel can be received from again.
  - Correct.
- `runningCommands.add`, `remove` and `holds`: a nil receiver does nothing on any of them. `remove(0)` after a failed start does nothing. The mutex guards the map. Correct.
- `runInvocations` registration: `started` runs synchronously inside `start` on the same goroutine as `runTiedChild` (`child.go:58`). So the closure's write to `commandPID` and the later `remove` do not race. `remove` runs after `killLeftovers`. Correct.
- `withoutForeign`:
  - A binary whose parent is not sampled is kept.
  - A chain from the binary to `go test` to a shell held in `foreign` is dropped.
  - A cycle in the parents ends through `seen`.
  - A foreign set that names no process keeps every tree.
  - Correct.
- `memoryGuard.run` with `foreign` nil runs unchanged. Correct.
- The step split (`judgeReport` and `writeReport` on reach, redundancy and mutation): each early-return case returns the same report the old `run` built. The default case calls `answer`, or `judge` for redundancy. `GateKey` and `Written` are stamped at write time, as before. Correct.
- `runTest` plan loop:
  - No earlier failure and an index equal to `reportsStart`: the chain starts. Correct.
  - An earlier failure: `code != 0`, so no chain starts. After the lints the steps judge red. Correct.
  - A later run fails, crashes or is blocked: `code != 0` and `chain.cancel()` runs. After the lints, `red` is true, so `stop` runs, then a red re-judge. Correct.
  - A halt: the call returns and the deferred `chain.stop()` waits for the steps, writing nothing. The defer was registered after `catchTerminatingSignals`'s `stop`, so it runs first. Correct.
  - `--stop-on-fail` breaks out of the loop: the chain was already cancelled, and the call goes through the red path. Correct.
  - The context is cancelled without a halt: the lint block is skipped and the deferred `stop` runs. Correct.
  - An empty plan: the chain stays nil and the steps judge synchronously with `red` false, as before. Correct.
  - A lint fails on a green tier run: `red` is computed before the lints, so the reports stay green, as before. Correct.
- The guard's lines and the steps' held lines: the reach map run uses the outer `guardKilled` and starts no guard of its own (`reach.go:278`). Its `stderr` serves only a guard it does not start (`testmap.go:199`). The chain goroutine is the only writer of each step buffer, and the guard goroutine the only writer of `guardLines`. `<-done` orders those writes before `write` reads the buffers. No race.

**Findings**

1. LOW, design, `internal/gatecli/reports.go:595-602` together with `internal/gatecli/redundancy.go:92`. The redundancy step's builds and tests now count toward the last tier's process cap while that tier runs. The cap trips on that combined count and then kills those processes too.
   - The redundancy step's children start from `e.env`. That env carries `LYM_GATE_TEST_PID=<pid>`, which `runTest` sets (`run.go`, the line that sets `exe.env`).
   - The tier's process cap finds its processes by that same entry: `runMarker(env)` at `processcap.go:94`, listed through `proc.PIDsWithEnv` with `runProcessCap = 1024` at `processcap.go:20`.
   - When the cap trips, the tier fails with the cap's line, which `tierrun.go:174-180` adds to its causes. A redundancy run can therefore turn a tier red that would have passed alone.
   - Before this diff the steps ran only after every tier ended, so their processes never shared the cap.
   - The margin is wide and the step runs one test per coverage binary, so the risk is low.
   - Fix: give the background steps an env without the call's marker, or a marker of their own.

No other function or branch the diff adds or changes gave a wrong result, a swallowed error or an inconsistent state on a failure path.

Tool calls:
1. Read `runMarker`, `newProcessCap` and the stray-file check in `strayfiles.go` and `tierrun.go`, to see what tier-level checks the concurrent steps can trip. My context did not hold them.
2. Read the value of `runProcessCap`, how the tier uses the cap's line, any overlay use, and the env of each step. These settled whether the steps' processes carry the tier's marker.
3. Read the `lockedWriter` and `stderr` uses in the map and mutate runs, to check whether the step buffers get concurrent writers.
4. Read the map run's guard start (`testmap.go` 185-215), the reach step's map run and `mapRequest`, to confirm that the reach refresh starts no second guard writing into its buffer.
