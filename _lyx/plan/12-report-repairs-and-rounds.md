# Card 12 — report-repairs-and-rounds

**Create:**
- `plan:internal/reportengine#ArbiterOutcome` -> `type ArbiterOutcome struct`
- `plan:internal/reportengine#SegmentRounds` -> `type SegmentRounds struct`
- `//internal/reportengine/rounds_test.go`

**Edit:**
- `//internal/reportengine/doc.go`
- `//internal/reportengine/fake_test.go`
- `plan:internal/reportengine#allowedImports`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/loomengine#ParseArbiterAttemptDir`
- `internal/discussionparser#ParseAddedDecisions`
- `internal/shedadapters#ReviewRounds`
- `internal/shedadapters#ArchivedRunDirSegment`
- `internal/parentreview#ParseRoundDir`
- `internal/loomengine#DiscussionDecisionRecordRel`
- `internal/loomengine#LoomReviewsDirRel`
- `internal/loomengine#LoomParentReviewDirRel`

**Intent:**
Each full run gains its repairs split by actor as far as the records allow (D8) and its review rounds and parent decisions (D9), in a new file `rounds.go`; every record is read at the tag through its owner's tolerant parse, and a source whose directory or file the tree lacks is absent, never zero (D5).

Repairs (D8):
- Automatic steps are card 11's `RunRow.Automatic`.
- Arbiter repairs: each attempt directory under `TreeRunRels(slug).Arbiter` is read through `loomengine.ParseArbiterAttemptDir` and counts in `RunRow.Arbiter`, an `ArbiterOutcome` per verdict word (`none` when no verdict file) with its count and how many carry the applied mark; the source is absent without the directory.
- LLM steps, one count since the records do not tell the loom driver from the orch, reported per source in `RunRow.LLMSteps`:
  `loom` counts every `goto` and `reroute` entry in the loom history at the tag, except the entries card 11's `AccountedChild` names (batten's verify retry and moved-parent resume) and a `goto` an applied arbiter attempt accounts for (the first `goto` dated after the attempt's spawn whose producer is the verdict's `goto` row);
  `batten` counts the `goto` and `reroute` entries in the batten record's history and every recorded child change from `blocked` or `failed` into `running` that no applied arbiter attempt accounts for (an applied attempt whose stop key's producer and state equal the change's producer and prior state, each attempt accounting for one change, the first after its spawn), and is absent with the batten source.
  Card 13 adds `landing` for a `hand` landing.
- Drive reports: `RunRow.DriveReports` is the number of files under `TreeRunRels(slug).DriveReports`, one per driver attempt, bodies unread; absent without the directory.

Review rounds and decisions (D9):
- Every subdirectory of `LoomReviewsDirRel()` but the parent-review one is a segment; `shedadapters.ReviewRounds` over its file names gives `SegmentRounds{Rounds, Outcome}` in `RunRow.Segments`, keyed by segment name.
  A subdirectory `shedadapters.ArchivedRunDirSegment` accepts is a Bouncer re-seed's archived generation, not a segment of its own: its `ReviewRounds` rounds add to its segment's `Rounds`, while `Outcome` stays the live directory's last round.
  The segment set of the report is every segment any selected run's tree holds, so a run lacking one of them reports it absent.
- Parent-review rounds count under `parent-review` in the same map: the round directories `RoundNumber` accepts whose `ParseRoundDir` result holds a request, with the outcome the last round's verdict kind (`approve` or `reject`), `none` when it has no verdict.
- Parent decisions: `discussionparser.ParseAddedDecisions` over `DiscussionDecisionRecordRel()` at the tag gives `RunRow.Decisions`, a count per `by`; absent without the file, and a parse error is a finding naming the tag and the file.

`seam_enforcement_test.go`'s `allowedImports` gains `discussionparser`.
`fake_test.go` gains builders for an arbiter attempt directory, drive reports and a decision record.
`doc.go` describes the actor split and the round counts.

Tests:
- `TestCompute_RepairsAndRounds` (new, in `rounds_test.go`) is table-driven, each row one run and its exact arbiter outcomes, LLM steps per source, drive reports, segments, parent-review rounds, decisions and findings:
  a resume backed by an applied arbiter verdict is an arbiter repair and no LLM step, and an arbiter attempt whose verdict was not applied shows under its verdict as not applied;
  a `goto` entry and a `reroute` entry each count as an LLM step, a resume after a halt with no automatic step or applied arbiter verdict is an LLM step, and the `goto` of batten's verify retry is none;
  two `Added after Discussion` entries by one author and one by another yield two and one;
  a segment whose last round holds an escalation file is `escalated`, one holding a circling-decision file is `circling-decided` with or without an escalation file, one holding neither is `accepted`, a numbered round with no review file is not a round, and a run with no directory for a segment another run holds reports it absent;
  a run with `plan` holding one round and `plan-20261012T101500Z` holding two reports `plan` with 3 rounds and no `plan-20261012T101500Z` key;
  a parent-review round takes its outcome from its verdict record;
  a run with no drive-reports directory reports drive reports absent;
  a malformed `Added after Discussion` heading is a finding naming the tag and the decision record, and the run's other counts stand.
- `TestCompute_FullRunCounts` (new, in `rounds_test.go`) builds one full loom run holding every source (batten state changes, halt notes, arbiter attempts, drive reports, alignment attempts, parent-review rounds, review rounds per segment and parent decisions) and asserts the expected count for each, counts only, since each classification rule has its own row above or in card 11.

`cmd/lyx/testdata/test-budget.yaml` raises `internal/reportengine` to 5.

**ImpactSummary:** Extends the unreleased report engine and its test support; nothing outside `internal/reportengine` and the budget file changes.

**Commit:** `12: report engine: repairs, LLM steps, review rounds and decisions`
