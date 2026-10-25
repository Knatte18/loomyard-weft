```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked every bound, guard, ordering promise and code claim in the record with concrete counterexamples.

**Findings**

1. **BLOCKING — D1/D4/D6/D7: a side capture can overwrite the trusted baseline.**
   - D1 lets a hub's `capture.command` leave out `{baseline_dir}`.
   - D6 then says a side capture renders that same command with `{baseline_dir}` pointed at `.lyx/drift/side/<n>`.
   - Counterexample: Models keeps its harness default (`.scratch/numerical-baseline/`) and omits the placeholder. `lyx drift capture --side` then runs capture mode into the harness's own baseline directory, which lyx neither backs up nor knows about.
   - The post-change output becomes the baseline and the next compare passes. That breaks D4's "no run deletes or overwrites a baseline" and D7's "nothing in this design deletes, overwrites or refreshes a baseline".
   - Fix: refuse a `capture` or `compare` config without `{baseline_dir}`.

2. **MEDIUM — D4/D6: a run that discovers zero tests passes by exit code alone.**
   - Issue #609 Required 1 says a gated project run without its MSBuild property discovers nothing and exits green.
   - Counterexample A: the hub drops `-p:RunBaseline=true`. Capture exits 0, so D4 records "passed" over an empty baseline. Compare exits 0, so D6 returns Done. Regression is green without running.
   - Counterexample B: in D6, the agent calls `lyx drift accept` on every drifting fixture. If the drift touches every fixture, the skip file covers them all, the compare runs zero tests, exits 0, and the row returns Done.
   - Fix: a capture or compare that ran no test, or whose skip set covers every test, is a failure. Also, a run that wrote no file into `{baseline_dir}` is a failed capture.

3. **MEDIUM — D4: no runproc entry point fits "ends its command's process tree through runproc" on a timeout.**
   - `runproc.End(dir, owner, callerPID)` ends every live process the slug holds: the batten, the loop and the agent sessions. Only the caller and its ancestors are spared, and the command is a child, not an ancestor.
   - `PATTERN-run-process-owner` reserves `EndProcess` for the orphans `lyx tender reap` ends.
   - On Windows, End and EndProcess signal nothing. Models runs on OSL-1046, a work laptop with Cortex XDR (board note A5), most likely Windows.
   - Result: a timed-out `dotnet test` keeps running after the capture records `failed` and exits, which also releases the slot. A `lyx loom goto Drift-Baseline` then starts a second capture beside it in the same worktree.
   - Fix: name the end route, either a new runproc entry for one registered child or a PATTERN change, and say what happens on Windows.

4. **MEDIUM — D6: the compare is not re-run after a `verify`-gate re-prompt fix.**
   - Sequence in D6: the agent ends, the compare passes, the `drift-verify` gate fails, and the session is re-prompted to fix the tests.
   - The agent's test fix can bring the drift back. Nothing re-runs the compare before Done, so the "compare passes, then Done" promise breaks.
   - Fix: every handoff re-runs the compare after the last code change.

5. **MEDIUM — D6: the compare can pass on uncommitted edits.**
   - The compare runs on the worktree as it is, uncommitted changes included.
   - Counterexample: the agent fixes the drift without committing. The mechanical compare passes. D6 runs the `verify` gate only "when the agent committed code", so the gate is skipped and the row returns Done.
   - The fix is never committed. Describe and Finalize then work from a HEAD whose drift was never compared.
   - Fix: refuse a dirty tree once the agent ends, re-prompting it to commit, before the final compare.

6. **MEDIUM — D3/D4: a recapture after Webster began can bless drift.**
   - Re-entering `Drift-Baseline` starts a capture whenever the record is absent or failed. The only guard is the dirty-tree check, and committed code passes it.
   - Counterexample: the capture fails, `Drift-Await` halts, and the operator takes the offered `lyx loom goto Webster`. Webster commits code. `Drift-Verify` halts on the failed capture, and the operator runs `lyx loom goto Drift-Baseline`. Drift-Baseline captures post-change code, and the compare passes.
   - This is the exact case issue #609 item 1 forbids.
   - Fix: the capture refuses when HEAD carries code commits beyond the task's branch point, or once Webster has been entered.

7. **MEDIUM — D5: the wait bound can expire on a capture that is still waiting for a slot.**
   - The wait is "the capture's own timeout plus one minute". If that is counted from the capture's start, a capture queued behind busy gate slots, for example other tasks' Webster gates, outlives the bound while it is healthy.
   - D5 also never says what happens when the bound runs out.
   - Fix: record the slot-acquire time and the command start separately, count the bound from the command start, and state the outcome at the bound.

8. **LOW — D5: a liveness race gives a false Stuck.**
   - Interleaving: Await reads `running`; the capture writes `passed` and exits; Await checks the pid, finds it dead, and returns Stuck.
   - Fix: re-read the record after a dead-process check.
   - A related gap: D3/D4 do not say whether the row or the child writes the `running` record. A crash between the spawn and that write makes re-entry find no record and start a second capture into the same directory.

9. **LOW — D5 on Windows: a dead capture can look alive.**
   - runproc keeps an empty-start-time record alive "while its pid does". A reused pid makes a dead capture look alive, so Await waits out the full bound. This ties into finding 7.

10. **LOW — D4: "never runs beside another heavy test run" is false when `slots` is above 1.**
    - The template default is `slots: 2`, so a capture holding one slot runs beside one other gate run. Reword to "within the hub's slot count".

11. **LOW — D7 attack surface: acceptance is unbounded.**
    - The only bound on `lyx drift accept` is the current row. Even under scope `full`, where any drift is by definition a bug, the agent can accept any number of tests and reach Done with no review and no escalation.
    - The decision states no bound on what accept lets through. One option: under `full`, an acceptance forces Stuck for the orch.

12. **LOW — D6: the budget counter's home is unnamed.**
    - If `verify_max_runs` is counted in a `.lyx/drift/` file, the agent can edit that file and reset its own budget.
    - The record should say the count is Go-held and out of the agent's write reach, or accept the gap.

13. **LOW — Constraints, VSTest exit code: the evidence names no source or version.**
    - The claim that `dotnet test` exits 0 on an all-Inconclusive capture rests on "VSTest" with no source or version.
    - Under Microsoft.Testing.Platform, which .NET 9/10 `dotnet test` can use, a run with no test run exits 8. Whether Inconclusive counts as run is unsettled.
    - The capture's pass criterion rests on this fact: either record it with its source and version, or make passing mean "files were written" (finding 2).

14. **LOW — D2: the darn entry gate may not build.**
    - In this tree, `Env.SettingKnown` is filled only in `internal/loomcli/wiring.go` and in testkit. A darn recipe built by another caller with a nil `SettingKnown` refuses `entry_gate: drift.enabled` at build.
    - The record should state that darn's build goes through the same wiring, or have the plan wire it.

**Claims attacked**

- Held:
  - D2: an ungated Drift-Await holds Webster when the flag is turned off mid-capture.
  - D2/D5: the `awaited_at` mark lets Webster's plan reroute pass.
  - D4: the dirty-tree refusal is not tripped by Drift-Baseline's own `_lyx/drift` writes, because `_lyx` is a symlink to the weft worktree.
  - D5: Stuck for a lost capture record after an adopt.
  - D6: Stuck for a missing baseline.
  - D7: accept is refused outside Drift-Verify.
  - The constraint that no build runs between Drift-Baseline and Drift-Await, for the rows themselves; agents may still run `lyx gate test`.
  - The constraint that pair teardown ends the capture.
  - D1: the hub-wide entry-gate operand in loom (`configreg.CheckSettingKey`).
- Broken: D1/D4/D7 baseline overwrite (1), the zero-test pass (2), the timeout end route (3), the compare after a gate fix (4), a compare on a dirty tree (5), the branch-point recapture (6), the wait bound (7), the liveness race (8), the "never beside" wording (10), unbounded accept (11), the budget counter (12).
- Unsettled: the VSTest exit code (13), darn's entry-gate wiring (14).

**Tool calls**

- `ls -la _lyx` and `.gitignore`: to learn whether Drift-Baseline's `_lyx/drift` writes dirty the code tree; my context did not show whether `_lyx` is tracked.
- grep for `SettingKnown`: to learn whether a `drift.enabled` entry-gate operand builds in loom and darn; my context did not show the env wiring.
- grep for `SettingKnown:` and the runproc `End`/`EndProcess` signatures: to check darn's wiring and whether runproc can end one child; my context lacked both.
