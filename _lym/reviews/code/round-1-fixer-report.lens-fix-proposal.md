```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal lens, challenging the review proposals that the fix carried out.

One finding.

1. **MEDIUM**. Changed text: `protectedParentWayForward` in `internal/fabricengine/pushclassified.go` (F5).
   - **Summary:** the line offers marking the parent `trail` as an alternative to landing by hand, but marking it does nothing for the landing that was refused.
   - **Evidence:** the text reads "mark the parent to land through a pull request … by setting beam.pull_request to trail …; **or** land the work through a pull request by hand; then run `git fetch origin` and `git reset --keep origin/<branch>`". By the time this record is written, Finalize has already ended Done and recorded a landing entry, and nothing re-runs the landing under the new mode.
   - **What goes wrong:** a reader who takes the first branch sets `trail` and then runs the reset. That drops the local landing commit while origin holds none of the work. The next `SettleRefused` finds `WorkUnderOtherCommit` false, so the landing stays. `removedFromParent` then applies on a later pass, and nothing ever lands the work through a pull request.
   - **What the removed advice said:** `protectedParentWayForward` in `landingshed` (commit `aef492c35`) joined the two with `; `, not "or". The mode was for later landings, and the hand landing of this run's work was required ("this run's work is on the local parent only, so merge task branch … by hand").
   - **Source of the error:** the review's `fix` key for F5 words it as "set beam.pull_request to trail … **or** land the work through a pull request by hand", and the fix copied that.
   - **Fix:** require the hand pull request for this landing, and name `trail` as the setting that stops later landings being refused. The reset comes only after that pull request is merged.

The other proposals carried out as written held up against the cases I tried: F1, F3, F7, F8, F9, F10, F14 and F16.

Tool calls: none; the context held everything the lens needed.
