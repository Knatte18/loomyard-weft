# Card 12 — bouncer-amendment

**Edit:**
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.Call`
- `internal/shedadapters#Bouncer.seedCall`
- `internal/shedadapters#Bouncer.budgetReached`
- `internal/shedadapters#Bouncer.atHardCap`
- `internal/shedadapters#Bouncer.settleUndecided`
- `internal/shedadapters#Bouncer.rubricNotes`
- `internal/shedadapters#RubricNotes`
- `internal/shedadapters#ReadRubric`
- `internal/shedadapters#TestReadRubric`
- `internal/shedadapters#NewBouncer`
- `internal/shedadapters#TestBouncer_SoftCapRule`
- `internal/shedadapters/doc.go`

**Create:**
- `plan:internal/shedadapters#Bouncer.startGeneration` -> `func (b *Bouncer) startGeneration(ctx context.Context, settled settleRecord, base string) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/shedadapters#Bouncer.amendment` -> `func (b *Bouncer) amendment() (generationRecord, bool)`
- `plan:internal/shedadapters#AmendmentNote` -> `func AmendmentNote(runDir string) string`
- `plan:internal/shedadapters#TestBouncer_Amendment` -> `func TestBouncer_Amendment(t *testing.T)`

**Uses:**
- `internal/shedadapters#readGenerationRecord`
- `internal/shedadapters#writeGenerationRecord`
- `internal/shedadapters#liveSettleRecord`
- `internal/shedadapters#newBouncerFixture`
- `internal/shedadapters#judgeFakeShuttle`
- `internal/shedadapters#layoutBouncerRun`
- `internal/shedadapters#escalationBouncer`
- `internal/shedadapters#taggedLedgerContent`

**Intent:** D6: a changed subject starts a new generation; for every segment but Merge-Review it is an amendment under a small allowance, scoped to the change.

`BouncerConfig` gains `AmendAllowance int` (the run-wide `review_amend_max_bounces`), `FullGeneration bool` (true for Merge-Review, whose changed merge-in is a new subject) and `RejectReview func() (path string, rejectedAt time.Time, found bool, err error)` (the PR-Gate reject's review file, nil for none).

Case 2 (a live record whose subject changed) and case 3 with a changed subject call `startGeneration`: it reads `RejectReview` and keeps the path when the reject is later than the settle record's `SettledAt` in case 2, or later than the generation record's `JudgeAt` in case 3, which has no settle record, archives the run directory as today (the records travel along), and writes the generation record with the next number and, for an amendment, the `Amendment` mark, the base (the settle record's base; after case 3 the generation record's `JudgeBase`; for code the merge base, the stored patch beside it) and the reject path; a `FullGeneration` record carries the number alone, with no mark, no base and no reject path.
A `RejectReview` error is returned as the Call's error before the archive, and a failed archive degrades as the clear's does today.
For a Plan-Bouncer re-entry after a PR-Gate reject, the base is the settle record's, which `markSettled` replaced with the weft commit that holds the settled plan and PR-Rework's archive carried along (card 14), so the amendment is told a base containing the settled plan.
An amendment with an allowance of 0 settles through `approveWithoutReview`, which writes its settle record; otherwise the seed runs.
A Call that failed between the archive and the seed leaves the generation record naming a generation no seed has run for, and the next Call's seed takes it from there.
`seedCall` marks its Stuck `GenerationStart` when the generation record names a generation past 1 and the seed reached `ensureFocus` without degrading; the re-bounce Stuck and a degraded seed never carry it.
The settle record kept across the archive names an older generation, so later Calls of the new generation take the round-by-round path, and its next settle writes a new record.

`amendment` reports the live generation record when it marks an amendment.
In an amendment `budgetReached` compares the spent count against `AmendAllowance` clamped to the `Bounces` seam's budget, and `atHardCap` uses `AmendAllowance + 1` clamped to `HardCap` (a `HardCap` of 0 leaves `AmendAllowance + 1`), so the soft-cap rule runs at round A and a CONTINUE at round A+1 escalates with accept alone, as at any perch's hard cap; with the seam unwired Shed's generic block applies the perch's budget.
A full generation keeps the perch's own budget and hard cap, and the allowance never applies to it.
The comments of `atHardCap` and `settleUndecided` that say a goto resets Shed's count now say a new generation does, and so do `doc.go`'s soft-cap paragraph ("a round past a goto that reset the count") and `TestBouncer_SoftCapRule`'s doc comment and its two "goto-reset count" row names, which become "a new generation's count".
`NewBouncer`'s doc comment, whose budget rule says the episode resets at the settling Done and a re-entered segment clears and re-seeds with a fresh budget, is rewritten to the generation rules.

`AmendmentNote(runDir)` renders, from the generation record, the note the seed, the reviewers and the judge read: `none` outside an amendment; for a document, review only the artifacts' change since weft commit `<base>`; for code, the settled side is the stored patch file at its path and the current side the working-tree diff against the current merge base, untracked files outside the exclusion set included, so a change already uncommitted at the settle is never shown as new and parent changes a Merge-In brought are left out; plus the reject's review file when one is recorded.
`RubricNotes` gains `Amendment`, `ReadRubric` fills an optional `amendment` marker from it (`none` when blank), and `Bouncer.rubricNotes` fills it from `AmendmentNote(b.cfg.RunDir)`.
`doc.go`'s budget rule replaces "goto grants a fresh budget" and "the episode never resets ... fresh again in the next generation" with the generation rules above.

`TestBouncer_Amendment` is one table:
a live record and a changed subject archive the generation, write generation 2 marked amendment with the record's base, seed with a prompt holding the amendment note, and return a Stuck marked `GenerationStart`;
its second Call, after round 1's report is written, judges round 1 without a second archive;
with `AmendAllowance` 1 and a `Bounces` fake at count 1, round 1's CONTINUE meets the soft-cap rule, and a CONTINUE at round 2 escalates hard-cap offering accept alone;
with `AmendAllowance` 3 above a `Bounces` budget of 2, the soft-cap rule runs at the perch budget, and with `AmendAllowance` 3 and `HardCap` 3 a CONTINUE at round 3 escalates hard-cap (the clamp), while with `HardCap` 0 and `AmendAllowance` 1 it does at round 2;
`AmendAllowance` 0 settles with no spawn and writes a settle record naming generation 2, and a re-entry after it with an unchanged subject returns Done with no spawn;
`FullGeneration` writes generation 2 with no amendment mark and no base, seeds marked `GenerationStart`, keeps the perch budget with an allowance of 0, and its second Call, after round 1's report is written, judges round 1 without a second archive;
a code-kind amendment's seed prompt holds the code note, naming the stored `settle-record.patch` as the settled side and the working-tree diff against the current merge base as the current side;
a `RejectReview` error is returned with nothing archived, and a failed archive degrades to a Stuck with the run directory untouched;
a Call whose seed failed after the archive is followed by a Call whose seed carries the mark, while a re-bounce Stuck carries none;
a settle interrupted at the perch's last round with the count at the budget, then a changed subject, returns the seed's Stuck rather than a halt;
case 3 with a changed subject starts the amendment from the generation record's `JudgeBase`, and a reject older than its `JudgeAt` names no reject file in the note;
a `RejectReview` later than the settle puts its path in the note, and one earlier does not.
`TestReadRubric` adds a row for the `amendment` marker filled and blank.

**ImpactSummary:** Bouncer budget and hard-cap reads change only inside an amendment generation; ReadRubric gains one optional marker that rubrics without it ignore.

**Commit:** 12: shedadapters: a changed subject starts an amendment generation under the allowance, scoped to the change
