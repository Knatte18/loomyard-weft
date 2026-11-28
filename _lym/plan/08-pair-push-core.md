# Card 8 — pair-push-core

**Create:**
- `plan:internal/fabriccore#SeamPushSide` -> `const SeamPushSide = "push_side"`
- `plan:internal/fabriccore#RecordsLandingFacts` -> `type RecordsLandingFacts struct`
- `plan:internal/fabriccore#RecordsSideLanding` -> `func RecordsSideLanding(snapshot Snapshot) Answer`
- `plan:internal/fabriccore#RecordsLandingVerdict` -> `func RecordsLandingVerdict(snapshot Snapshot) fabriclayout.PushFailureClass`
- `plan:internal/fabriccore#BranchReading` -> `type BranchReading struct`
- `plan:internal/fabriccore#BranchAhead` -> `func BranchAhead(b BranchReading) bool`
- `plan:internal/fabriccore#BranchHeldOnOrigin` -> `func BranchHeldOnOrigin(b BranchReading) bool`
- `plan:internal/fabriccore#RunAhead` -> `func RunAhead(prime bool, records, code BranchReading) bool`
- `plan:internal/fabriccore#TipsHeldOnOrigin` -> `func TipsHeldOnOrigin(prime bool, records, code BranchReading) bool`
- `plan:internal/fabriccore_test#TestRecordsSideLanding` -> `func TestRecordsSideLanding(t *testing.T)`
- `plan:internal/fabriccore_test#TestRunAhead` -> `func TestRunAhead(t *testing.T)`

**Edit:**
- `internal/fabriccore#Snapshot`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine#recordsSideNotLanded`
- `internal/fabricengine#readLocalBranch`
- `internal/fabricengine#localBranch`
- `internal/fabricengine#unpushedUnder`
- `internal/fabricengine#RunCommitsAhead`
- `internal/fabricengine#TipsOnOrigin`
- `internal/fabricengine_test#TestRunCommitsAhead`
- `internal/fabricengine_test#TestTipsOnOrigin`
- `internal/fabricengine_test#TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`

**Delete:**
- `internal/fabricengine#localBranch.ahead`
- `internal/fabricengine#localBranch.lacks`
- `internal/fabricengine#localBranch.heldOnOrigin`

**Extends:**
- `internal/fabriccore/doc.go`

**Uses:**
- `internal/fabricengine#pushClassified`
- `internal/fabricengine#runRule`
- `internal/fabricengine#recordOwedRecordsPush`
- `internal/fabricengine#teardownLockOwedError`

**Intent:**
The pair push's own loop and the pusher's local reads become core rules.

Records side of `PushPairAnchored`:
- Today `pushRecordsSide` pushes the records side, re-reads whether it is still unpushed under its ceiling, and pushes again after each backoff delay while origin holds the branch.
  `RecordsSideLanding` is that loop: it answers `ActionSeamCall` with `SeamPushSide`, whose runner is one `pushClassified` of the records side, then re-reads `RecordsLandingFacts`, then `ActionWait` and the seam call again while the side is unpushed and still on origin.
  `Snapshot` gains `RecordsLanding RecordsLandingFacts`: the ceiling, whether the side is unpushed under it, whether a deferred record was written since the push started, whether origin still holds the branch, and whether that lookup failed in transport.
- `RecordsLandingVerdict` is `recordsSideNotLanded`'s verdict over those facts, in today's order:
  a deferred record since the start is that deferral;
  a transport failure of the lookup is an owed `transport` record;
  a branch still on origin once the delays are spent is an owed `deferred` record;
  a branch gone from origin is its entry ended, `ErrEntryEnded`, no record;
  any other failed lookup is returned unclassified, as today.
- The engine keeps the teardown lock, the push lock through `coalesceFlagged`, the pending flags and the record writes (`recordOwedRecordsPush`, `teardownLockOwedError`); `recordsSideNotLanded` reads the facts and returns the error the verdict names, so `PushPairAnchored`'s errors are unchanged.

Pusher reads:
- `BranchReading` holds a branch's tip (empty for no commits), its upstream state, and the ancestry answers the reader takes: whether the tracking ref lacks a commit of the tip, whether it holds the tip, and whether it lacks the ceiling.
  `readLocalBranch` reads them with today's git calls, and only the ones the caller's question needs.
- `BranchAhead`, `BranchHeldOnOrigin` and the ceiling's lack are today's three methods' rules: no commits is never ahead and always held; an unset upstream is ahead with a commit and never held; a pruned tracking ref reads level and held.
- `RunAhead` is `RunCommitsAhead`'s rule: the prime is ahead when its records branch is; a pair when either branch is, its records branch counted only up to the held ceiling.
  `TipsHeldOnOrigin` is `TipsOnOrigin`'s: the records tip held, and for a pair the code tip too.

Tests (the commit message lists them):
- `TestRecordsSideLanding` (core, tier0): a landed side ends; an unpushed side still on origin waits once per delay and ends with the deferred verdict; a deferral since the start ends at once; a transport lookup failure; any other lookup failure returned unclassified; a gone branch ends as entry ended.
- `TestRunAhead` (core, tier0) takes `TestRunCommitsAhead`'s and `TestTipsOnOrigin`'s rows as readings.
- `TestRunCommitsAhead` keeps `PairRecordsAheadOnlyByTheHeldRefsCommitReadsLevel`, `RecordsBranchWithItsTrackingRefDeletedReadsLevel` and `PairCodeBranchWithNoUpstreamReadsAhead`, which pin the reads of the ceiling, the pruned ref and the unset upstream; it deletes the rest.
  `TestTipsOnOrigin` keeps `BothTipsPushed` and `CodeBranchWithItsTrackingRefDeletedReadsHeld`; it deletes the rest.
- `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` keeps its interleaving rows (`PrimeRecordsCommitDuringThePushIsPushedAgain`, the `PrimeRecordsPushOwed_*` rows, the deferral and owner-guard rows, the held and capped rows) and its first section's mutation assertion, which it widens to the exact `mutations` array the push prints, entry for entry, against the array today's code prints (D4, D5).
  It deletes the inline section "a remote that always declines gives a bare rejection after the whole backoff and one `other` record", held by `TestPushSide`'s rejection loop row and `TestPushFailureClasses`' kept `Code` row.
The core tests add about 1 second to one tier0 run.

**ImpactSummary:** PushPairAnchored, RunCommitsAhead and TipsOnOrigin keep their signatures for their callers in tendercli, runpusher, beamcli, battencli, loomcli and pairteardown; only private readers change.
