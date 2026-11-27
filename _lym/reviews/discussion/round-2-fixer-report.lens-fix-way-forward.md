```yaml
lens: fix-way-forward
outcome: reported
```

Fix-way-forward lens: checking that every way forward the fix added can be carried out from the state it names and makes progress.

**MEDIUM: D4's "retried on a later tick" cannot be reached as written**
- Changed text (D4): "The lift gives such a run a batten pass … a resume refused because the driver has not parked is retried on a later tick."
- The dobby marks a lift only once, on the edge where a run stops being covered. `triggerFor` in `internal/dobbyengine/passes.go:360-361` sets `d.liftPending[run.Slug] = true` only when the run was covered before and is not now.
- Passes go only to a drivable run. For any other run the mark is deleted (`passes.go:210-212`: "a later pass must not inherit it"), and after the pass is spawned (`passes.go:228-231` area).
- After a refused no-wait resume the run is still halted and not drivable, and the mark is gone. No later tick sees a lift, so nothing retries and the run stays halted until the orch or the operator acts.
- The mark lives only in memory (`wasCovered`, `liftPending`), so a dobby restarted between the lift and the retry loses it too.
- The fix needs to keep the lift mark until the resume succeeds, written to the dobby's state record, and to let D4's lift give a pass to a run handed back for that pause even when it is not drivable. Without that, the new D4 criterion "resumes the run on a later tick" cannot be met.

**LOW: D7's answer "the operator decides" leaves the agent with nothing to act on**
- Changed text (D7): "for an agent question it relays as severe, it answers the agent that the operator decides, which ends that question's chase."
- Once the question is answered, D11 no longer counts it as a correct wait. A D11 open question is "neither answered nor settled". So the agent's next idle raises the quiet notice for an agent that is waiting correctly, which is the noise D11 removes.
- Nothing names how the operator's later answer reaches the agent: the orch relaying it with `lym creel post` to the agent.
- A fix would either hold the run as for a relayed halt (D4) or state that the orch forwards the operator's answer to the agent, with the agent's wait for it read as a correct wait.

**LOW: D6's "handed on" leaves the unattended orch no rule for an unreadable input**
- Changed text (D6): "An escalation whose record or detail names an input that could not be read … is handed on at every level."
- Under `unattended`, D6's orch rule covers only "an open BLOCKING finding … or a red gate". A round whose ledger could not be read shows neither, so nothing stops the orch from accepting it.
- The escalation record's own way forward ("fix or delete the file the error names, then run `lym loom start`", `internal/shedadapters/bouncer.go:1671`) is not named for the orch.
- A fix would state that the unattended orch holds such a run (D4), naming that way forward in its note.

The other ways forward lead to progress:
- A `budget` escalation at the hard cap goes to the parent, where `accept` is never refused by the cap.
- A Finalize hold is lifted by `lym tender resume <slug>`.
- A bolt-hold entry replaced by a hold entry ends with the orch or the operator resuming the run.
- With the button off, the dobby sends the severe post's line to the orch, which relays it in its pane.

Calls made:
- I read `internal/dobbyengine/passes.go:195-213,345-365`, because my context showed only the grep hits for `liftPending`, not when the mark is set and cleared or whether it survives a refused pass.
