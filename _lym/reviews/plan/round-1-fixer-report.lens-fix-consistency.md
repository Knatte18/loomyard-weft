```yaml
lens: fix-consistency
outcome: reported
```

Task: check, through the fix-consistency lens, that the fixer's plan changes still agree with the sibling cards, rows and cross-references they touch.

**MEDIUM: card 6's starter still counts held commits as owed, so F4's fix applies to only one of the two ahead reads**
- Changed text: card 5's new sentence, "`NewDeps`' `Ahead` counts the records branch only up to `<hold>^` … so a pair ahead only by held commits reads level and its pusher exits keeping the entry."
- Evidence: card 6 says "`starter.start` with `ifAhead` writes the target's ledger entry when the target is ahead and not refused." The dobby tick and `DobbyRuns` start pair pushers through `StartIfAhead`.
- The starter's ahead read was not made hold-aware. Each tick it reads a held-only pair as ahead, writes `FirstOwed` again and restarts the pusher. The entry then reads `Owed` (card 4's new `Entry.Owed`), which reopens the owed period that cards 20 and 21 now exclude.

**MEDIUM: card 5's new `TestRun` row cannot reach the code it claims to pin**
- Changed text: card 5's new row, "a pair ahead only by its held commit, held through a landing entry written through `pushledger.Update`, reading level, exiting after one pass and keeping its entry".
- Evidence: `TestRun` runs over `runFake`, whose `Ahead` is the test's own. The hold-aware read lives in `NewDeps`' `Ahead`, which `TestRun` never builds.
- The row needs a `NewDeps`-built test, such as the tier1 `TestNewDeps_*`, or a stated `runFake` that wraps the real read.

**MEDIUM: card 13's reason for moving its rows contradicts card 12's untouched rows**
- Changed text: card 13 says "The rows that need a filled `ParentPushState` drive it through the real `landingDeps` wiring, so no row stands in for `runpusher` or the ledger."
- Evidence: card 12's `TestFinalize_LockFirstAndFastForward` rows still fill `ParentAheadByOwed` and `LandLocally` by hand. Card 12 itself says `landingDeps` fills those from `fabricengine.AheadByOnly` over `pushledger.OwedTips` and from `landLocally`.
- Under the same PATTERN-test-stand-ins reading the fix applied to card 13, those rows are the same kind of stand-in.

**LOW: the overview's trailer decision still states the old rule**
- Changed text: card 11 step 4, which now refuses an unrecorded landing commit "only while `RunPastFinalize` reads that run still at Finalize" and otherwise re-records it.
- Evidence: 00-overview.md, trailer decision, Rationale: "D5 refuses a parent push whose tip carries a landing commit no entry records".
- That rationale also omits the trailer's new use of rebuilding the lost ledger.

**LOW: card 10's ImpactSummary is stale**
- Changed text: card 10 now edits `PushPairAnchored` so that a pair push with a lost ledger sends no records.
- Evidence: card 10's ImpactSummary still reads "Adds one failure class, one trailer key and six parent-branch helpers; `FailedPushWayForward` gains one case…", which misses the change to every task pair push.

**LOW: card 11's widened landing `Ahead` has no test**
- Changed text: card 11, "`NewDeps` widens the landing target's `Ahead` to the same two reads".
- Evidence: card 11 lists `TestPusherDeps` under Edit but names no row for that `Ahead`. Card 6's `TestPusherDeps` row pins only "the landing target's level read" from the ledger.

**LOW: card 13 does not list the failure class its test writes**
- Changed text: card 13's test writes "the prime code branch's `held_back` record".
- Evidence: card 13's Uses lists `internal/fabricengine#RecordPushFailure` but not `plan:internal/fabriclayout#PushFailureHeldBack`, which card 10 creates.

Further tool calls: none. Everything I checked was already in my context.
