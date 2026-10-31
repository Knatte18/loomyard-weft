```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens: one LOW finding and no BLOCKING or MEDIUM.

**LOW: the seed-read refusal in `review.go` has no way forward.**
- Changed text: `internal/loomcli/decision.go:847`, the seed-read refusal in `preRun`. It now ends with `; way forward: restore the run's seed, or begin the run again with `lym loom start``.
- Evidence: `internal/loomcli/review.go:59` is the same refusal for the review verbs: `loom: %s %s: read the run's seed: %w`. It was left without a way forward.
- Caveat: F5 only named `decision.go`, so this twin may be out of the review's scope. A reader of the two refusals would still notice the mismatch.

Checked and consistent:
- `decisionParent` and `decisionReceivers` have one production call each (`decision.go:869` and `decision.go:907`), both updated to the new signatures. The only other callers are the new integration tests.
- The F7 refusal names `lym beam start &lt;slug&gt;`. The existing `lym beam start` way forward (`internal/fabriccli/anchor.go:29`) carries no slug, and this is not a conflict.
- The F4 doc addition to `ProjectRecordHeld` agrees with the `checkRunLocks` doc and the `ProjectRecordLockedError` text.

Further tool calls (1):
- One `grep` for the callers of `decisionParent`, `decisionReceivers` and the seed-read message. My context held only the edited lines, not the rest of the repository.
