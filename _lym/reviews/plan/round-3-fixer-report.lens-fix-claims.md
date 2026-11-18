```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the F17 fix added to card 14 holds against the code at HEAD.

**[MEDIUM]** Changed text: card 14, load-turn paragraph ("the shape `Wait`'s liveness read gives a strand a stop removed") and the same phrase in the new record-write paragraph sentence.
Summary: `Wait`'s liveness read gives `OutcomeDied` for a removed strand only when the stop went through this handle's `Run.Stop`. A `StopStrand` stop, which is what the card's stop-race rows drive from `OnEndStrandProcess`, gets a mechanism error instead.
Evidence: in `internal/shuttleengine/wait.go:1484-1492`, an untracked strand returns `OutcomeDone` when the files exist, `OutcomeDied` only `if run.markedStopped()`, and `errStrandNotTracked` otherwise. Lines 1448-1450 scope that answer to "a stopped handle". `markedStopped` is set only by `Run.Stop` (`stop.go:47`). `Runner.StopStrand` → `stopRecorded` (`stop.go:87-102`) writes the record and removes the strand without touching the handle. So for the stop the rows drive, the cited shape is not what `Wait` gives. Returning `OutcomeDied` is still a coherent choice, but the justification states a behaviour the code does not hold. The claim came over from the reviewer's proposed fix.

**[LOW]** Changed text: card 14, record-write paragraph, "A record a stop made terminal before the mark leaves the stop to remove its own strand, so that relaunch makes no reed op."
Summary: the stop does not always remove its strand. A stop whose removal fails leaves a terminal record over a live strand, and the relaunch then returns died with no reed op and the strand still up.
Evidence: `stop.go:39-40`, "a failed removal leaves the terminal record over a live strand". `stopRecorded` writes the record before `removeStrandIfPresent` (`stop.go:95-101`), and a failed `RemoveStrand` returns an error and leaves the strand (`stop.go:152-153`). "Leaves the stop to remove its own strand" holds only when the stop's removal succeeds. A removal still in progress is fine, because the stop's own call finishes it.

Claims that hold:
- `startupStrandRemoved` false keeps `startRetryable` false: `run.go:645-646`, `result.Outcome == OutcomeDied && run.startupStrandRemoved`.
- `start` returns no run, plus the result and an error, when the attempt is not retryable: `run.go:630-631`.
- `Start` drops the result, so the new row's outcome assertion needs `start`, as the row says: `run.go:539-544`.
- A stop's terminal record reads stopped, not done, unless the output files exist; with the files present it stays running and is not terminal: `stop.go:22-27, 92-99`.

Tool calls:
- Read `internal/shuttleengine/wait.go:1440-1519` (`checkLivenessTick`), to check the "shape `Wait`'s liveness read gives" claim. My context held only `doc.go:411`'s summary of it.
