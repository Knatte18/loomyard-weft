# Card 11 — bouncer-reentry

**Edit:**
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.Call`
- `internal/shedadapters#Bouncer.approveWithoutReview`
- `internal/shedadapters#Bouncer.settle`
- `internal/shedadapters#Bouncer.acceptRound`
- `internal/shedadapters#Bouncer.seedCall`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedadapters#Bouncer.settledGeneration`
- `internal/shedadapters/doc.go`

**Create:**
- `plan:internal/shedadapters#Bouncer.recordSettle` -> `func (b *Bouncer) recordSettle(round int, subject Subject) error`
- `plan:internal/shedadapters#Bouncer.markSettled` -> `func (b *Bouncer) markSettled() error`
- `plan:internal/shedadapters#Bouncer.resettle` -> `func (b *Bouncer) resettle(rec settleRecord) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/shedadapters#Bouncer.liveSubject` -> `func (b *Bouncer) liveSubject() (Subject, bool, error)`
- `plan:internal/shedadapters#TestBouncer_Reentry` -> `func TestBouncer_Reentry(t *testing.T)`

**Uses:**
- `plan:internal/shedadapters#liveSettleRecord`
- `plan:internal/shedadapters#writeSettleRecord`
- `plan:internal/shedadapters#readGenerationRecord`
- `plan:internal/shedadapters#writeGenerationRecord`
- `plan:internal/shedadapters#Subject.Unchanged`
- `internal/shedadapters#newBouncerFixture`
- `internal/shedadapters#judgeFakeShuttle`
- `internal/shedadapters#layoutBouncerRun`
- `internal/shedadapters#layoutApprovedGeneration`
- `internal/shedadapters#carryOverRecorder`
- `internal/shedadapters#withClock`
- `internal/shedadapters#layoutCirclingBouncer`

**Intent:** D5's settle record and re-entry cases 1, 3 and 4 in the Bouncer.

`BouncerConfig` gains three told seams, each nil by default, which keeps a Bouncer that tells none, and every existing test, on today's paths:
`Subject func() (Subject, bool, error)` reads the live subject, false meaning it has none (a standalone run's code segment, which writes no record and takes case 4);
`MergeInOnly func(since string) (bool, error)` is `Subject.Unchanged`'s merge-in test for a code segment;
`SettledBase func() (string, error)` reads a document segment's weft HEAD after Commit returns.
A `Subject` read error is returned as the Call's error, as a failing seam is today.

Writes.
`recordSettle` writes the settle record (generation from the generation record, else 1; the settled round; `SettledAt` now; `Settled` false; the subject, its base and, for code, its patch) and `markSettled` sets `Settled` and, when `SettledBase` is told, replaces the base with the weft HEAD it reads.
They run on every approved settle: `settle`'s CONVERGED branch and `acceptRound` call `recordSettle` before carry-over, Approve and Commit and `markSettled` once Commit returns; `approveWithoutReview` (the skip seam, a zero-round perch) does the same, and first writes a generation record when the directory holds none.
`seedCall` writes the generation record at every seed, generation 1 included, when the directory holds none; `judgeCall` writes the live subject and its base into the generation record as `JudgeSubject` and `JudgeBase` before the judge spawn.

Re-entry, at Call entry after the skip seam, at any round including 0:
1. A live settle record whose subject is `Unchanged` against the live one: `resettle` runs carry-over for the settled round (the closing taken from the round's decision: an accept settles as accepted, else converged), skipping it for a record of round 0, which `approveWithoutReview` writes and whose settle never called carry-over; then Approve and Commit again, marks the record settled keeping its `SettledAt`, and returns Done with the settled round's ledger as the pointer when it exists; no seed, judge or review spawns and no Stuck is returned, so no budget is spent; a failing step returns its error, as today.
3. No settle record, a generation record, and a settled verdict at the current round (`settledGeneration`): when the live subject equals the generation record's `JudgeSubject`, the verdict is settled through `settle` as a harvest (`spawned` true), which writes the record first; no clear, seed or review spawns.
4. No settle record and no generation record over a settled verdict: today's clear and full re-seed.
A live record whose subject changed (case 2), and case 3 with a changed subject, take today's clear and re-seed in this card; the next card turns them into a new generation.
A record that is not live takes the round-by-round path with no comparison, and on that path a settled verdict at the current round is settled through `settle`, never cleared: the clear belongs to case 4 alone.
`settledGeneration`'s doc and `Call`'s doc state the four cases, and `doc.go`'s Bouncer contract states the record, its writes and the cases.

`TestBouncer_Reentry` is one table over `newBouncerFixture` with a fake `Subject` and a `Bounces` fake that fails the row when called on a no-spawn case:
a CONVERGED settle whose Commit failed, resumed with an unchanged subject, runs Approve and Commit again, returns Done, spawns nothing, writes `settled: true` and keeps the first write's `SettledAt` under a clock advanced between the Calls;
the same with a carry-over seam and open findings re-runs carry-over (the Discussion case);
a settled circling accept whose Commit failed re-settles with the accepted closing and no spawn;
a re-entry after a skip-seam approval and after a zero-round approval, each with a carry-over seam installed, returns Done with no spawn and never calls carry-over;
a document settle with a `SettledBase` fake marks the record settled with the base it reads after Commit, replacing the base written with the record, and an interrupted settle keeps the first base;
a code subject whose merge base moved with a merge-in-only history and equal patch-id returns Done with no spawn;
case 3 with an equal judge subject settles through the harvest path, writes the record and archives nothing;
a settle record with a round N+1 report on disk judges round N+1 and, on CONVERGED, writes a new record whose `SettledAt` is later under the advanced clock;
a settle record made not live by an unharvested CONVERGED round N+1 already on disk at entry settles that round with no archive and no spawn;
a `Subject` seam answering false writes no record and clears on re-entry as case 4;
a `Subject` read error is returned as the Call's error, and a Commit failing again inside `resettle` returns its error with the record left unsettled.
Case 4 over a directory with no records keeps the coverage of `TestBouncer_Clear_ApprovedRunDirClearsAndReseeds` and `TestBouncer_Clear_AfterCommitFailureSubsequentCallClears`, whose nil `Subject` keeps that path, so the table adds no row for it.

**ImpactSummary:** Bouncer.Call's entry and every approved-settle path gain record writes behind nil-default seams; the loom and darn wiring tells the seams in a later card.

**Commit:** 11: shedadapters: the Bouncer records its settle and re-settles an unchanged subject without a review
