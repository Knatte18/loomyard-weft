```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that each set-premise finding's fix covers every member of its set.

Unchecked: none.

- **F1, writers and removers of the dobby-stop marker:** I checked this from the grep of `DobbyStopMarker` and the `beamawait.go` read already in context, with no new call. The only writer is `landChecks`. The removers are `enter`'s lifted-hold branch, `accept` and `refuse`, plus the planned `recheckStanding`. The changed card 10 text names all of them, and it also covers the marker that stays after a `refuse` whose restart failed.
- **F2, places that quote the resolver's `Halt` with `lym loom resume`:** No other beam verb card quotes it. Card 11 names `lym loom resume` only for Finalize's own halts, which is correct there. Card 18's resume is about a halted Finalize, also correct.
- **F3, GitHub reads in step 6:** Both `PullRequestSide` and `ReadMergeState` are covered.
- **F4, satisfied conclusions:** `success`, `neutral` and `skipped` are all named. Card 14's status reads the cause through the same `ReadMergeState`, so it has no separate rule to update.
- **F5, other uses of "done" for `judgeAwaitingRun`:** Cards 13 to 15 have no done-run row or branch.
- **F6, the caller list in `pullrequest/doc.go`:** The list appears only in line 1, which card 3 now rewrites. Line 4 describes record fields, not callers, so it is outside this set.

no findings

Calls made:
- One grep over `internal/pullrequest/doc.go` and plan cards 11–18 for the F6 caller sentences, the F2 `Halt`/`loom resume` quotes and the F5 done-run rows. My context had not searched cards 11, 13, 14, 16, 17 and 18 for these terms.
