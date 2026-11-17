# Card 7 — Build the round context and the changes file in the review producer

**Create:**
- `internal/shedadapters#BuildRoundContext`
- `internal/shedadapters#roundChangesPath`
- `internal/shedadapters#writeRoundChanges`
- `internal/shedadapters#readRoundChanges`
- `internal/shedadapters#roundParentReportPath`
- `internal/shedadapters#TestBuildRoundContext`

**Edit:**
- `internal/shedadapters#BurlerReviewProducer.recordRoundStart`
- `internal/shedadapters#BurlerReviewProducer.roundProfile`
- `internal/shedadapters#BurlerReviewProducer.Call`
- `internal/shedadapters#TestBurlerReviewProducer_Call_ProfileAndOpts`

**Extends:**
- `internal/shedadapters/doc.go`

**Uses:**
- `internal/burlerengine#ParseFixerReport`
- `internal/burlerengine#CheckFixerReport`
- `internal/burlerengine#ParseReview`
- `internal/burlerengine#ReadParentAsks`
- `internal/shedadapters#readRoundSnapshot`
- `internal/shedadapters#addedDecisionDigests`
- `internal/shedadapters#readGenerationRecord`
- `internal/shedadapters#roundSeed`
- `internal/shedadapters#roundFixerReportPath`
- `internal/shedadapters#newReviewProducer`
- `internal/shedadapters#writeAcceptedRound`
- `internal/shedadapters#writeJudgedRound`

**Intent:**
D2's completeness test, D3's context and D4's incremental rule and changes file, in one exported builder that both the review producer and `lym burler validate-review` call.

`BuildRoundContext` returns nil, with a nil error, when round N-1 is not complete:
round is 1; round N-1's review is absent or does not parse through `ParseReview`; its snapshot record is absent or holds no `decisions_routed`;
or its fixer-report is neither a Go-written one (frontmatter parses with `Spawned` false) nor a fixer-written one that passes `CheckFixerReport` with that flag.
A read error other than absence is returned.
For a complete round N-1 it fills:
- `Prior` from round N-1's findings in review order, each with its disposition by id and `Held` set for a `decision`-class finding when `decisions_routed` is true;
- `AddedTitles` and `AddedTexts` with every entry of `addedDecisionDigests(decisionRecordPath)` whose digest round N-1's record lacks, so an entry added after a `lym loom decision replace` fold, or under a folded entry's title with a body of its own, is listed, and one the record holds is not;
- `Incremental` true unless the run directory's generation record carries the `Amendment` mark;
- `Changed` and `HasChanges` from `readRoundChanges`;
- `ParentReportPath` from `roundParentReportPath` over `burlerengine.ParentReviewFilesOf(anchorPath)`, empty for an empty anchor;
- the three paths for the block.

`roundParentReportPath` reads the ask record and the generation's seed (`roundSeed`) and returns the report path of the round's ask, or empty when the round holds none or either read fails.
`roundChangesPath` is `round-<n>-changes.md` beside the other round paths; the file holds one worktree-relative path per line, written by `writeRoundChanges` and read by `readRoundChanges` (false when absent).

`recordRoundStart` also writes round N's changes file, when none exists, the producer's `Snapshot` is set, round N-1's record holds a `target_id`, the generation carries no `Amendment` mark and `Snapshot.ChangedSince(that id)` answers true.
The written paths are the target's paths changed since round N-1's snapshot; a false answer writes no file and the BLOCKING rule holds alone (D4).
`roundProfile` sets `profile.Round` from `BuildRoundContext(runDir, round, decisionRecordPath, "")`; a builder error is logged at Warn and leaves the round full with today's text.
`Call` sets the context's `ParentReportPath` to the report path of the round's ask once `roundParentAsk` answers a non-zero ask, on both the profile and the probe profile, so the probed, resumed and spawned reviewer judge the same context.

The first round of a segment, the first round of a new generation (the Bouncer archived the run directory, so no round N-1 remains) and every round of an amendment generation are full.

Tests, tier0:
- `TestBuildRoundContext`, one table over a temp run directory. Rows:
  - nil context: round 1; round N-1's review absent; its review unparseable; its snapshot record absent; its record lacking `decisions_routed`; its fixer-written report failing the fix check;
  - an error: round N-1's snapshot record path is a directory, so the read fails other than by absence;
  - a Go-written `spawned: false` report gives a context;
  - routed true marks the `decision`-class finding held and false does not;
  - a held entry is not listed and a new one is, including an entry added after a fold removed earlier ones and one under a folded title with a new body;
  - an amendment generation gives `Incremental` false; a changes file fills `Changed`;
  - `ParentReportPath`: an anchor whose ask record names round N's ask with a report gives that report's path; an anchor with no ask for the round gives empty; an unreadable ask record gives empty.
- `TestBurlerReviewProducer_Call_ProfileAndOpts` gains rows, each laying round 1 out as complete and judged with `writeAcceptedRound` and `writeJudgedRound`, and taking round 1's snapshot by calling the row's overlay `TargetSnapshot.Take` before writing round 1's record:
  - round 1's profile has no `Round`;
  - round 2 gets an incremental `Round` and a changes file listing exactly the target files edited since round 1's snapshot;
  - round 2 whose round 1 record holds no `target_id` gets an incremental `Round` and no changes file;
  - round 2 whose round 1 cache copy was deleted, so `ChangedSince` answers false, gets no changes file;
  - under an amendment generation round 2's `Round` is not incremental and no changes file is written;
  - a row whose deps carry a parent seat that answers a non-zero ask with a report gets that report's path as `ParentReportPath` on the spawned and the probed profile.

**ImpactSummary:** The review producer's profile gains a round context from round 2 on and writes one new file per incremental round; rounds whose round N-1 is incomplete, old run directories included, keep today's prior-rounds text.
