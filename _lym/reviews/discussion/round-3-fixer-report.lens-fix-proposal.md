```yaml
lens: fix-proposal
outcome: reported
```

Proposals challenged: F1 (option a, push the parent before the records and the board `done`), F2 (the `--take-hold` flag), F3 (drop D10), F4 (Open risks empty), F5 (records side first in `PushPairAnchored`), F6 (remove paths delete the record), F7 (bounded admission, and the upgrade hold only when an intent was written).

1. **MEDIUM**: the F1 proposal breaks a landing whose parent push is skipped.
   - Changed text: D1's new "A `landing` run is past its parent-side merge when origin's parent branch, as this machine last fetched it, holds the task branch's work…", and D4's "After the merge into the local parent branch, the Finalize pushes the parent branch, then commits and pushes its transition, then writes the board `done`".
   - The case: `Deps.PushSkipped` suppresses the parent push (`internal/landingshed/deps.go:56`, `finalize.go:776,787`). Today `markTaskDone` runs ahead of the push so that "a skipped or failed push still leaves the task done" (`finalize.go:708-709`).
   - Under the new order, a landing with the push skipped never gets origin's parent branch holding the work. D1 then reads the run as not past its merge forever, and the board `done` is written "only after" a push that never happens.
   - The record does not say how a skipped push gets past the merge or reaches `done`.

2. **MEDIUM**: the F7 proposal lets queued tasks start after the upgrade with no adopt check.
   - Changed text: D7's "A first read whose entries were all dropped writes no hold, so every upgrade hold names subjects to re-run."
   - The case: a pre-upgrade adopt ran its pause step and was interrupted. Its subjects hold only released tasks with no batten record, plus landed runs.
   - D7 drops every such entry ("A released task's entry with no batten record is dropped"). Under the new rule no hold is written, so after the upgrade the dobby claims those queued tasks before any adopt check is green.
   - That contradicts the operator decision D7's rationale cites: "nothing happens in the hub until adopt is done".
   - The old text wrote the hold for any standing entry for exactly this reason: "so no queued task starts". A sound fix keeps the hold whenever a dropped entry belonged to a released `queued` task, and names that task's subject in the hold.

3. **LOW**: under F6, "whose pair is gone" is read on one machine.
   - Changed text: D1's new "A `cancelled` batten record whose pair is gone … counts as no batten record, so a released task with one reads `queued`."
   - The case: a cancel with keep leaves its pair on the owner machine. On another machine with no local pair (for example after D6 step 4 removed it), the record reads as no record, so the run reads `queued` there while it reads `cancelled` on the owner.
   - This breaks D1's rationale: "gives every machine the same state from the same pushed bytes".
   - The condition should read the pair's absence from origin, such as its task branch deleted on origin. `--remove` deletes that branch; a kept cancel does not.

4. **LOW**: under F5, the fabric push verb's code side now waits on an unrelated records conflict.
   - Changed text: D2's "It sends the code side only once the records side has landed, and none when the records side fails or is rejected, so the order holds for every caller of a pair push".
   - The case: an operator's fabric push verb (`internal/fabriccli/weft_verbs.go:286`) on a pair whose records side is rejected over other runs' commits on a shared records branch, persisting past the replay retries. Today the code side still goes ("The records side is attempted even when the code side failed", `pushanchored.go:128`, and the code side is pushed first). Under the change, the operator's code commits stay local with no way forward named.
   - The record states no way forward for a non-transition caller whose code side is withheld.

No findings against the F2, F3 or F4 proposals.

Tool calls:
- I grepped `internal/battencli` and `internal/landingshed` for `ClearClaim` and `PushSkipped`. My context lacked the push-skip dependency and where it is used, which the F1 case rests on.
