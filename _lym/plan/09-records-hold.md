# Card 9 — records-hold

**Edit:**
- `internal/fabriclayout#PairPush`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine_test#TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`
- `internal/pairteardown#PushRun`

**Create:**
- `plan:internal/pairteardown_test#TestPushRun_StopsShortOfAHeldRecordsCommit` -> `func TestPushRun_StopsShortOfAHeldRecordsCommit(t *testing.T)`

**Uses:**
- `internal/fabriclayout/doc.go`
- `internal/fabricengine/doc.go`
- `internal/pairteardown/doc.go`
- `plan:internal/pushledger#HeldRecordsCommit`
- `plan:internal/pushledger#OwedTips`
- `plan:internal/pushledger#RecordsKey`
- `plan:internal/pushledger#Update`

**Intent:** D5's hold: while a run's landing entry stands, and past a dropped landing until the pair goes, no push of the run's pair records sends the records commit the entry holds, nor any commit after it;
and no ordinary push of a beam pair sends a landing commit its parent target has not yet judged.

`fabriclayout.PairPush` gains `RecordsHold`, a records commit, and `CodeCeiling`, a code commit.
With `RecordsHold` set, `PushPairAnchored` pushes the records branch only up to the hold's first parent, `<hold>^`, and reads the records side as landed once origin holds that commit;
it skips the records push when origin already holds it.
With `CodeCeiling` set, it pushes the code branch only up to that commit, and reads the code side as landed once origin holds it.

`PushRun` sets `RecordsHold` only when `l` is the run's own pair, never for the prime's records, which every batten run shares and D5 does not hold:
it reads `pushledger.HeldRecordsCommit` for its slug, which covers a standing landing and a dropped landing's `HeldRecords`.
The hold therefore binds every caller that pushes the run's pair: the pair's pusher, the catch-up, PR-Review's record commit push and the hand-over's owed push all go through it; cancel's and the catch-up's prime records pushes go whole.
`PushRun` sets `CodeCeiling` when `l` is a beam pair whose own target holds landings (`pushledger.OwedTips` of its `RecordsKey`): the first parent of the oldest landing's tip,
so the beam's own commits before its first landing go out and no landing commit reaches origin before the parent pusher's owner reads (card 11), which pass the tip it judged as the ceiling.
A run with no landing and no held commit, and a pair holding no landing, push as today.
Bound: `lym fabric push` in a beam's worktree is the operator's verb and the way forward D5 names for a refused beam parent, so it pushes the beam pair whole.

Tests.
`TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`, tier1, gains rows: with `RecordsHold` naming the second of three unpushed records commits, origin's records branch ends at the first and the code side is pushed;
with `RecordsHold` already on origin the records push is skipped; with `CodeCeiling` naming the first of two unpushed code commits, origin's code branch ends at it.
`TestPushRun_StopsShortOfAHeldRecordsCommit`, tier1, over a pair fixture with a landing entry written through `pushledger.Update`,
pins that `PushRun` of the pair leaves the held commit and the one after it off origin, that `PushRun` of the prime for the same slug pushes the prime's records whole,
that with the landing removed and the commit left as the pair entry's `HeldRecords` the pair push still stops short, and that once that entry is removed the next `PushRun` sends both;
and that a beam pair holding a landing pushes its code branch only up to the landing commit's first parent.

**ImpactSummary:** Adds two fields to the pair push request; only `PushRun` sets them, for a held run's own pair and a beam pair holding landings, so every other push is unchanged.
