# Card 6 — bouncer-per-round-exclusions

**Create:**
- `plan:internal/shedadapters#Bouncer.excludesLenses` -> `func (b *Bouncer) excludesLenses(round int) (bool, string)`

**Edit:**
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedadapters#roundFacts`
- `internal/shedadapters#renderRoundFacts`
- `internal/shedadapters#writeRoundFacts` -> `func writeRoundFacts(name, runDir string, round int, reportName func(int) string, clusterExcludes bool, usageNote string) error`
- `internal/shedadapters#TestBouncer_ClusterExcludesReachesSeedAndJudgePrompts`
- `internal/shedadapters#TestComputeRoundFacts_ClasslessReviewIsAParseErrorRow`
- `internal/shedadapters#TestWriteRoundFacts_IsDeterministic`
- `internal/shedadapters#TestWriteRoundFacts_LensSection`
- `internal/shedadapters/doc.go`

**Uses:**
- `plan:internal/shedadapters#readRecordedFans`
- `plan:internal/shedadapters#roundFan`
- `plan:internal/shedadapters#recordRoundFans`
- `internal/shedadapters#newBouncerFixture`
- `internal/shedadapters#withStencilOverrides`
- `internal/shedadapters#assertExcludeLensesText`
- `internal/shedadapters#mergeRoundUsage`

**Intent:**
Key the judge's `exclude_lenses` request to the next round's fan (D10, D2): the judge of round N asks for `exclude_lenses`, and the facts file carries its `## Lenses` section, only when round N ran a fan and round N+1's resolved fan is the same non-empty fan.

`BouncerConfig` gains `ClusterFans`, the segment's per-round reviewer fan list as card 5's `RoundFans.Review`; nil keeps the told `ClusterExcludes` bool as the whole answer, so a row the wiring names no list for behaves as today.
`excludesLenses(n)` answers for a non-nil list: it reads round n's recorded names through `readRecordedFans`;
a record that is missing, lacks `fans_recorded` or does not decode counts as a round that ran no fan, answering false with a note naming the reason, and never fails the judge;
a recorded empty fan answers false;
otherwise it answers whether the recorded fan equals `roundFan(ClusterFans, n+1)`.
`judgeCall` passes the answer to `focusSchemaMarkers` and `writeRoundFacts`, and the note to `writeRoundFacts`, in place of `b.cfg.ClusterExcludes`.
`roundFacts` gains `UsageNote`, which `renderRoundFacts` renders as one line, `Usage record unreadable, so this round counts as solo for lens exclusions: <note>`, so the facts file notes the record as `computeLensFacts` notes a usage error today.
The existing eligibility rules in `excludedLenses` are unchanged.

Tests:
`TestBouncer_ClusterExcludesReachesSeedAndJudgePrompts` gains `ClusterFans` rows in its judge subtest, each a judge call of round 1 over a round-1 record written with `recordRoundFans` and then a review half merged through `mergeRoundUsage`, so `computeLensFacts` reads a completed record.
Each row asserts the prompt through `assertExcludeLensesText` and the facts file's `## Lenses` section and note: `[fan-a, ""]` asks for none; `[fan-a]` asks; `[fan-a, fan-b]` asks for none; `[""]` over a recorded empty fan asks for none, which reaches the recorded-empty guard where `"" == ""` would otherwise answer true;
a missing record, a flagless record and one that does not decode each ask for none, write the note, and the call succeeds.
The four existing `writeRoundFacts` call sites in `bouncerfacts_test.go` pass an empty note, and `TestWriteRoundFacts_LensSection` gains a subtest that a non-empty note renders its line.
`internal/shedadapters/doc.go`'s judge paragraph states the per-round rule.

**ImpactSummary:** `writeRoundFacts` has one production caller, `Bouncer.judgeCall`, and four test call sites; a Bouncer told no `ClusterFans` keeps today's prompt and facts file.
