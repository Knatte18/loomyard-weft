```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F5) and build cases where a proposed change is wrong.

Proposals challenged: F1 (third slug source: task branches), F2 (bullet for an open own fix task), F3 (tries file, hand-adopt-only reset), F4 (`awaiting` read as halted), F5 (D3 criterion reworded).

1. **MEDIUM**: F4 adds `awaiting` to the halted states in D6's removal clause and in D7's `halted` state.
   - **Changed text:** D6 now reads "or halted, its entry's run status having the state `blocked`, `failed` or `awaiting`". D7 `halted` now reads "state is `blocked`, `failed` or `awaiting`, which behaves like blocked wherever a halt is observed".
   - **Problem:** An `awaiting` run is not always stopped. With a review escalation pending, the driver goes on through the arbiter. D6 would then remove a fix task the arbiter is still settling, and D7 would show it as `halted` while it is moving.
   - **Evidence:** `decideDrive` in `internal/sheddriver/driverules.go` has the row `state == awaiting && facts.EscalationPending` → `driveActionArbiter`, "an awaiting review escalation: the arbiter decides". Only the other two `awaiting` rows hand back.
   - **Concrete case:** A hand `lym adopt` runs while the subject's fix task stands `awaiting` with an escalation pending, and its check is red. D6 cancels and removes the task, discarding the arbiter's pending decision, and files a new `-t<N+1>`.
   - **Suggested change:** Count `awaiting` as halted only when no escalation is pending. Otherwise treat it like `running`, which falls under F2's new bullet.

2. **LOW**: F3's tries file, in the changed text "A filing adds one to it before D8's piece files the task, so a filing that dies midway still counts".
   - **Problem:** A filing that fails at its first board write also adds one, though no task exists. An example is the create refused because origin cannot be read. Such failures spend tries without any fix task being filed.
   - **Effect:** After `adopt_fix_tries` of them, the notice says the tries ran out and points at the failing tiers. It is false: no fix was ever tried.
   - **Suggested change:** Name the order as after the create write and before the release write. A filing that dies between the two still counts, and one refused at the create does not.

3. **LOW**: F3's reset rule, in the D5 green-end bullet "resets the fix-try count", which the fix removed.
   - **Problem:** The green end no longer resets the count. A check started after a green end, without a hand adopt, inherits the old count.
   - **Evidence:** D8 says a hand removal of a fix task runs its `adopt-check` action. A green end whose removal failed tells the operator to remove the task by hand.
   - **Concrete case:** The count stands at the limit, the check goes green, and the green end's removal fails. The operator removes the task by hand, which starts a check. If that check is red, it files nothing and says the tries ran out, though the last check was green.
   - **Suggested change:** Keep the green-end reset beside the hand-adopt reset, and reword "Only a hand `lym adopt` resets it" to match.

No case found against F1, F2 or F5.

No further tool calls were made.
