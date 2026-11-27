```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the code that already reads or writes each state the decisions change. Six findings: four MEDIUM and two LOW.

**Findings**

1. **MEDIUM — D1: three validators already refuse `unattended`, and the record names only the lookup.**
   - `dobbyengine.checkTaskSetting` accepts only a YAML boolean for `autonomy` ("must be true or false").
   - Three other paths call it besides the lookup:
     - the beam config load's check of the resolved file (`internal/dobbyengine/config.go`, the loop over `PullRequestKey, LandNoticeKey, AutonomyKey, RunWatchKey`);
     - `CheckLocalValues`, which `configreg` registers as the beam module's `CheckValues` and which gates `lym config beam --set` and a task's `run_settings`;
     - the lookup's read of the template value.
   - D1 says only "Any other value is refused by the lookup", so a plan could widen the lookup alone. `lym config beam --set autonomy=unattended` and a board `run_settings` holding it would then still be refused.
   - Fix: D1 names `checkTaskSetting` as the one place that learns the third value, and states its refusal text for all of these paths.

2. **MEDIUM — D1: an older binary refuses the whole beam config once `unattended` is set.**
   - The beam module's load check is strict through `checkTaskSetting`.
   - The hub's tracked beam config is shared by every machine and every lym build. That includes a long-lived dobby started by an older build and a driver kept on a pinned build until hand-over.
   - Any of them would refuse the resolved beam config and fail every beam read: `max_runs`, the gate hold, all of the dobby's config.
   - The record says no reinterpretation ships, but says nothing about an older reader.
   - The record should state the bound: what an older binary does with the value, and that the value should be set only once every machine runs the new build. Otherwise it should pick a shape older readers tolerate.

3. **MEDIUM — D4 with D5: a driver waiting on a pause, or on a network stop, never parks, so a resume waits out its cap and then refuses.**
   - Today a pause lift posts a resume line to the orch (`internal/orchengine/pause.go`, `queueResumeLine`), and the orch's usual answer is `lym loom resume`.
   - Under D4 the driver holds its driver lock and waits through the pause; `push-retry-not-halt`'s D6 network wait does the same.
   - D5 makes resume wait "for the park" while the driver lock is held. A waiting driver never parks, so the orch blocks for the keep-warm cap and is then refused `driver_not_parked`, which is the noise D5 means to remove.
   - D5 should read a held driver lock whose driver keeps a wait record as "already being driven": it answers at once and does nothing. It should wait only for a driver that is about to park.

4. **MEDIUM — D4 with D3: a machine wake resets the repair count, so D4's "per run at the row" bounds do not hold.**
   - `drive.go` zeroes `facts.RepairCount` when a completed-wake stamp comes at least `wakeRestartGap` (10 min) after a key's last re-step.
   - A row that keeps failing across machine wakes or long waits gets its "two settles" or "one fresh start" back each time.
   - D3's whole-history count still reaches the severe verdict, but D4's stated bound is false under the existing wake path, and the record is silent.
   - The record should say whether D4's bounds count from D3's history count, so the wake does not reset them, or keep the wake reset and restate the bound.

5. **LOW — D7: the hand-back sends notices only when an orch strand is recorded.**
   - `handBackRun` in `internal/sheddriver/drivehandback.go` otherwise only logs and writes the stop report.
   - D7 puts the Slack post in the hand-back. Unless the record says the post does not depend on a recorded orch strand, a run with none never reaches the operator, which is the case D7's rationale is about.

6. **LOW — D8 with D11: two existing guards filter out what these decisions add.**
   - D8: `readWait` returns no wait unless the child's state is running and its row is in `Rows`. A child held at a severe halt is not running, so extending `Rows` alone reminds of nothing. D8 should say how a severe-held run is identified, for example from D7's record of the post.
   - D11: the held notice in `internal/shuttleengine/hold.go` comes from runners. The reply probe is wired only in the loom, burler and webster wirings (`runner.SetReplyProbe`). The other runners, darn and raddle among them, would still send the held notice while their agent awaits a reply.

**Paths traced**

- **D1:**
  - `slackcli` hook's `orchAutonomous` (bool; reads "on" correctly if both levels map to on): survives.
  - `orchcli.WithRunAutonomy` (formats `%t`; it is the one writer of the stamp, and no Go parser of the stamp was found): survives once changed there.
  - `loomcli` pawl and drift wiring, and `orchcli` paths' `Autonomy`: bool readers, survive.
  - `hubgeom` beam modes: survives.
  - `landingshed` Finalize skips the autonomy key in its changed-keys list: survives.
  - `checkTaskSetting`'s three other paths (load, `CheckLocalValues`, template): break (finding 1).
  - Older-binary loads: break (finding 2).
- **D3:**
  - `repairCount`, plus the wake reset in `drive.go`: breaks D4's bound (finding 4).
  - `go-settle.jsonl` lives in the run directory; I found no sweep that removes it: survives.
- **D4:**
  - Pause lift, the orch's resume line, then `loom resume`: breaks with D5 (finding 3).
  - The bounce-budget resume made by the driver while it holds the driver lock has to be the in-process transition, not the verb, or the verb's own refusal stops it. The record's PATTERN-run-fsm line covers this: survives.
- **D5:**
  - `internal/battencli/wire.go` maps the kind to `ErrChildNotParked`. Batten's in-process resume would now block for up to the cap inside a dobby or batten tick, which the record does not mention: LOW, not filed separately.
- **D6:**
  - The existing `AutoAccept` and `softCapRule` accepts still settle before any escalation, and `BlockingEscalates` still raises budget, which the arbiter now admits: survives.
- **D7:** `handBackRun`'s orch-strand gate: finding 5.
- **D8:** `readWaits` and `readWait` running-state guard: finding 6.
- **D10:**
  - The run watch's stop mail runs alongside the chase; they do not conflict.
  - A run in a wait is never handed back, so it is never chased: survives.
- **D11:** the reply probe is wired only into some runners: finding 6.

**Tool calls**

1. Grepped for the readers of `hubgeom.Autonomy` and `LookupAutonomy`: my context lacked the full list of autonomy readers.
2. Grepped for the `autonomy:` stamp and the value checks: to find whether anything parses the stamp or validates the value.
3. Read the beam config validation in `internal/dobbyengine/config.go` and `landingshed`'s use of the key: to see which paths refuse a non-boolean value.
4. Grepped for the callers of `CheckLocalValues` and `checkTaskSetting`, and read `reinterpret.go`: to find the `run_settings` and `--set` gates.
5. Grepped for pause-lift resumers, `wakeRestartGap` and `repairCount`, and the tackler record functions: to trace what resets the settle count.
6. Grepped the pause watcher's resume line and the settle log's file name and removal: to check the pause-lift path and the settle records' lifetime.
