# Card 18 — treadle-callers

**Edit:**
- `internal/shuttleparity#producers`
- `cmd/lym#pathRules`

**Delete:**
- `internal/shuttleparity#treadleJudgeInputs`
- `internal/shuttleparity#buildTreadleJudge`
- `internal/shuttleparity#buildTreadleTargeting`

**Uses:**
- `internal/shuttleparity/doc.go`

**Intent:**
Remove every caller of `internal/treadleengine` outside the package, ahead of its deletion in card 19 (D8).

`producers` in the parity tests drops its two treadle rows, and the three treadle builders go with them.
`pathRules` in `cmd/lym` drops its `treadleengine.PauseFlagPath` row.
Both are tests whose remaining rows keep their coverage; nothing in production calls treadle.

**ImpactSummary:** Test-only rows and builders removed; after this card nothing outside internal/treadleengine imports it.
