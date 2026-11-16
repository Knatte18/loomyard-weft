```yaml
lens: fix-proposal
outcome: reported
```

Task: test each review proposal the fixer carried out as proposed, and build the case where it is wrong.

**1. MEDIUM. Card 14, F8's reset-pending marker** (the step 2 rewrite and the new bullet: "writes the reset-pending marker … removes it once the commit lands")
- **Summary:** a marker that outlives its reset reopens the unbounded commit F8 was meant to close.
- **Case:** a reset's commit lands, then the process dies before it removes the marker.
  Afterwards the state is absent and the marker is still there.
  Later a Webster run or a plan review leaves uncommitted plan edits, and someone runs `lym weaver reset`.
  Step 2 finds the marker and `RecordPending` true, so it commits those unrelated edits as a finished reset.
  If nothing is pending, step 2 refuses with "no Weaver run has begun" and keeps the marker forever.
- **Evidence:** step 2 only removes the marker when it commits.
  No card clears it at a run's begin.
  The new bullet's claim that step 2 "commits only what a reset this marker records left behind" does not hold for a stale marker.
- **A sound version:** step 2 removes a marker it finds with nothing pending, and the first batch begin removes any marker.

**2. LOW. Card 9's dropped `TestClassifyAttempt`, merged into card 12's TestRun** (F10)
- **Summary:** the merge drops one verdict the old test held: a reed wait error classified `lost`.
- **Evidence:** card 9 still lists "a wait error from reed" among the `lost` causes.
  TestRun's new row covers only "a machinery loss, an `OutcomeDied` and an `OutcomeAPIError` alike".
  F10's list of rows to keep left this one out, and the fixer carried that list over unchanged.

The other proposals carried out as proposed hold:
- **F2:** `configengine.CategoryGuard` exists at `internal/configengine/layers.go:64`.
- **F11:** card 6 says a nil `Setting` answers `webster`, so `NewTracer` calling `Active` with a nil `Setting` is legal.
- **F18:** `planwrite.go:150` hands `archiveDir/webster` as `dest`, so joining the slot onto `dest`'s parent is right.
- **F3, F5, F6, F9, F12, F14 to F17:** no breaking case found.

Outside this lens: card 11's `planVerifySites` entry names "the file that holds `verifytree.Verify` as `RunDeps.Verify`'s production value". Card 11 says a nil `Verify` takes the production value, but card 12's `Run` refuses nil seams. If that default lives in card 12's `Run` file, card 11's tripwire entry fails at card 11.
