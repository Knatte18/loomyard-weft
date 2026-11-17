# Card 4 — The round's snapshot record, written by the review and fix producers

**Create:**
- `internal/shedadapters#roundSnapshot`
- `internal/shedadapters#addedDecisionDigest`
- `internal/shedadapters#roundSnapshotPath`
- `internal/shedadapters#readRoundSnapshot`
- `internal/shedadapters#updateRoundSnapshot`
- `internal/shedadapters#addedDecisionDigests`
- `internal/shedadapters#BurlerReviewProducer.recordRoundStart`
- `internal/shedadapters#TestBurlerReviewProducer_Call_RecordsRoundSnapshot`

**Edit:**
- `internal/shedadapters#BurlerHalfDeps`
- `internal/shedadapters#BurlerReviewProducer`
- `internal/shedadapters#NewBurlerReviewProducer`
- `internal/shedadapters#BurlerReviewProducer.Call`
- `internal/shedadapters#BurlerFixProducer.Call`
- `internal/shedadapters#halfProducerSpec`
- `internal/shedadapters#newReviewProducer`
- `internal/shedadapters#newFixProducer`
- `internal/shedadapters#TestBurlerFixProducer_Call_FinishedFixIsRecorded`

**Extends:**
- `internal/shedadapters/doc.go`

**Uses:**
- `internal/shedadapters#writeAcceptedRound`
- `internal/discussionparser#AddedDecisionTexts`
- `internal/discussionparser#ParseAddedDecisions`
- `internal/state#UpdateJSON`
- `internal/state#ReadJSON`

**Intent:**
D5: one Go-written record per round, `round-<n>-snapshot.json` in the run directory, tracked with the run and written only through `internal/state` under `lock.TrackedFileLockPath` (`PATTERN-tolerant-records`).

`roundSnapshot` holds `fix_scope`, `target_paths` (worktree-relative), `target_id`, `post_fix_target_id`, `added_decisions` (a list of `addedDecisionDigest`: `title` and `digest`) and `decisions_routed` as a `*bool`, so an absent flag is told apart from false.
`roundSnapshotPath` sits with the other round paths in `round.go`.
`updateRoundSnapshot` is a `state.UpdateJSON` over the record, so unknown keys survive every rewrite.
`addedDecisionDigests` pairs each `### Added after Discussion` entry's text from `discussionparser.AddedDecisionTexts` with its title from `discussionparser.ParseAddedDecisions`, in record order, and takes each digest as the hex sha256 of the entry's text, heading and body.
An empty or absent record path gives an empty list.

`BurlerHalfDeps` gains `Snapshot *TargetSnapshot` and `DecisionRecordPath string`; `NewBurlerReviewProducer` stores both on `BurlerReviewProducer`.
A nil `Snapshot` writes no target id: every round then counts as changed (D6) and no changes file is ever built (D4).

`recordRoundStart` runs in `Call` once the round to review is resolved and its acceptance check falls through, before the profile is built, the probe and any spawn.
It writes the record only when none exists for the round, so a review half run again in the same round, a retried attempt or a resumed reviewer keeps the first record and a dead fixer's partial edits never become the round's start.
It fills `fix_scope` and `target_paths` from the stored profile, `target_id` from `Snapshot.Take()` when `Snapshot` is set, and `added_decisions` from `addedDecisionDigests(DecisionRecordPath)`.
A failed id or digest read leaves that field empty and a failed write is logged at Warn; neither fails the round, since a round with no record or no target id only counts as changed.
No exit archives the record, so it outlives an archived review attempt of its own round.

`BurlerFixProducer.Call` sets `decisions_routed` to `p.opts.DecisionsRouted` through `updateRoundSnapshot`, creating the record when absent, once no live fixer is found and before `sendDecisionFindings`, so a spawned and a skipped fix both record it.
A failed write is logged at Warn.

doc.go gains a paragraph on the record.

The test helpers: `halfProducerSpec` gains `Snapshot` and `DecisionRecordPath` fields, and `newReviewProducer` and `newFixProducer` pass them, and the spec's notifier fields, into `BurlerHalfDeps`.

Tests, tier0:
- `TestBurlerReviewProducer_Call_RecordsRoundSnapshot` is a new table, since `TestBurlerReviewProducer_Call_AcceptedReviewIsRecorded` is a usage-record table over engines whose rows do not take a target.
  Its rows run over an overlay-scope `TargetSnapshot` on a temp target and a decision record holding two added entries:
  - an accepted review's `Call` leaves a record holding the scope, the paths, the target id `Snapshot.ID` gives and both entries' titles and digests;
  - a first `Call` whose runner answers a `ReviewErrs` error fails through its failure exit, a target file is edited, and a second `Call` that is accepted leaves the first target id in the record;
  - a nil `Snapshot` leaves a record with no target id.
- `TestBurlerFixProducer_Call_FinishedFixIsRecorded` gains two rows: one whose spec carries a notifier records `decisions_routed: true`, and one with neither notifier nor escalated decisions records `false`.

**ImpactSummary:** Both producers write one new record per round and gain two optional deps fields; with the fields unset the review producer writes a record without a target id and every existing exit is unchanged; the shedadapters test helpers gain the fields the new rows set.
