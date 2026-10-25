```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against its stated purpose.

**List: each decision, the case used, purpose met or not**
- D1, flag on, `capture` declared with `{baseline_dir}`, read by `Drift-Baseline`'s entry gate: met (`SettingKnown` is filled for loom and darn alike, `internal/loomcli/arm.go:248` and `wiring.go:765`).
- D2, flag off, loom run reaching `Drift-Await`: **not met**. The body and the acceptance criteria say Done and ungated, but the parent's F10 ruling says gated and Skipped. Under the ruling, D2's stated guard is gone: a capture still running when the flag flips leaves Webster unguarded.
- D3 (loom), scope `partial` in loom: capture started detached, scope and `baseline.md` committed, then `Plan-Write` runs: met on Linux; **not met on Windows** (see D4).
- D3 (re-entry), the agent wrote `scope.json` and died before `baseline.md`: **not met**. Re-entry skips the agent because a scope record exists, so `baseline.md` never gets written, and `Drift-Verify`'s agent is handed a missing expectation.
- D3 (darn), foreground capture longer than `step_idle_timeout_min`: **not met** (watchdog, see D5).
- D4, Linux timeout: the capture process `StartTied`s the command, `Tied.Kill` kills the group, then `failed` is recorded and the lock and slot are released: met.
- D4, Windows, loom capture detached from the `Drift-Baseline` step: **not met**. The capture dies when the step exits.
- D5, passed capture, HEAD unchanged: Done, `awaited_at` written, a later Webster reroute passes: met.
- D5, failed capture at the branch point: `lyx drift capture` then `lyx loom resume` waits for the new capture, which passes: met on Linux.
- D5, capture running longer than 60 minutes while `Drift-Await` waits: **not met**. The loop's idle watchdog kills the step.
- D5, adopt with `awaited_at` set, then the `Drift-Verify` halt naming `lyx loom goto Describe`: **not met**. The goto is refused.
- D5, deferral past a failed capture, then the `Drift-Verify` halt naming the goto to Describe: **not met**. Same refusal.
- D6, scope `full`: the agent accepts X, the compare passes, Stuck as a decision halt; the parent resumes, the mechanical compare skips X and passes, Done with no spawn: met.
- D6, scope `partial`: the agent fixes and accepts, the handoff runs the dirty check, then the `drift-verify` gate, then the final compare, which passes: Done, met.
- D6, a halt on no passed capture (darn naming `Code-Bouncer`): **not met**. The goto is refused.
- D7, the parent withdraws X; on resume X runs again, the agent's accept of X is refused, and the agent fixes or stops: met.

**Findings**

1. BLOCKING · design · D5 ¶4, D6 ¶1, acceptance criteria (defer and adopt lines): every "go on without regression" way forward is a goto the engine refuses, and the halt can never be lifted.
   - The `Drift-Verify` halts name `lyx loom goto Describe`, and in darn `Code-Bouncer`.
   - Both targets lie after `Drift-Verify` in their recipe: loom has Code-Bouncer → Drift-Verify → Describe, and darn has Darn-Write → Drift-Verify → Code-Bouncer.
   - `CheckGotoTarget` refuses a target past the current row ("lies past the run's current row … goto only moves a run back"), and `lyx loom goto --help` says the same.
   - `lyx loom resume` only re-enters the same halt, since there is still no passed capture.
   - So after a deferral, or after an adopt once `awaited_at` is set, the run is stuck for good, the parent included.
   - Fix: give `Drift-Verify` its own lyx-written way past, for example a `lyx drift defer`-style verb at the `Drift-Verify` halt that records the skip and lets resume return Done. Name it in D5, D6 and the acceptance criteria, and bound it under scope `full`.
   - Evidence: `internal/shedengine/goto.go:149-164`; `lyx loom goto --help` ("goto moves a halted run only back"); `contracts/recipes/darn-recipe.yaml` (Darn-Write `on_done: Code-Bouncer`).

2. BLOCKING · design · D4 ¶1 and ¶4, D3 ¶4 (loom detached capture): on Windows the detached capture is killed the moment `Drift-Baseline`'s step exits.
   - Each loop step child starts through `proc.StartTied`, which on Windows puts it in its own unnamed kill-on-close job.
   - `Wait` then calls `releasePlatform`, which closes that job's handle.
   - A grandchild started with `proc.Detach` has no breakaway flag, so it stays in that job and dies with it.
   - The capture therefore dies right after the row returns Done. `Drift-Await` then always reads "lock free, record `running`" and halts. The `Tied.Kill` rule in D4 covers only the timeout case.
   - The Models target (.NET, OSL-1046) makes Windows the primary platform for this path.
   - Fix: D4 states how the capture escapes the step's job, for example `DetachBreakaway` and its refused-breakaway fallback as `spawnLoop` does, and how it is then ended on teardown and adopt.
   - Evidence: `internal/shedverbs/loopchild.go:16-34`; `internal/proc/tied_windows.go` (`startPlatform`, `releasePlatform`); `internal/proc/proc_windows.go:31-44`; `internal/shedverbs/loopwait.go:158-161`.

3. BLOCKING · design · D5 ¶1 (the `running` + lock-held wait), D3 ¶4 (darn foreground capture), D6 ¶1 (mechanical compare): the loop's step-idle watchdog kills these long waits, so none ends with its stated outcome.
   - The loop kills a step that shows no activity for `step_idle_timeout_min`, 60 minutes by default.
   - Activity is a trace-file write, a live verify marker, a parent-lock wait, a shuttle wait marker or a live agent.
   - None of those exists while `Drift-Await` waits on a capture, while darn's `Drift-Baseline` waits on its foreground capture after the scope agent ended, or while `Drift-Verify` runs its mechanical compare.
   - A wait poll logs at Debug, which reaches the trace file only at Debug verbosity.
   - The step ends as an interrupted stop instead of Done or "Stuck naming the log". D5's "no bound of its own" therefore holds only for captures under an hour.
   - The record also gives the new rows no interrupt policy and no re-entry rule for a darn row re-entered while its capture still holds the lock.
   - Fix: make a held capture lock or a running compare count as activity in `agentActivityFor`, as a parent-lock wait does. State each Drift row's interrupt policy and the darn re-entry-while-locked rule.
   - Evidence: `internal/shedverbs/loopwatchdog.go:162-225`; `internal/loomcli/looparm.go:129-170`; `internal/loomengine/template.yaml:44`; `internal/logger/logger.go:46-50,314-322`; `internal/loomshed/interruptpolicy.go`.

4. MEDIUM · consistency · D2 ¶3, the acceptance criteria's first line, support-log Rejected alternatives and Q6, against the "Added after Discussion: Drift-Await is skipped through its entry gate" ruling.
   - The record still describes `Drift-Await` as ungated and recording Done.
   - It still keeps "Drift-Await behind `entry_gate`" as a rejected alternative, which the parent ruled for.
   - Run under the ruling, the flag-off case records `Drift-Await` as Skipped. The case D2's departure existed for, a capture running when the flag turns off, now lets Webster build beside the capture, and the record says nothing about it.
   - Fix: rewrite D2 ¶3 and the acceptance line to the ruling, drop the rejected alternative and Q6's pick, and state the mid-capture flag flip under the ruling.
   - Evidence: `_lyx/discussion/decision-record.md`, D2 and the F10 addition, as read above.

5. LOW · design · D3 ¶6 (re-entry): "a present scope record skips the agent" loses `baseline.md` when the agent dies between `lyx drift scope` and writing the report.
   - Re-entry keys on `scope.json` alone.
   - `baseline.md`, one of the row's two outputs and `Drift-Verify`'s expectation input, is then never written.
   - Fix: skip the agent only when both outputs exist, or re-spawn to write `baseline.md`.
   - Evidence: `decision-record.md`, D3 ¶4 ("the row's outputs are both files") against ¶6.

Tool calls:
1. `loopchild.go` `StartTied`, `TraceFilesFor`, `InterruptPolicyFor`: the context lacked how step children are tied and the row interrupt policies.
2. `interruptpolicy.go` table and logger doc grep: the context lacked the policy table and the trace-level rule.
3. Logger files holding the trace sink: to locate the durable-sink level policy.
4. `logger.go:40-60,310-330`: the context lacked whether Debug poll lines count as trace activity.
