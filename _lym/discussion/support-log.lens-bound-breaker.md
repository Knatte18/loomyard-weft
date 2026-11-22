```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I tried to break every bound, guarantee and claim about existing code in the record. 4 tool calls used.

**Findings**

1. **MEDIUM — D1, inherited worktree value.** "A nested beam is governed by its own `beam.yaml` stack, never by its parent's" fails in practice, because a new worktree's stack starts as a copy of its parent's worktree file.
   - Counterexample: the operator sets `autonomy: true` on the prime's worktree layer so root tasks run autonomous. Every beam made after that inherits `autonomy: true` without anyone choosing it. The goal "one beam autonomous while another beside it is not" then needs a manual opt-out on each new beam.
   - Evidence: this task pair's records worktree (`_lym` links to `…-weft/_lym`) already holds `_lym/config/beam.yaml` with `max_runs: 8`. That is the parent's worktree value, carried over at fork.
   - The record states the opposite of the observed behaviour and gives no rule for it. Pick one: `beam create` resets `autonomy`, or the record states that the value is inherited.

2. **MEDIUM — D3, incomplete list of operator seats.** The title says every operator-seat session becomes autonomous, but `raddle_auto_approve: false` (loom template) has the raddle agent "put the proposal to the operator in its pane".
   - Counterexample: a motif or arras run under an autonomous beam reaches Raddle-Write and waits in its pane for the operator. That is a third operator case the goal rules out.
   - D3 forces only `judge_interactive` and `discussion_interactive`. It should also force `raddle_auto_approve` (or route the proposal to the parent), or state why it leaves raddle out.

3. **MEDIUM — D4, unbounded wait.** PR-Review's wait has no timeout, and the parent ask is sent once (`rec.ParentReviewAsked`).
   - Counterexample: the orch's context cycles, or the orch is down, when `[pr-review request]` arrives. The mail is never answered, no operator sits in the pane, and the run waits forever with nobody to see it.
   - Under autonomy nothing re-asks, and nothing lands the stall in D5's list. D4 needs a re-ask or a deadline, or a severe-halt class for an unanswered ask.

4. **MEDIUM — D4, bound enforced only by prose.** "Runs `lym loom pr-continue` on the parent's satisfied answer" is a stencil rule only. Nothing records the parent's satisfied answer, and `pr-continue` does not check for one.
   - Counterexample: an autonomous PR-Review agent that misreads a findings reply, or a hold notice, runs `pr-continue`. The review ends with the parent's findings still open.
   - The Guard argument in D1 rests on the parent's answer standing in for the operator's, so the bound should be mechanical. One option: route the PR review through the `parent-review answer --ask` seat and have `pr-continue` refuse without a recorded satisfied answer.

5. **MEDIUM — D5 class 5, bound that cannot be enforced.** "Settled twice without moving past that row" depends on the orch's memory, which a context cycle wipes. The orch note holds run state, but nothing requires it to carry the count.
   - Counterexample: the orch settles the stop, its context cycles, it settles again, cycles again, and so on. The loop has no bound.
   - "Same cause" is also undefined for stop reasons whose text changes each time, such as a fixer reason naming different kept commits. The count needs a durable home (the orch note, explicitly) and a defined cause key, such as the row plus the halt kind.

6. **MEDIUM — D9, launch stencils left out.** Only start, resume and reload carry the autonomy line. `internal/orchengine/prompt.go` also has handoff, handoff-soft, compact, note and usage-limit-wake stencils.
   - Counterexample: after a handoff or compact cycle, the orch's new context has no autonomy line. It falls back to waiting for the operator's word, which contradicts the Auto-mode assumption that a change reaches the orch "at its next launch or context cycle".
   - `WithPresence` in `internal/orchengine/prompt.go` already appends a launch-time line to every pointer. The autonomy line belongs on the same path.

7. **LOW — D6/D7, re-arm undoes the edited description.**
   - Counterexample: step 1's `lym beam merge-parent` files a new member task, so the beam reopens. When that task lands, D6 posts a second `[beam drained]`. The orch re-runs step 2's `describe --draft`, which overwrites the `summary.md` it had already edited.
   - D7 should start at the first unfinished step, or skip `--draft` when a description is already recorded.
   - Separately, D6 does not say where the "once per drain" mark lives. Beam-Await is reinvoked on restart, so an in-memory mark posts again after a driver restart.

8. **LOW — D1 vs D2, two different owners for the beam's own run.** D2 answers a run's autonomy from its parent worktree. The beam's own pawl run is a run whose parent worktree is the prime, so D2 gives the prime's value. D1, D6 and the Auto-mode assumption give the beam's own value. The record should name which reader the beam's own Finalize and pawl rows use.

**Claims attacked**
- D1 "run_settings can never make a run autonomous": held. `PrepareRunSettings` runs `ValidateLocal`, which refuses non-None keys.
- D1 "a worktree layer overrides the default layer": held, per the `LayerFile` order.
- D1 "a nested beam is governed by its own stack": broken (finding 1).
- D2 "an unreadable stack reads off": held. It falls back to asking the operator.
- D3 "every operator-seat session becomes autonomous": broken (finding 2).
- D3 "turning autonomy off restores each key": held, since nothing rewrites a layer file.
- D4 "the resolver is unchanged": held.
- D4 "PR-Review is ended only by the parent's satisfied answer": broken (finding 4). Not mechanical.
- D4 "the session never waits": broken (finding 3). It waits with no bound.
- D4 "a GitHub human-approval rule halts the run at Finalize": held, per role-landing-halts.
- D5 "notices already reach the parent orch": held. Beam-Await's Notify uses `orchcli.NotifyOrchAt(location…)`.
- D5 class 5 two-settlement bound: broken (finding 5).
- D6 "Beam-Await reaches the beam's orch through Notify": held (`pawlwiring.go` `NotifyOrchAt`).
- D6 "posts once per drain": broken (finding 7).
- D7 "the go-ahead covers only these four steps" and "`lym beam ready` waits for the operator": held. Re-entry is weak (finding 7).
- D8 "the filter drops only unmarked replies while the presence is slack": held. It depends on the orch marking its answers to the operator, which is prose only.
- D9 "the orch learns its autonomy at launch": broken for the handoff, compact and usage-limit-wake paths (finding 6).
- Constraint "the verbs are open to an orch session": not attacked further. The explore results show no shellguard row against them.

**Tool calls**
1. Grepped the loom template for raddle keys and `pawlwiring.go` for Notify. My context lacked the full list of interactive keys and Beam-Await's Notify target.
2. Grepped orchengine for the orch stencil names, plus `ConfigFile` and `AnchorPath`. My context lacked the full set of launch stencils and where the worktree layer file lives.
3. Ran `ls`/`readlink` on `_lym` and read the orch pointer code. I needed to know whether `_lym` is the records worktree and how launch-time lines are added.
4. Read the task pair's `_lym/config/beam.yaml` and grepped the pointer builders. I needed evidence that a worktree's `beam.yaml` is inherited from its parent.
