---
round: 1
focus:
  - "internal/creelengine/deliver.go `wakeHookStrand`: settle whether the new `woken` hold (`!mark.at.IsZero() && !idleness.TurnEndedAt.After(mark.at)`) holds the bell for the goal's burst case, an acknowledged ring with a queued turn, and for no other case where the bell is still needed. Check a zero `TurnEndedAt`, a ring from an earlier set long past the window, and a ring answered through a claim or a running-turn probe."
  - "The goal expects a strand whose wake is queued and whose waiter has read its mail to get no reminder line. Settle whether the diff reaches the pane-path reminder line the goal quotes, or only the hook-strand bell, and whether the decision record's wording makes that difference matter for the diff's scope."
  - "internal/creelengine/deliver.go: settle whether the branch order after the `belled` return (hold within the window, then the grace with a waiter live or recently ended) keeps every earlier behavior outside the new hold: an unanswered ring past the window, a released or cancelled ring, and the grace."
  - "internal/creelengine/deliver_test.go `TestScheduler_HookWake`: settle whether each new row covers a behavior the diff adds and whether an existing row already covers it; check that the rows exercise a turn end read after the ring through the same reading path production uses."
  - "internal/creelengine/doc.go and the `wakeHookStrand` and `ringAnswerWindow` comments: settle whether each claim, including the added Bound line, matches what the code does, and whether `_lym/landing/summary.md` claims anything the diff does not bear out."
  - "Commit messages in the range: settle whether any trailer or claim departs from the decision record's Goal."
---

The diff is small: one function in `deliver.go`, two test rows, and the matching `doc.go` text.
The decision record holds only the Goal, so the measuring stick is the Goal section; its other sections are empty.
