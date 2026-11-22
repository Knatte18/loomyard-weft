```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the ways forward the fix added to the decision record (fix-way-forward lens).

**MEDIUM — D6, prime pusher: "it replays nothing while one of those reads is open or failed"**
The fix names no next step after a failed takeover read.
It does not say whether the failure counts as owed, so the pusher retries it in D5's backoff, or as a refusal, so it writes a refused-push record and exits.
Under the refusal reading, D5's new prime-wide stop holds every batten run's pushes until an operator clears the record, and the record names no way forward.
Under neither reading does the text show that the pusher reaches a later pass that makes progress.
Evidence: D6 bullet 2, and D5 bullet 2 ("Records are keyed by repo side and branch … holds every batten run's records").

**MEDIUM — D6, prime pusher: "That run's unpushed prime commits go under an owner archive tag"**
This names a step that no existing site carries out, and the record names no new site for it.
`fabricengine.HandOverTips` "archives the pair's local code and records tips that origin lacks" (`internal/fabricengine/handover.go:58`).
For a batten run, the pair is the task pair, not the prime's records branch.
The only change site the bullet names is the `runSlug != ""` gate of `retryWeftPush`.
The record also leaves open what happens when the archive-tag push fails, for example with origin unreachable.
It does not say whether the replay then keeps that run's commits or waits, so "never replayed" has no path that guarantees it.

**MEDIUM — D6, prime pusher: "hands each taken-over run over … The pusher then goes on with the other runs"**
The record does not say what happens when `HandOver` fails for one run.
`HandOver` returns that step's error, and "The next rejected push then hands over again" (`internal/pairteardown/handover.go:95`).
The fix does not say whether the replay of the other runs goes on, or whether the pusher stops, owes or refuses.
So the other runs' commits have no stated way to origin while one hand-over keeps failing.

**LOW — D5 dobby-tick bullet and D3 start conditions, together with D6 bullet 1 ("The pusher then exits")**
After a loom hand-over with `InsidePair` true, the pair stays standing and ahead of origin.
No refused-push record stands for it and no pause covers it.
So every dobby tick and exit re-take restarts its pusher, which meets the takeover again, until the parent's dobby removes the pair.
That is a loop without progress, because the start conditions never exclude a pair that has been handed over.
Evidence: the D5 dobby bullet lists only the lock, refused-record and pause conditions, and `handover.go` returns "its pair is left standing for its parent's dobby to remove".

I made no further tool calls.
