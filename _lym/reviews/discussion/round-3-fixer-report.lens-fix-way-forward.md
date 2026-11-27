```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: I checked whether each refusal, next step and test the fix introduced can be reached from the state it names and leads to progress.

1. **MEDIUM.** Changed text, D5: "for a run owned elsewhere it is `lym adopt` of the run's subject … then that push, or the hand-over below, which leaves the run to its owner."
   - Summary: the hand-over option gives the operator nothing to run, so it is not a reachable way forward.
   - Evidence: the new hand-over paragraph names only "the catch-up's or Finalize's" hand-over. Finalize of a landed run is done, so it does not run again. The catch-up hands over through `pairteardown.HandOverOwnedElsewhere` (`internal/battencli/catchup.go:136`). Round 2's accepted rebuttal says the catch-up is run-scoped and no landing target triggers it.
   - The fix's own D13 change closes the manual route: `lym fabric remove` now refuses while the entry stands.
   - So an operator who wants to leave the run with its owner can only `lym adopt` it here, which is the opposite of what they want. The refusal needs a verb that starts the hand-over, or the option should be dropped.

2. **MEDIUM.** Changed text, D5 diverged case: "the way forward drops the local landing commit … and the run's landing entry goes with no `done` write."
   - Summary: nothing named removes the entry or clears the refused-push record after the operator drops the commit.
   - Evidence: D5's settle rule (":152-154") removes an entry, and clears the parent's refused record, only once "origin's parent already holds" the entry's owed tip. After `git reset --keep origin/&lt;parent&gt;`, the local landing commit is gone and origin never holds that tip.
   - The new hand-over path drops the entry; this path names no actor. The refused-push record also blocks the dobby from starting the pusher (D3, D5), so no pusher pass removes it.
   - The entry then stands for good. The parent target owes a push forever (D5 :125), D3's all-clear never posts, and the new D13 refusal blocks every teardown of the pair.
   - The text has to name who drops the entry and clears the record, such as the pusher or a dobby tick finding the recorded tip absent from the local parent.

3. **LOW.** Changed text, D5: "after a drop by rebase, each standing entry's owed tip is read again as the local parent commit that merges its run."
   - Summary: no new criterion drives this re-read, and nothing says which component makes it or when.
   - Evidence: the added D5 criteria cover the diverged refusal's wording and the hand-over drop, but not the push after a rebase drop. With no actor and no test, a standing newer entry may keep naming a commit that the rebase rewrote, so its tip is never one origin holds and the entry never settles.

Further tool calls:
- `grep` of `internal/battencli/catchup.go` and the repo for hand-over call sites: my context did not show what triggers a hand-over of a landed run.
- `sed` of `internal/battencli/catchup.go:120-145` and `internal/pairteardown/handover.go:195-215`: to read `HandOverOwnedElsewhere`'s contract and how the catch-up wires it in, which my context lacked.
