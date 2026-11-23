```yaml
lens: way-forward
outcome: reported
```

Task: way-forward lens review of the plan's refusal, stop and recovery ends.

**1. MEDIUM — Card 10 (`teardown-pushes-first`), D5: the push-and-check loop never ends on a code branch origin ended and a fetch pruned.**
- Card 10 removes the pair only once `Level` reads true. On a false read it releases the lock and pushes again.
- Card 2's `TipsOnOrigin` reads "a branch with no upstream or no tracking ref" as not held.
- The push already skips a branch that is gone from origin with its entry ended, and returns nil for it. See `internal/fabricengine/pushclassified.go`, `pushJob.checkOrigin`, `case view.ended: return false, nil`.
- So a landed run whose code branch origin deleted, with the tracking ref pruned, pushes "successfully" forever and `Level` stays false.
- No Stuck is raised and no way forward is named, so the row neither progresses nor halts.
- D5 itself says such a branch "reads as level and is left to its teardown". Card 2 applies that reading to `RunCommitsAhead` but not to `TipsOnOrigin`.
- Fix: have `TipsOnOrigin` read a branch whose upstream is set and whose tracking ref is absent as held, the same way `RunCommitsAhead` does, and add a `TestTipsOnOrigin` row for it. Otherwise, bound the loop with a Stuck that names the branch.

**2. MEDIUM — Card 10, D5: the records side of an ended pair halts the teardown with no next step that can succeed.**
- When the records branch is gone from origin with its entry ended, the push returns `recordsSideNotLanded`'s "left to its teardown" error. It is unclassified and not owed (`internal/fabricengine/pushanchored.go`, `recordsSideNotLanded`).
- Card 10 makes any failed `Push` Stuck "naming the push", with the pair kept.
- A resume re-runs the row and meets the same error, so the teardown that error defers to never runs.
- Fix: treat that error, or an ended or gone branch, as nothing left to push, and let removal proceed. Add a `TestWorktreeTeardown_PushesWithThePrimeLockReleased` row for it.

**3. LOW — Card 10: its Stuck reasons name no next step.**
- "Stuck naming the push" does not say whether the reason carries the push error's way forward. A `*PushFailedError` carries it in `Error()`; a hand-over or an unclassified failure carries none.
- Fix: state that the reason includes the push error text and names the resume, `lym batten run &lt;slug&gt;`, once the record's way forward is done.
- The card also leaves `RunPush.InsidePair` unset, so a takeover during the teardown push removes the pair through `HandOver`. That stop should be stated as terminal: the run is handed over and needs no resume here.

**4. LOW — Card 6, D7: a run owned elsewhere gets a spawn, a reconcile fetch and a notice on every transition.**
- The guard's owned-elsewhere stop writes no pause and no hand-over mark (`internal/loomcli/prepush.go`, `prePushCheck.guard`).
- So nothing blocks the next start: under D1 each local transition starts a new pusher, which fetches and posts "stopped before its push" again.
- The way forward (`lym adopt`, then the catch-up's hand-over) does progress, so this is cost and notice noise, not a dead end.
- Fix: card 5's start conditions could skip a pair whose last guard stop was owned-elsewhere until the catch-up runs. Or card 6 could note the repetition as accepted.

Extra calls: one grep of landing, fabricengine, githubclient and loomcli for a remote-branch delete, to learn whether landing deletes origin's task branch; my context lacked the landing code. It found no delete at landing itself, so finding 1 rests on a branch deleted outside the landing (such as GitHub deleting a merged pull request's head branch) and later pruned by a fetch.
