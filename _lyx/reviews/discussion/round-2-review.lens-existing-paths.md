```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. For each state, file or invariant the record introduces or changes, I traced the existing code paths that write, read, reset or reapply it.

**Paths traced, per decision**

- D2/D6, Stuck halt with no `on_stuck`: `shedengine/run.go:394-403` persists `StateBlocked` with `stuckReason(output.Reason)`, which keeps the text unchanged (`run.go:45-51`). `arbiterstop.Check` (`arbiterstop.go:165`) and `HandsBack` (`:63`) read the `DecisionHaltPrefix`. This **survives**: the scope-`full` decision halts are detected.
- D2, `HandBack` wiring for darn: darn is built from the same `c.env`/`HandBack` as loom (`loomcli/arm.go:248,312`). **Survives.**
- D5/D6, the way forward `lyx loom goto Describe` (loom) and `goto Code-Bouncer` (darn): `shedengine.CheckGotoTarget` (`goto.go:149-163`) refuses any target "past the run's current row". **Breaks.**
- D2, loom's Finalize reroute: `on_reroute: Parent-Align`, then `Code-Bouncer`, then (new) `Drift-Verify`. D2's claim that Finalize's merge goes uncompared is **misread** for loom. It **holds** for darn (`on_reroute: Code-Bouncer`, which sits after `Drift-Verify`).
- D5, Webster's reroute through `Plan-Burler-Review`, `Plan-Bouncer`, then `Drift-Await`: `awaited_at` lets it pass. **Survives.**
- D3, an operator `goto Discussion-Write` or `goto Plan-Write` after Webster: `Plan-Write` leads through `Drift-Await`, where `awaited_at` passes (**survives**). `Discussion-Write` leads through `Drift-Baseline` re-entry; with the last capture failed or deferred, the row tries a main capture that D4 refuses off the branch point, and the row's outcome is unstated. **Breaks.**
- D5/D4/D6, the loop's step-idle watchdog (`loopwatchdog.go:170-225`; `step_idle_timeout_min: 60` in `loomengine/template.yaml:44`). Its activity sources (`looparm.go:129-170`) are the verify marker, a parent-lock wait, shuttle wait markers and agent activity. None covers a `Drift-Await` capture wait, darn's foreground capture, or `Drift-Verify`'s mechanical compares. **Breaks.**
- D4, Windows lifetime of the detached capture:
  - The loop holds a kill-on-close named job (`shedverbs/loop.go:187-193`, `namedjob_windows.go:18-33`).
  - `proc.Detach` sets no breakaway on Windows (`proc_windows.go:31-35`).
  - The loop returns at the first stop (`loop.go:1-3,161`).
  - So the capture the row spawns dies with the loop at any halt, for example a Plan-perch halt. **Breaks.**
- D4/Constraints, ending the capture through `runproc.End`/`EndSparing` at teardown or adopt: on Windows, End "signal[s] nothing" (`runproc/doc.go`). A capture started by `lyx drift capture` from a caller outside the loop job is not ended. **Breaks** on Windows; **survives** on Linux, where `planRecords` walks descendants and own-group children (`end.go:313-345`).
- D4, the timeout kill through `StartTied`/`Tied.Kill`, as `gatecli/child.go:22-47` does: with `Pdeathsig` and `Setpgid` (`tied_linux.go:16-19`), the command also dies with the capture process. **Survives.**
- D4, gate slot and lock released by process death: `gateslot/doc.go`, where an OS lock is released at death. **Survives.**
- D1/D2, `entry_gate: drift.enabled` on darn: `SettingKnown` is already filled for darn through the shared env (`wiring.go:765`, `arm.go:248`). The plan-writer note's premise does **not hold**.
- D2/D5 against the F10 ruling: with `Drift-Await` gated, the shed entry gate records it Skipped when the flag is off. `Relaunch` clears the entered row so the gate is re-evaluated (`goto.go` Relaunch doc). D5's promise that a capture still running when the flag flips off holds Webster back **breaks** under the ruling, and the record is silent about it.

**Findings**

1. **BLOCKING**, design. D5 and D6 (Drift-Verify's halt on no passed capture; D5's deferral and adopt paths; the acceptance criteria for defer and adopt).
   - Summary: every "go on without regression" way forward is a forward goto, `lyx loom goto Describe` in loom and `goto Code-Bouncer` in darn. `CheckGotoTarget` refuses a target past the current row, so the halt names a refused step and the arbiter or parent is stranded.
   - Evidence: `internal/shedengine/goto.go:149-163`; `lyx loom goto --help` ("goto moves a halted run only back").
   - Fix: give `Drift-Verify` a verb, such as a `lyx drift defer`-style record valid at a `Drift-Verify` halt, that `lyx loom resume` reads to return Done. Name that verb instead of the goto.
2. **BLOCKING**, design. D5 (the unbounded wait), D3 (darn's foreground capture), D6 (the mechanical compares).
   - Summary: the loop's step-idle watchdog kills a step that shows no activity for `step_idle_timeout_min`, 60 by default. None of these long, LLM-free waits is an activity source, so a capture or compare longer than the window gets the step killed. The kill takes its tied tree with it, darn's capture command included. The record is silent.
   - Evidence: `internal/shedverbs/loopwatchdog.go:170-225`; `internal/loomcli/looparm.go:129-170`; `internal/loomengine/template.yaml:44`.
   - Fix: a held capture lock or a running Drift compare reads as activity, as a parent-lock wait does.
3. **BLOCKING**, design. D4 ("ends with the run's processes") and the Constraint that "pairteardown ends ... the capture included, through runproc.EndSparing".
   - Summary: on Windows the row's detached capture sits inside the loop's kill-on-close named job and dies when the loop ends at any halt. `runproc.End` signals nothing on Windows, so a capture a caller started at a halt is never ended by teardown or adopt.
   - Evidence: `internal/shedverbs/loop.go:187-193`; `internal/proc/namedjob_windows.go:18-33`; `internal/proc/proc_windows.go:31-35`; `internal/runproc/doc.go` ("On Windows ... End and EndProcess signal nothing").
   - Fix: state the Windows lifetime. One option is `DetachBreakaway` plus a capture-held named job that teardown terminates, as the loop's job is.
4. **MEDIUM**, design. D3's re-entry rule.
   - Summary: after a `goto Discussion-Write` (or any re-entry of `Drift-Baseline`) once code has changed, a failed, dead or deferred capture makes the row try a main capture. D4 refuses that capture off the branch point, and the row's outcome (Done, Stuck, or which way forward) is unstated.
   - Evidence: `contracts/recipes/loom-recipe.yaml` (`Discussion-Bouncer` → [`Drift-Baseline`]); decision record D3 and D4.
5. **MEDIUM**, consistency. D2's "Changes made after Drift-Verify by Finalize's merge and merge fixer ... go uncompared".
   - Summary: in loom, Finalize's reroute runs `Parent-Align`, `Code-Bouncer`, `Drift-Verify` again. A rerouted merge is compared, so re-entry can spawn the agent and, under scope `full`, end in a decision halt. The record misstates this path and is silent on re-entry after a prior Done, such as acceptances already recorded and the counter.
   - Evidence: `contracts/recipes/loom-recipe.yaml` (Finalize `on_reroute: Parent-Align`; Parent-Align `on_done: Code-Bouncer`).
6. **MEDIUM**, design. D2 and D5 against `Added after Discussion` F10.
   - Summary: with `Drift-Await` behind `entry_gate: drift.enabled`, the shed entry gate records it Skipped when the flag is off. A capture still running when the flag flips off no longer holds Webster back, and D5's "holds Webster back" path is broken. The record does not say what bounds it, for example a refusal to flip the flag while a capture lock is held, or `Drift-Await` reading the lock before its gate.
   - Evidence: `internal/shedengine/goto.go` (Relaunch doc: "the entered row is cleared, so the target's entry gate is evaluated"); the F10 entry in the record.
7. **LOW**, consistency. Notes for the plan writer: "darn's recipe build must fill it too".
   - Summary: darn is already built from the same env, which fills `SettingKnown`, so the note's premise is false.
   - Evidence: `internal/loomcli/arm.go:248`; `internal/loomcli/wiring.go:765`.

**Tool calls**
- `grep StateBlocked` in shedengine and resume: to confirm how a Stuck with no `on_stuck` persists its reason, which the manifest lacked.
- `sed internal/shedengine/run.go:385-420`: to read the Stuck halt branch that keeps `DecisionHaltPrefix`.
- `grep func stuckReason`: to confirm the reason is passed through unprefixed.
