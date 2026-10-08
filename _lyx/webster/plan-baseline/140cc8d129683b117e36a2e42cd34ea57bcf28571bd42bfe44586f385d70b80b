# Card 10 — judge-exclusion-prompt

**Edit:**
- `internal/shedadapters#focusSchemaMarkers`
- `internal/shedadapters#Bouncer.runSeedSpawn`
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#TestBouncer_ClusterExcludesReachesSeedAndJudgePrompts`
- `internal/shedadapters/doc.go`

**Intent:**
The prompt half of decision D5.
The seed call is asked for no `exclude_lenses`, since no round has run for it to judge settled: `Bouncer.runSeedSpawn` fills the focus-only variant of `focusSchemaMarkers` whatever `ClusterExcludes` says.
The fanned variant, which only the judge call now gets, adds to its `focus_list_rules` text that an exclusion is permanent for the segment generation, that a lens is excluded only when the latest round ran it (the facts' `## Lenses` section lists them) and its area is settled, meaning the latest round's findings from it carry nothing at MEDIUM or worse and its area is not in the next round's `focus`, and that earlier exclusions are never restated.
The text stays Go-held in `focusSchemaMarkers`, so the stencils stay conditional-free (`PATTERN-stencil-ownership`).
The `ClusterExcludes` field doc on `BouncerConfig` says the judge prompt alone asks for `exclude_lenses`.

`TestBouncer_ClusterExcludesReachesSeedAndJudgePrompts` asserts the seed prompt names no `exclude_lenses` in either mode, and the fanned judge prompt carries the permanence, eligibility and no-restatement rules.
The `shedadapters` package doc's exclusion paragraph matches.

**ImpactSummary:** The seed prompt loses the exclude_lenses schema and the fanned judge prompt gains three rules; a Bouncer not told ClusterExcludes renders exactly as before.
