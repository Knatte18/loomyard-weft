# Card 9 — records-hold

**Edit:**
- `internal/fabriclayout#PairPush`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine#coalescePushSides`
- `internal/fabricengine_test#TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`
- `internal/pairteardown#RunPush`
- `internal/pairteardown#PushRun`
- `internal/pairteardown/doc.go`
- `internal/beamcli#hubLandDeps`
- `internal/beamcli#landDeps`
- `internal/beamcli#prOpen`
- `internal/beamcli#prUpdate`
- `internal/beamcli/doc.go`
- `internal/fabricengine#Topology.removePair`
- `internal/fabricengine#remoteTeardownJob`
- `internal/fabricengine#runRemoteTeardown`
- `internal/fabricengine#archiveWeftTip`

**Create:**
- `internal/fabricengine#recordsPushCeiling`
- `internal/pairteardown_test#TestPushRun_StopsShortOfAHeldRecordsCommit`
- `internal/beamcli#TestHubLandDeps_PushBeamRefusesWhileTheBeamHoldsLandings`

**Uses:**
- `internal/fabriclayout/doc.go`
- `internal/pushledger#HeldRecordsCommit`
- `internal/pushledger#OwedTips`
- `internal/pushledger#RecordsKey`
- `internal/pushledger#Update`

**Intent:** D5's hold: while a run's landing entry stands, and past a dropped landing until the pair goes, no push of the run's pair records sends the records commit the entry holds, nor any commit after it;
and no ordinary push of a beam pair sends a landing commit its parent target has not yet judged.

`fabriclayout.PairPush` gains `CodeCeiling`, a code commit.
The records hold is read by `recordsPushCeiling`, the one step both pair push paths call before their records push: `PushPairAnchored`, and `coalescePushSides`, which `lym fabric push`'s detached bypass reaches through `DetachedPush` and `CoalescePushRequest`.
So every push of a task pair meets it: the pair's pusher, the catch-up, PR-Review's record commit push, the hand-over's owed push, `lym fabric push` and its detached bypass alike, and a removal's records push below.
For a task pair, never the prime, whose records every batten run shares and D5 does not hold, it reads `pushledger.HeldRecordsCommit` for the pair's slug, which covers a standing landing and a dropped landing's `HeldRecords`.
With a hold it pushes the records branch only up to the hold's first parent, `<hold>^`, and reads the records side as landed once origin holds that commit;
it skips the records push when origin already holds it.
Card 10 adds the read of a landing the ledger lost.
With `CodeCeiling` set, it pushes the code branch only up to that commit, and reads the code side as landed once origin holds it.

A removal's records push meets the same hold.
`Topology.removePair` reads the slug's hold through `recordsPushCeiling` before its first mutation and carries it on its `remoteTeardownJob`,
since the pair's ledger entry goes as soon as `Remove` returns and the teardown child runs detached.
`runRemoteTeardown` passes it to `archiveWeftTip`, which then pushes the records branch only up to `<hold>^` and makes the archive tag at `<hold>^` too, so no branch or tag pushed to origin reaches the held commit.

`pairteardown.RunPush` gains `CodeCeiling` and `BackoffSleep`.
`PushRun` copies `BackoffSleep` into the push's `SyncOptions.BackoffSleep`; nil, as every production caller leaves it, sleeps the production backoff, and a test passes one that returns at once (PATTERN-test-speed).
`PushRun` passes a set `RunPush.CodeCeiling` through to `PairPush.CodeCeiling`, which the parent pusher's beam push sets to the tip it judged (card 11).
With the field empty, `PushRun` derives the ceiling when `l` is a beam pair whose own target holds landings (`pushledger.OwedTips` of its `RecordsKey`): the first parent of the oldest landing's tip,
so the beam's own commits before its first landing go out and no landing commit reaches origin before the parent pusher's owner reads.
Card 10 widens the ceiling to a landing commit no entry records.
A run with no landing and no held commit, and a pair holding no landing, push as today.
Bound: `lym fabric push` in a beam's worktree is the operator's verb and the way forward D5 names for a refused beam parent, so it sets no `CodeCeiling` and pushes the beam's code side whole, while its records side meets the hold like every push.

`beamcli#hubLandDeps`' `PushBeam` refuses while the beam's own ledger key holds landings, before it pushes, since a capped push would leave origin's beam branch below the head `lym beam pr` records and opens the pull request on.
Its way forward: wait for the beam's parent push, or run the step its refused-push record names, then rerun `lym beam pr`.
`internal/beamcli/doc.go` gains one line under `lym beam pr`: `open` and `update` refuse while the beam carries a landing commit its parent push has not settled.
The doc comments of `landDeps.PushBeam`, `prOpen` and `prUpdate`, which say they push the beam branch, add that the push refuses while the beam holds landings.
`internal/pairteardown/doc.go`'s hand-over section says a push of a run's pair stops short of the records commit a landing holds, and that a beam push holding landings stops below the oldest landing unless its caller sets the ceiling.

Tests.
`TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`, tier1, gains rows over a pair fixture whose hold is a landing entry written through `pushledger.Update`: with the hold on the second of three unpushed records commits, origin's records branch ends at the first and the code side is pushed;
with the hold already on origin the records push is skipped; with `CodeCeiling` naming the first of two unpushed code commits, origin's code branch ends at it.
`TestPushRun_StopsShortOfAHeldRecordsCommit`, tier1, over a pair fixture with a landing entry written through `pushledger.Update`,
pins that `PushRun` of the pair leaves the held commit and the one after it off origin, that `PushRun` of the prime for the same slug pushes the prime's records whole,
that with the landing removed and the commit left as the pair entry's `HeldRecords` the pair push still stops short, and that once that entry is removed the next `PushRun` sends both;
that a beam pair holding a landing pushes its code branch only up to the landing commit's first parent;
that the same beam pair pushed with `RunPush.CodeCeiling` set to the landing tip ends with origin's code branch at that tip and its records branch pushed first;
and that `lym fabric push` run in the held pair's task worktree, and its detached form through `DetachedPush`, each leave origin's records branch without the held commit.
`TestHubLandDeps_PushBeamRefusesWhileTheBeamHoldsLandings`, tier1, adding about two seconds to one tier1 run, over a hub fixture with a beam pair and a landing entry on its key written through `pushledger.Update`, pins that `hubLandDeps`' `PushBeam` refuses with the way forward above and leaves origin's beam branch where it was, and that with the entry removed it pushes and returns the head origin then holds.

**ImpactSummary:** Adds one field to the pair push request and two to the run push, and makes every task pair push read the ledger's hold; a pair holding nothing, and the prime, push as before, and `lym beam pr` refuses while its beam holds landings.
