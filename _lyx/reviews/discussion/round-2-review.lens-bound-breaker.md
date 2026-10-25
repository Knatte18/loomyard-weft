```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: attack every bound, guarantee and existing-code claim in the decision record.

**Claims attacked**

1. D5/D6: `lyx loom goto Describe`, and in darn a goto to `Code-Bouncer`, lets a run go on past `Drift-Verify` without regression — **broken**.
2. D4 and the Constraints: a capture started by the loom row outlives the row and "ends with the run's processes" — **broken on Windows**, held on Linux.
3. D4: a capture started by `lyx drift capture` at a halt "ends with the run's processes", and `pairteardown` ends it through `runproc.EndSparing` — **broken on Windows**.
4. D5: the wait on a running capture has "no bound of its own", and only the capture's timeout bounds it — **broken**.
5. D2/D6: under scope `full`, no drift is accepted before the parent rules — **broken**.
6. D7: a parent's `lyx drift withdraw` holds across resumes — **broken under `partial`**.
7. D6: "a further attempt takes a resume by the parent", and the budget bounds the agent's runs — **broken**.
8. Constraints: between `Drift-Baseline` and `Drift-Await` no row builds in the task worktree — **broken**.
9. D4: a released lock tells every reader the capture is dead, with no pid involved — held.
10. D4: the main capture refuses off the branch point and on a dirty tree, and `.lyx/` is excluded at clone — held (`internal/fabriccli/clone.go:113`).
11. D4: the tree kill at timeout matches `runTiedChild` and frees the lock and slot; the log file as stdout avoids a pipe hang — held.
12. D4: the monotonic timeout does not advance during sleep — held for Linux; Windows is not stated (not a broken bound).
13. D5: re-reading the record after the lock check sees a capture that just ended — held.
14. D3/D4: the backup is the only refresh, and only before the first code change — held.
15. D1: requiring `{baseline_dir}` means no capture writes where lyx cannot see it — held only in text, since a command may carry the placeholder without the harness honouring it. D4's "at least one file written" covers the risk. No finding.
16. D2: changes Finalize makes after `Drift-Verify` go uncompared — held for a plain landing. In loom a Finalize reroute goes back through `Parent-Align` and the Code perch to `Drift-Verify`, so they are compared there. This is a precision point, not a broken bound.

**Findings**

**F-BB1 — BLOCKING, design. D5 (the halt naming `lyx loom goto Describe`), D6 (the "no passed capture record" halt), the darn `Code-Bouncer` goto, and the acceptance criteria on defer and adopt.**
Every "go on without regression" way forward is a forward goto, and goto refuses a forward move.
`CheckGotoTarget` refuses any target past the current row: "goto target %q lies past the run's current row %q; goto only moves a run back, so it never skips a row's own work".
`Describe` comes after `Drift-Verify` in loom, and darn's `Code-Bouncer` comes after `Drift-Verify` in darn.
Counterexample: defer past a failed capture, then resume. `Drift-Verify` halts naming `lyx loom goto Describe`, the goto is refused, and the run has no way forward.
The adopt path ends at the same halt, and so does every scope-`full` halt over a missing baseline.
Fix: give the halt a real way forward. One option is a `lyx drift` verb, allowed only at that halt, that records regression as given up, so that `lyx loom resume` returns Done. Under `full` it is the parent's decision.
Evidence: `internal/shedengine/goto.go:149-164`; `lyx loom goto --help` ("moves a halted run only back"); `contracts/recipes/darn-recipe.yaml` row order.

**F-BB2 — BLOCKING, design. D4 (the row "detached … returning Done once the child holds the capture lock, so Plan-Write runs meanwhile"; "ends with the run's processes") and Constraints (`pairteardown` … `runproc.EndSparing`).**
On Windows the capture dies when the `Drift-Baseline` step exits.
The loop starts each step with `proc.StartTied`. On Windows that puts the step in an unnamed kill-on-close job whose handle `releasePlatform` closes once the step's `Wait` returns.
`proc.Detach` on Windows does not break away from a job, and a breakaway would be refused anyway because the job does not allow it. So the capture child sits in the step's job and is killed when the row returns Done.
`Drift-Await` then reads "running with lock free" and halts on every Windows run.
The other direction fails too: a capture started by `lyx drift capture` from a caller's shell is in no lyx job, and `runproc.End` signals nothing on Windows. Teardown and adopt never end it, which breaks the Constraint.
The operator's C# harness, Models on OSL-1046, is a Windows/.NET target.
Fix: state how the capture is created on Windows. It must survive the step but still be reachable by teardown, for example by holding a named job of its own that `pairteardown` terminates, as the loop does. Or the record states that the design is Linux-only.
Evidence: `internal/shedverbs/loopchild.go:16,34`; `internal/proc/tied_windows.go` (`startPlatform`, `releasePlatform`); `internal/proc/namedjob_windows.go` (`createKillOnCloseJob` sets only `KILL_ON_JOB_CLOSE`); `internal/proc/proc_windows.go:31-35`; `internal/runproc/doc.go` ("On Windows … End and EndProcess signal nothing").

**F-BB3 — BLOCKING, design. D5 (capture `running` with its lock held: "wait … with no bound of its own"), D3 (darn's foreground capture), D6 (the mechanical and final compares).**
The loop's step idle watchdog is a bound the record never mentions.
`--until-stop` steps are killed after `step_idle_timeout_min` (default 60) with no activity.
`agentActivityFor` counts only four things: a live verify marker, a parent-lock wait, an unheld shuttle wait marker, and live agent runs.
None of the following is an agent or a verify:
- `Drift-Await` waiting on a multi-hour capture;
- darn's `Drift-Baseline` running the capture in the foreground after its agent ended;
- `Drift-Verify`'s mechanical compare;
- the final compare.

Counterexample: a 90-minute Models capture. `Drift-Await` is watchdog-killed at about 60 minutes and the run stops interrupted, not waiting.
In darn the kill takes the tied capture command down with the step.
The parent-lock wait is the precedent: it reads as activity.
Fix: the record states that a held capture lock and a running compare read as activity, through the same seam.
Evidence: `internal/loomengine/template.yaml:44`; `internal/loomcli/looparm.go:129-170` (`agentActivityFor`, parent-lock precedent at 139-143); `internal/shedverbs/loopwatchdog.go:170-260`.

**F-BB4 — BLOCKING, design. D2 ("Under scope `full`, every `Drift-Verify` halt over drift is a decision halt … so no drift … is accepted before the parent rules"), D6 (the halt list), D7 (accept verb bound).**
Some `full`-scope `Drift-Verify` halts carry no `DecisionHaltPrefix`:
- the dirty tree after three re-prompts;
- the `drift-verify` test gate failing after three re-prompts;
- an error or transient stop of the row.

`arbiterstop.Eligibility` admits a blocked halt without the prefix, and the arbiter is a "caller at a `Drift-Verify` halt".
D7's verb gate checks only the current row, so the arbiter's `lyx drift accept` succeeds under `full`.
D7 says the arbiter accepts "under scope `partial`", but no verb rule enforces it, and the verb cannot tell the arbiter from the parent.
Counterexample: under `full`, the agent fixes and commits code, and the `drift-verify` gate keeps failing. The row goes Stuck with no prefix. The arbiter accepts the drifting test and resumes. The compare passes with no acceptance by the resumed session, and the row returns Done without the parent ruling.
Fix: every `Drift-Verify` halt under `full` carries the prefix. Or `lyx drift accept` and `withdraw` under `full` are refused to anyone but the parent, by a stated mechanism.
Evidence: `internal/arbiterstop/arbiterstop.go:62-87,164-165`; decision record D6 lines 166-170 and D7 line 182.

**F-BB5 — MEDIUM, design. D7 ("so a rejection holds across resumes; only a caller at a halt accepts it again").**
Under `partial` the arbiter is a caller at a halt and is eligible for non-decision halts.
Counterexample: the parent withdraws test T and resumes. The compare fails on T, so the row is Stuck naming `lyx drift accept`. The arbiter runs `lyx drift accept T` and resumes, and the parent's rejection is undone without the parent.
Fix: an accept of a withdrawn test is refused to everyone but the parent, or such a halt is a decision halt.
Evidence: decision record lines 181-185; `internal/arbiterstop/arbiterstop.go:66-70`.

**F-BB6 — MEDIUM, design. D6 ("The row resets the counter at each fresh spawn of its agent, so each attempt has `verify_max_runs` runs and a further attempt takes a resume by the parent").**
Under `partial`, and under `full` at the halts F-BB4 names, the arbiter may settle a `Drift-Verify` halt with `lyx loom resume`.
Each resume spawns a fresh agent and resets the counter.
So the budget bounds one attempt, not the agent's total runs, and a further attempt does not need the parent.
Fix: state the arbiter-resume bound, meaning whatever caps the stop chain's resumes, or keep the counter across resumes the parent did not make.
Evidence: decision record line 160; `internal/arbiterstop/arbiterstop.go:66-70`.

**F-BB7 — MEDIUM, design. Constraints ("Between `Drift-Baseline` and `Drift-Await` no row builds in the task worktree: Discussion and Plan rows write only records").**
The Plan perch's `Plan-Burler-Review` runs with `tool-use: true`, and the burler explore step tells reviewers to "Drive the real substrate: build, run, test what you review".
Those reviewers run in the task worktree while the detached capture builds there.
In a .NET hub, a reviewer's `dotnet build` or `lyx gate test` writes the same `bin`/`obj` the capture's `dotnet test` is building. The gate slot bounds how many run at once, not whether they run in the same directory.
Counterexample: a plan reviewer runs a test tier mid-capture, and the capture's build output is corrupted or rebuilt under it.
Fix: state the guard, such as a held capture lock refusing a build in the worktree, or bound the claim to what holds.
Evidence: `contracts/recipes/loom-recipe.yaml` (Plan-Burler-Review `tool-use: true`); `.lyx/burler/review-3958036387/instruction-1-explore.md:181`.

**Tool calls**
- `grep` of `internal/shedverbs` for the loop's step child runner: my context lacked how a step child is started, which decides whether a detached grandchild survives on Windows.
- `grep` for `info/exclude` writers in `internal`: my context lacked the code behind D4's claim that `.lyx/` is excluded at clone.
