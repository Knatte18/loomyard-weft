```yaml
lens: fix-proposal
outcome: reported
```

Fix-proposal lens: challenging the reviewer's proposals the fixer carried out in the decision record.

Proposals challenged: F1, F2, F3, F4, F5, F6, F7, F8, F9, F11, F12, F13, F14, F16, F17, F21 (the fixer's chosen option).

1. **MEDIUM**: D8's dedup key and D7/D10's state-record key use "its row and history entry". A new halt can fold into the same history entry, so it reads as the old halt.
   - Changed text: D8 "the same slug and the same halt, its row and history entry"; D7 "named by its row and history entry"; D10 "still halted at that entry".
   - Evidence: `internal/shedengine/run.go:702-710` folds an identical consecutive call into the last entry and increments only its `Repeats`.
   - Case: a severe halt is posted and the operator resumes. The same Stuck recurs and folds into the same entry with `repeats` one higher.
   - Result: D8 sends no mention inside the window, and D7's "posted" state record keyed on the entry suppresses the post entirely. D10's one chase per entry likewise never chases the recurrence. This is the "new halt after a resume" that F16 meant to cover.
   - The key needs the entry's `repeats` as well as its index.

2. **MEDIUM**: the F17 split makes the held notice fire for an agent holding an unread reply. The runner deliberately treats that state as no hold.
   - Changed text: D11 "the quiet notice and the held notice read a probe that answers only an open question of the strand's own", and the D11 criterion "that it is sent for an agent that holds only an unread reply".
   - Evidence: `internal/shuttleengine/doc.go:58-61` says the reply probe covers a pending reply so that "the reply starts a turn in a live session; the hold sends no notice". It also says the hold ends only after two probes in a row find no reply awaited.
   - Case: an agent asks, ends its turn, and the reply arrives before the bell or hooks deliver it. The new held notice fires in that gap for an agent that is about to resume.
   - The split is sound for the quiet notice, which names the reply. The held notice should keep today's combined probe.

3. **LOW**: F21 option chosen (the orch leaves "no settling action open" to D10 under `unattended`) delays a known severe halt by two chase intervals.
   - Changed text: D7 "under `unattended` it leaves that halt to D10's post".
   - Case: the orch sees no settling action at once. With `beam.halt_chase_min` at 10, D10 first chases the orch, which already knows. The severe post comes one interval later, about 20 minutes after the orch could have relayed it.
   - The review's other option would post at once with a single mention: end D10's chase on the orch's own `[operator] severe halt &lt;slug&gt;` relay.

4. **LOW**: a hold that a blanket resume lifts turns into a severe post under `unattended`.
   - Changed text: D4 "never resumed by the lift" and D10 "a pause covering the run, the orch's hold among them" ends the chase.
   - Case: the operator runs a blanket pause lift. The held run stays halted with `settle: orch` and is no longer covered by a pause. D10 then chases it and, one interval later, posts it as severe.
   - A quiet `held` run becomes a severe post, and nothing says whether the chase that ended counts as the halt's one chase.

Calls made:
- grep of `internal/shedengine` for `Repeats`: to learn whether a re-halt folds into the same history entry, which no file in context showed.
- grep of `internal/shuttleengine` for the reply probe: to learn why the held notice reads an unread reply, since `hold.go` was not in context.
- grep of `internal/shedadapters` for `segment`: to confirm each review segment has its own run directory, so F9's per-segment count over the circling decision files holds. No finding came from it.
