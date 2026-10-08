# Card 9 — lens-facts

**Create:**
- `internal/shedadapters#lensFacts`
- `internal/shedadapters#computeLensFacts`

**Edit:**
- `internal/shedadapters#roundFacts`
- `internal/shedadapters#renderRoundFacts`
- `internal/shedadapters#writeRoundFacts` -> `func writeRoundFacts(name, runDir string, round int, reportName func(int) string, clusterExcludes bool) error`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedadapters/bouncerfacts_test.go`
- `internal/shedadapters/doc.go`

**Uses:**
- `internal/shedadapters#excludedLenses`
- `internal/shedadapters#readRoundUsage`

**Intent:**
Decision D6: the judge of a fanned segment sees each lens's findings.
`lensFacts` holds the lenses the latest round ran, the latest review's finding counts per `origin` value (each `lens:<name>` and `handler`, an absent origin counted under its own label) by severity and by class, and the lenses already excluded for the next round.
`computeLensFacts` reads the lenses and the fan from round `round`'s usage record, counts the round's review through `burlerengine.ParseReview` (never a second parser), and takes the excluded lenses from `excludedLenses(name, runDir, round, fan)`;
a missing usage record or an unparseable review renders as a line saying so, never an error.
`roundFacts` gains an optional lens part, and `renderRoundFacts` renders it as a `## Lenses` section, deterministically ordered like the rest of the file, only when it is set.
`writeRoundFacts` computes the lens part only when `clusterExcludes` is true, so a solo segment's facts file is byte-identical to today's.
`Bouncer.judgeCall` passes its `Name` and `ClusterExcludes`.

`bouncerfacts_test.go`: the existing `writeRoundFacts` calls pass `false` and keep their expected bytes, and a new test pins a fanned round's lens section with two lenses and a handler finding, the excluded lenses, and a missing usage record.
The `shedadapters` package doc's facts-file paragraph names the lens section.

**ImpactSummary:** writeRoundFacts gains two parameters; its only production caller is Bouncer.judgeCall and its test calls sit in bouncerfacts_test.go, and solo facts files are unchanged.
