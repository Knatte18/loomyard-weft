# Card 1 — shuttle-usage-split

**Edit:**
- `internal/shuttleengine#SessionUsage`
- `internal/shuttleengine#UsageReader`
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#TestRun_Finalize_ReportsUsageAndTimes`
- `internal/shuttleengine/claudeengine#Claude.SessionUsage`
- `internal/shuttleengine/claudeengine#TestSessionUsage_SumsParentAndForks`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shuttleengine/claudeengine#parsedTranscript.tokens`
- `internal/shedadapters#newHalfUsage`

**Intent:** D12's token split.
`SessionUsage` gains the four counts apart, `Input`, `CacheWrite` and `Output` beside the existing `CacheRead`, each an `int64`, and `SplitKnown bool`, which says the four were read; `Known`, `Fresh` and the fork fields stay as they are, so the round usage record (`newHalfUsage`) keeps its own reading unchanged.
`Claude.SessionUsage` fills the four from the same per-transcript totals it sums today, main transcript and forks together, and sets `SplitKnown`; an unreadable session stays the zero value, every count unknown.
`Run.finalize` reads the usage at every terminal outcome, not only `OutcomeDone`: the `outcome == OutcomeDone` condition on the `UsageReader` call is dropped, so a stuck, died, timed-out or API-error run carries `Result.Usage` too.
The doc comments of `Result.Usage`, `UsageReader` and the "Session usage" paragraph of `internal/shuttleengine/doc.go` say "every terminal outcome" in place of "known only for an OutcomeDone run"; `claudeengine/doc.go` names the four counts.
`TestRun_Finalize_ReportsUsageAndTimes` today asserts a died run makes no `SessionUsage` call: that row now asserts the died run's `Result.Usage` is the fake's reading, and its `//testtiming:keep` reason is reworded to match; a row of each of the done and the died outcome asserts the four counts pass through.
`TestSessionUsage_SumsParentAndForks` adds the assertion of the four split counts summed over parent and forks, and `SplitKnown` false for an unreadable session.
Both tests stay tier0.

**ImpactSummary:** Adds fields to one shared struct and widens one read to every outcome; the two round-usage readers in shedadapters read only `Known`, `Fresh` and `CacheRead`, so their records only gain readings for non-done halves.
