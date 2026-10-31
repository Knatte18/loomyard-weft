```yaml
lens: way-forward
outcome: reported
```

Way-forward review: each refusal, stop and recovery step the cards add, the next step it names, and whether that step can succeed from the state it leaves.

**Findings**

1. **MEDIUM — Card 4 (D5, D6): the start-failure halt and the dead-strand halt name only `lym loom resume`, and that resume cannot succeed once the recorded session can no longer be resumed.**
   - Card 4 lines 30–31: a non-transient start failure is Stuck with Halt naming `lym loom resume`.
   - Card 4 line 35: a strand that died unstopped is Stuck with Halt naming `lym loom resume`.
   - Card 4 line 30: the resume re-enters the row. With no current flag, `ensureSession` starts the spec again with `ResumeSessionID` set to the same recorded id, fails the same way, and halts again with the same advice. Nothing in the state changes, so the loop never ends.
   - The record's D6 Bound gives the way out: a flag written while halted (`lym loom sampler-done`, after committing) ends the row at the next resume without a session; failing that, `lym batten cancel`. Neither halt reason names either.
   - The same loop hits the entry path of card 4 line 32: a dirty checkout at the flag clears it and ensures the session, which fails to resume.
   - Fix: both halt reasons should also name committing the work and running `lym loom sampler-done` before `lym loom resume`, or `lym batten cancel <slug>`.

2. **MEDIUM — Card 10 (D6): the no-session refusal's way forward cannot be met when the row's first start failed.**
   - Card 10 line 31 refuses a record that is absent or has no strand, with "wait until the Sampler-Session row has started its session".
   - When the first start fails non-transiently (card 4 line 31), the run halts with no strand recorded. Each `lym loom resume` fails the start again (finding 1), so the session never starts and the wait never ends.
   - The verb's refusal and the row's halt then send the operator back and forth with no exit. The refusal should also name `lym batten cancel <slug>`, or the halt reason should carry the exit.

3. **MEDIUM — Card 9 (D3): the new refusal of a malformed `sampler.yaml` names a way forward that cannot fix the file.**
   - Card 9 line 40 refuses a malformed `sampler.yaml` with `hubConfigWayForward`. That text is "run lym config reconcile --apply in the hub, then step again" (`internal/loomcli/wiring.go:1057`).
   - The test row (card 9 line 43) is `agent: "opus[[bad"`. That value is one the hub tuned, and the sparse hub base keeps it, so reconcile leaves the bad value in place and the next step refuses again.
   - Fix: a model-spec refusal should name `lym config sampler --set agent=<model-spec>` from the prime, then `lym loom resume`.

4. **LOW — Card 4 (D5): an error from the `Spec` seam has no stated end or way forward.**
   - Card 4 lines 28–31 classify only failures of `StartSession`. They do not say how `Spec()` failing ends the call.
   - `SessionSpec` errors on an agent the registry cannot resolve or a missing stencil (card 3 line 34). Such an error is neither transient nor fixed by a re-step.
   - As a plain error, the run ends `failed` with no way forward. The card should say it is Stuck with Halt naming `lym config sampler --set agent=…` (or restoring the stencil), then `lym loom resume`.

**Checked, no finding**
- Card 4, damaged record at the flag → restore under `_lym`, `lym fabric commit`, `lym loom resume`. Resume adds no history entry, so the flag still carries the current stamp and the row ends Done.
- Card 4, dirty checkout during the wait → the mail names committing and `sampler-done`, so the next flag ends the row.
- Card 10, no Sampler-Session row → "none is needed".
- Card 10, wrong current row → `lym loom status`.
- Card 10, dirty worktree → commit, then re-run. `DirtyPaths` is plain `git status --porcelain` in the code checkout (`internal/verifytree/verify.go:261`), so a commit clears it.
- Card 10, failed seam read → re-run.
- Card 11, sampler strand with another label → `--by operator`, or the parent.
- Card 11, a strand after the row ended → the existing parent rule names the parent and the operator's shell.
- Card 11, reader error → retry.
- Card 5, config key on the entry → remove the key.
- Card 7, batten child refusal → names the four admitted recipes.
- Card 8, sampler Finalize reroute with the perch off → `keepFixerCommits` halts with "review and fix on the task branch, then `lym loom resume`" (`internal/landingshed/finalize.go:819`), which re-runs Finalize.
- Card 1, not initialized → `lym fabric reconcile`.

**Tool calls**
- Read `hubConfigWayForward`, Finalize's reroute halt and `verifytree.DirtyPaths`: my context lacked the way-forward text, the halt wording and the dirty-path source.
- Read `keepFixerCommits`' halt text: the first read did not show the way forward Finalize gives when no review follows.
