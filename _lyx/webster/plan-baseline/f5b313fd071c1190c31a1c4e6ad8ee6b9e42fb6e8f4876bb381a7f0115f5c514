# Card 8 — accumulated-exclusions

**Create:**
- `plan:internal/shedadapters#excludedLenses` -> `func excludedLenses(name, runDir string, through int, fan string) (excluded, dropped []string)`

**Edit:**
- `internal/shedadapters#BurlerProducer.Call`
- `internal/shedadapters/burler_test.go`
- `internal/shedadapters/doc.go`

**Uses:**
- `plan:internal/shedadapters#readRoundUsage`

**Intent:**
The producer half of decision D5: exclusions accumulate across a segment generation's rounds, held in Go.
`excludedLenses`, in `focus.go`, reads the focus files for rounds 2 through `through` in `runDir` through the existing `readRoundFocus` path (`PATTERN-sole-parsers`) and returns the union of their `exclude_lenses` entries that are eligible.
An entry in round k's focus file is eligible only when round k-1's usage record (`readRoundUsage`) names `fan` as its fan and lists the lens among those it ran;
a missing or unreadable record makes that file's entries ineligible.
So a switch from solo or from another fan drops every exclusion written over the earlier rounds, and the union only ever names lenses of `fan`.
`dropped` names the ineligible entries of round `through`'s own focus file.
A run directory holds one generation, since an archived generation moves the whole directory aside, so a new generation starts from the whole fan.

`BurlerProducer.Call` for round 1 passes no exclusion and drops a round-1 focus file's `exclude_lenses` with a warning naming them.
From round 2 on, a fanned profile gets `ClusterExclude` from `excludedLenses(p.name, runDir, round, fan)` and `ClusterExcludeHeld` from `excludedLenses(p.name, runDir, round-1, fan)`, and the round's `dropped` names are logged as a warning.
Card 3's held fallback then applies when the union would empty the fan.
A solo profile keeps today's warning and drop for any focus excludes.

Bound: the judge can only shrink a fan by lenses the latest round ran under the same fan; an excluded lens returns only with a new generation.

`burler_test.go`: `TestBurlerProducer_Call_ClusterExcludeDropWarning` grows into the one exclusion table at the producer surface, renamed `TestBurlerProducer_Call_ClusterExclusions`, each row stating its wanted `ClusterExclude`, `ClusterExcludeHeld` and drop warning.
Beside its fan-less rows it gains rows for round 1 dropping a focus file's excludes, the union of rounds 2 to N with the held set reaching `ClusterExcludeHeld`, a record naming another fan or no fan dropping the newest focus file's excludes, a lens the latest round did not run being dropped, and a missing usage record making the previous round's entries ineligible.
`TestBurlerProducer_Call_ProfileCarriesDerivedFields` loses its focus file and its `ClusterExclude` assertion, keeping the derived paths and the marker.
That table is the one test of the union and eligibility rules; `excludedLenses` gets no direct test.
The `shedadapters` package doc states the accumulation and eligibility rules.

**ImpactSummary:** Only BurlerProducer.Call's exclusion assignment changes; a fan-less profile and every non-exclusion field behave as before.
