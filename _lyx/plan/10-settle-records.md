# Card 10 — settle-records

**Create:**
- `plan:internal/shedadapters#Subject` -> `type Subject struct`
- `plan:internal/shedadapters#Subject.Unchanged` -> `func (s Subject) Unchanged(settled Subject, mergeInOnly func(since string) (bool, error)) (bool, error)`
- `plan:internal/shedadapters#settleRecord` -> `type settleRecord struct`
- `plan:internal/shedadapters#generationRecord` -> `type generationRecord struct`
- `plan:internal/shedadapters#readSettleRecord` -> `func readSettleRecord(runDir string) (settleRecord, bool, error)`
- `plan:internal/shedadapters#writeSettleRecord` -> `func writeSettleRecord(runDir string, rec settleRecord, patch []byte) error`
- `plan:internal/shedadapters#readGenerationRecord` -> `func readGenerationRecord(runDir string) (generationRecord, bool, error)`
- `plan:internal/shedadapters#writeGenerationRecord` -> `func writeGenerationRecord(runDir string, rec generationRecord) error`
- `plan:internal/shedadapters#liveSettleRecord` -> `func liveSettleRecord(runDir string) (settleRecord, bool, error)`
- `plan:internal/shedadapters#SettleLive` -> `func SettleLive(runDir string) (settledAt time.Time, live bool, err error)`
- `plan:internal/shedadapters#CarryRecords` -> `func CarryRecords(fromDir, toDir string) error`
- `plan:internal/shedadapters#TestSettleLive` -> `func TestSettleLive(t *testing.T)`
- `plan:internal/shedadapters#TestSubject_Unchanged` -> `func TestSubject_Unchanged(t *testing.T)`
- `plan:internal/shedadapters#TestCarryRecords` -> `func TestCarryRecords(t *testing.T)`

**Edit:**
- `internal/shedadapters#archiveRunDir`
- `internal/shedadapters#TestArchiveRunDir`

**Uses:**
- `plan:internal/state#WriteFile`

**Intent:** The Bouncer's two records of D5 and D6, their liveness test and their survival across an archive, before any Bouncer path writes them.

`Subject` is a segment's subject identity: `Kind` (`document`, `code` or `merge`), `Hash` (a document's identity-text hash, a merge-in's identity), and for a code segment `MergeBase`, `Head`, `FullHash` and `PatchID`; `Base` is the amendment base read with it (a document's weft HEAD, a code segment's merge base).
`Patch` holds a code segment's full diff in memory only and is never encoded.
`Unchanged` decides D5's equality: equal `Kind` and `Hash` for a document or a merge-in; for code, an unchanged merge base and equal `FullHash`, or a moved merge base, `mergeInOnly(settled.Head)` true and equal `PatchID`.
A `mergeInOnly` error is returned.
Bound: a moved merge base with a merge-in-only history and an equal patch-id reads as unchanged, which lets through, unreviewed by this segment, a parent change that interacts with the task's change without sharing a line with it;
the verify tiers bound it, at Publish in loom and at darn's Merge-In, and in loom the Merge-Review segment, unless its merge-overlap skip seam or a merge budget of 0 settles it with no round.

`settleRecord` holds the `Subject`, the amendment `Base`, the `Generation` and `Round` it settled, `SettledAt` (RFC 3339) and `Settled` (true once Commit returned); it lives at `settle-record.json` in the run directory, and a code segment's full diff at `settle-record.patch` beside it.
`generationRecord` holds the generation `Number`, `Amendment`, the amendment `Base`, `RejectReview` (the reject's review file a re-entry after a PR-Gate reject is told) and the `JudgeSubject`, `JudgeBase` and `JudgeAt` (RFC 3339) the latest judge spawn recorded; it lives at `generation-record.json`.
Each read decodes tolerantly through `internal/state` (an absent file is not found, an unknown key reads); each write goes through `internal/state` with its lock beside the file, the patch through `state.WriteFile`.

`liveSettleRecord` returns the settle record when it is live: it names the run directory's current generation (the generation record's number, or 1 when the directory holds a settle record and no generation record), and no `round-<n>-review.md` report numbered past its settled round is on disk.
`SettleLive` is its exported form for the goto guard: the settle time and whether a live record exists, an unparseable settle time an error.
`CarryRecords` copies the three record files that exist from one run directory into another, skipping an absent file and treating an absent source directory as holding none;
it copies nothing when the target already holds a generation record, since every seed and settle with a told subject writes one there, and it copies the generation record last, so a carry that failed part way leaves none and a repeat completes it;
`archiveRunDir` calls it after recreating the directory empty, so the Bouncer's own generation archive keeps them; `PR-Rework`'s archive reaches it through a told seam later.

Tests run at the exported surface, so the unexported read and write helpers are covered through it.
`TestSettleLive` is one table over records written by `writeSettleRecord` and `writeGenerationRecord`: no record (not live, no error); a record naming the current generation with no later report (live, its settle time returned); a later report present (not live); a generation record past the record's generation (not live); a record and no generation record (generation 1, live); a record file holding an unknown key (decodes, live); a record file an older binary wrote with none of the generation keys (decodes); an unparseable `settled_at` (error).
`TestSubject_Unchanged` is one table over the document, merge and code rules: an unchanged merge base with equal and unequal `FullHash`; a moved base with a merge-in-only history and equal patch-id (unchanged); a moved base whose `mergeInOnly` answers false (changed); a moved base with a different patch-id (changed); a `mergeInOnly` error (returned).
`TestArchiveRunDir` asserts the archived generation's three record files are present in the recreated directory, the patch byte for byte, and still in the archive; a row whose run directory holds only the settle record carries that one file and creates no other.
`TestCarryRecords` is one table over the target's state, which `archiveRunDir`'s empty recreated directory never reaches: a target holding a generation record keeps all its files byte for byte; a target holding only the settle record and patch from an earlier, part-way carry gets the three source files; an absent source directory leaves the target as it was.

**ImpactSummary:** archiveRunDir, called only by Bouncer.Call's clear, now also copies the record files; everything else is new and unused until the next card.

**Commit:** 10: shedadapters: the Bouncer's settle and generation records, their liveness and their survival across an archive
