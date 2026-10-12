# Card 5 — review-round-and-decision-readers

**Create:**
- `plan:internal/shedadapters#RoundOutcomeAccepted` -> `const RoundOutcomeAccepted = "accepted"`
- `plan:internal/shedadapters#RoundOutcomeEscalated` -> `const RoundOutcomeEscalated = "escalated"`
- `plan:internal/shedadapters#RoundOutcomeCirclingDecided` -> `const RoundOutcomeCirclingDecided = "circling-decided"`
- `plan:internal/shedadapters#RoundSummary` -> `type RoundSummary struct`
- `plan:internal/shedadapters#ReviewRounds` -> `func ReviewRounds(names []string) RoundSummary`
- `plan:internal/shedadapters#TestReviewRounds` -> `func TestReviewRounds(t *testing.T)`
- `plan:internal/shedadapters#ArchivedRunDirSegment` -> `func ArchivedRunDirSegment(dirName string) (string, bool)`
- `plan:internal/discussionparser#ParseAddedDecisions` -> `func ParseAddedDecisions(record []byte) ([]AddedDecision, error)`
- `plan:internal/discussionparser#TestParseAddedDecisions` -> `func TestParseAddedDecisions(t *testing.T)`

**Edit:**
- `internal/discussionparser#TestAppendDecision_AppendsAtEndOfDecisionsSection`
- `internal/shedadapters#TestArchiveRunDir`
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D9 counts review rounds per segment with the last round's outcome, and parent decisions per author, each from bytes or names a git tree holds, read through the package that writes them.

`shedadapters.ReviewRounds` (in `round.go`, beside the round path helpers that own the `round-<N>-*` names) takes the file names directly in one review segment's directory and returns a `RoundSummary`:
`Rounds` is the number of round numbers that have a `round-<N>-review.md`, so a numbered round with only a focus file is not a round;
`Outcome` is the last such round's: `RoundOutcomeCirclingDecided` when it holds `round-<N>-circling-decision.md`, whether or not it also holds `round-<N>-escalation.md`, `RoundOutcomeEscalated` when it holds only the escalation file, and `RoundOutcomeAccepted` otherwise; it is empty when `Rounds` is 0;
`EscalatedRounds` lists, ascending, the round numbers that hold an escalation file, which card 11 pairs with Bouncer halts.
A name it does not recognise is ignored.

`shedadapters.ArchivedRunDirSegment` (in `archive.go`, beside `archiveRunDir`, which writes the `<segment>-<20060102T150405Z>[<suffix>]` names when the Bouncer re-seeds) takes one directory name and returns the segment a timestamped run-directory sibling belongs to, with `true`; a name that does not end in that stamp, with or without a `-<n>` collision suffix, returns `""` and `false`.

`discussionparser.ParseAddedDecisions` (in `append.go`, beside `AppendDecision`, which writes the entries) returns every `### Added after Discussion (<by>, <date>): <title>` heading outside a fenced block, in order, as an `AddedDecision` with `By`, `Date` (parsed as `2006-01-02` in UTC) and `Title` set and `Decision` and `Rationale` empty.
A heading that starts `### Added after Discussion (` but does not parse to a non-empty author and a valid date is an error naming its line number, so the report turns it into a finding (D5); a record with no such heading returns none.
It reuses the package's `fencedLines`, and the package keeps its stdlib-only import set.

Tests:
- `TestReviewRounds` (new, in `round_test.go`) is a table over name lists: rounds 1 and 2 with reviews and a `round-3-focus.md` give 2 rounds; a last round with a circling-decision and an escalation file is `circling-decided` with `EscalatedRounds` `[2]`; a last round with only an escalation file is `escalated`; a last round with neither is `accepted` while an earlier round's escalation still lands in `EscalatedRounds`; an empty list gives 0 rounds and an empty outcome; `round-x-review.md` and `notes.md` are ignored.
- `TestParseAddedDecisions` (new, in `append_test.go`) is a table: a record with two entries by `alice` and one by `bob` returns three entries in order with their dates; a heading inside a fenced block is skipped; `### Added after Discussion (alice): t` (no date) is an error naming its line; a record with none returns an empty slice.
- `TestAppendDecision_AppendsAtEndOfDecisionsSection` adds that `ParseAddedDecisions` over the written record returns the appended entry's author, date and title, which pins the reader to the writer's format.
- `TestArchiveRunDir` adds that `ArchivedRunDirSegment` over each archived sibling's name, the collision-suffixed one included, returns `run` and `true`, and over `run` itself returns `false`, which pins the reader to `archiveRunDir`'s naming.

`cmd/lyx/testdata/test-budget.yaml` raises `internal/shedadapters` to 155 and `internal/discussionparser` to 18.

**ImpactSummary:** Two existing tests each gain a round-trip assertion; the new members have no caller yet, and both packages keep their import sets.

**Commit:** `5: review round and parent decision readers`
