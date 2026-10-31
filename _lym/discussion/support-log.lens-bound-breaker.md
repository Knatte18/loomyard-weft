```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens, attacking the sampler record's claims and bounds.

**Findings**

1. **MEDIUM — D9: the trial rule guards too little.**
   - The rule only forbids a dev build's `lym reed` verbs in this hub. But reed re-pins the bindings on *any* reed op under the op lock, not only on a `lym reed` verb (reedengine doc: "Every op under the blocking op lock reads the stamp … and re-pins the bindings when it … names another build whose commit time is not after the running one's").
   - Counterexample: a dev build runs `lym shuttle open`, a batten or loom verb that spawns a strand, or anything else that reaches reed in this hub. The operator's server is re-pinned to the dev build.
   - A production lym with an older commit time then never re-pins back, so the operator keeps the dev bindings.
   - Fix: the rule should forbid running the dev build in this hub at all, not just its reed verbs.

2. **MEDIUM — D1, D6 and the last acceptance criterion: the empty-landing claim is only half verified.**
   - The claim that a board-note-only run lands "through the existing empty-landing path" rests on commit `5f444b56`. That commit covers the PR-Review and Finalize resolver only.
   - Describe runs before them, with a description gate (`attempts: 3`). Its stencil (`landing-template-describe.md`) reads `git diff {{.merge_base}} {{.task_branch}}` and has no case for an empty diff.
   - No darn or tapestry run has gone through Describe and its gate with an empty diff. Whether the gate accepts a description of an empty diff was not checked.
   - Counterexample: an agent writes only a note, Describe produces a description the gate refuses three times, and the run halts at Describe.

3. **LOW — D1: "a docs-only diff runs no tests at Finalize" holds only on some hubs.**
   - `darnTreeScope` (`internal/loomcli/darnwiring.go`) returns not-scoped when `darn.yaml`'s `tree_scope` is empty, and the template default is `tree_scope: {}`.
   - On a hub that lists no trees, a docs-only sampler diff runs the full landing gate.
   - The claim needs "when the hub's `tree_scope` lists the touched trees".

4. **LOW — D6: the claim "a resume of the same entry keeps the stamp" has nothing behind it.**
   - The record never says what the entry stamp is derived from. Without that, the row cannot tell a `goto` back to the row from a resume after a halt.
   - Counterexample: the agent runs `sampler-done`, then the strand dies before the wake. The halt is followed by `lym loom resume`. If the halt or resume writes a new stamp, the valid flag is ignored.
   - That case is benign, since the agent can run the verb again. The reverse case is not: if a `goto` and a resume look the same, a stale flag ends a re-entered row at once.
   - Fix: name the stamp's source, for example the status history length, and what moves it.

5. **LOW — D6: the order of the pause path and the dead-strand check is unstated.**
   - The row stops the session while a pause covers the run, and halts on a dead strand at any wake.
   - If the dead-strand check runs on the first uncovered wake before the resume, the strand the row stopped itself reads as dead, and the run halts on every pause.
   - Fix: state that resuming at the first uncovered wake comes before the dead-strand check.

6. **LOW — D6: an operator who closes the window to stop is overruled.**
   - The halt on a dead strand goes through the stop chain (`PATTERN-shed-stop-chain`), and the tackler or arbiter may run `lym loom resume` itself. That resumes the session the operator just closed.
   - Separately, `lym reed resume` relaunches a non-live unawaited strand, since "an empty done-when list never drops it as finished" (shuttleengine doc on unawaited sessions).
   - The record says nothing about either path. It should say the halt is judged first, or accept the reopening.

7. **LOW — D7: the bound's wording is off.**
   - It says a wedged waiting row "is ended by the operator's pause". A pause holds a step; it never ends one. Only `lym batten cancel` or the row's own halts end it.

**Claims attacked**
- D5, no shuttle deadline: held. The shuttleengine doc says an unawaited session has "no timeout classification".
- D5, reuse a live strand, resume a recorded session id, else start new: held. `prsession.go`'s `ensureSession` updates the strand GUID, name and session id on a resume.
- The constraint that without D7 a waiting row is killed by the watchdog: held. `agentActivityFor` skips waiter-gone runs and a held marker is not activity.
- D7, quiet notice suppressed by wait-live: held.
- D4, the `--by operator` limit and the end of the row: held. Its bound already admits the label is unverified.
- D3, `entry_gate` reaching a `sampler` module: held. Env.Setting goes through `configreg.SettingLookup`, so the module needs its configreg registration, which D3 includes.
- D1, PR-Review passes with no pull request on an empty branch: held for that row. Describe is the gap, see finding 2.
- D2, a template sampler block per profile with no default: held. An existing hub is refused at seed with a way forward, already an open risk.
- D9, one server per hub: held. Re-pinning reaches further than the rule covers, see finding 1.
- D6, the dirty tree refused at the verb and re-checked at the flag: held.

**Tool calls**
1. Read the shuttleengine doc on the orphan sweep and unawaited sessions, to check the deadline and waiter claims in D5 and D7; my context lacked them.
2. Searched for the `verify_source`/`tree_scope` wiring and the entry-gate setting source, to check D1's docs-only claim; my context had no wiring for it.
3. Read the Describe stencil's diff handling and how `prsession.go` records the strand, for the empty landing and D4/D5; my context lacked both.
4. Read how Env.Setting resolves and the Describe stencil's lines on the diff, to check D3's entry gate and Describe on an empty diff; my context lacked them.
5. Read how reed and tender revive and resume strands, to check D6's dead-strand halt against relaunching; my context lacked it.
