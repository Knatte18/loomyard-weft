```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked the change set against the sibling cards, the decision record and the overview.

1. **MEDIUM.** The new restart rule contradicts D7 for errors that happen before `Apply`.
   - Changed text: card 17 says "Every other such error restarts no dobby … an error from the pass wait, the parent-lock acquisition, the dobby end or the log-sink close … each releases the parent lock when held, calls `Close` and returns the error naming the re-run". The new card 15 restore-failure sentence and the overview line 43 rewrite say the same.
   - Problem: these failures come before the marker commit, and D7 says such a failure removes the move record and restarts the dobbies. The fix leaves the record standing and restarts nothing, and the decision record has no `Added after Discussion` entry for that.
   - Evidence: `_lyx/discussion/decision-record.md:147` says "A failure before the commit, or before the integration moves the branch, restores the working tree, the index, `A/.lyx` and the marker, rewires `A`, removes the move record, restarts the dobbies step 2 ended and releases the weft write lock and the parent lock."
   - Either the cards remove the record and restart the dobbies on these paths, which a wait, lock or end error allows because nothing has moved yet, or the departure gets recorded through the parent. Records written that way exist at decision-record.md:237 and :243.

2. **MEDIUM.** `UpstreamRunUnfinished` relies on a rule card 17 never states.
   - Changed text: card 15 says it "judges the status unfinished as the checked-out refusal of card 17 does".
   - Problem: card 17's refusal only says "its run status names an unfinished run". It never says which status fields mean unfinished, and never says which status file it reads: the pair's own run, or the prime's batten status of the slug. So the engine reader points at a rule nobody wrote down, and the two readings can come out different.
   - Evidence: card 17 line 44 as changed. The code reads two separate statuses, `shedrun.StatusFile(pairLocation, shedrun.SelfRunID)` and `shedrun.StatusFile(primeLocation, slug)` (`internal/fabricengine/replay.go:255,283`). The new reader covers only the pair's own run.

3. **LOW.** The new card 15 restore-failure test row leaves the pair in a state its own named re-run refuses.
   - Changed text: card 15 says "a failure before the move commit whose restore also fails, the hook first making a non-empty `A/.lyx` so the rename back of `B/.lyx` fails, returns `landed` false with the record standing and an error naming `lyx fabric anchor <slug> --set sub`".
   - Problem: afterwards both `A/.lyx` and `B/.lyx` exist. Card 15's not-landed re-run "refuses naming both when both exist". So the re-run the error names hits that refusal, not the recovery the new `Apply` sentence promises ("which recovers a not-landed record by resetting the move's paths to `HEAD`").
   - Evidence: the card 15 Intent sentence on not-landed re-runs, and the row "a not-landed re-run with both `A/.lyx` and `B/.lyx` present refuses naming both".

Tool calls:
- Grepped cards 16 and 19 for record removal and restart wording, to find twins of the F3 and F4 changes. My context had only cards 13–15, 17 and 18.
- Grepped `decision-record.md` for D7's refusal and failure-path wording, which my context lacked, to check the F1 and F4 changes against the decision.
