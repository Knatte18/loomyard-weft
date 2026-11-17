# Round 3 fixer-report

All five findings were fixed as proposed.
`lym loom validate-plan` answers `ok: true`, with informational findings only.

## Changes

- **F1** (BLOCKING): card 6's Create group gains `plan:internal/shuttleparity#TestMain`.
  Its Intent says the package's `testmain_test.go` mirrors `internal/seatengine/testmain_test.go`: it calls `gitkit.HermeticGitEnv()` and then `os.Exit(tmuxkit.Main(m))`.
- **F2** (MEDIUM): the cards now add interim `testOnlyCodeAllowed` entries, each with a `Why` naming the card that removes it.
  - Card 5 adds `cmd/lym#testOnlyCodeAllowed` to its Edit group, an Intent paragraph adding `internal/shuttleengines.NewWithSeams`, and an ImpactSummary clause.
  - Card 6 gains an Edit group holding `cmd/lym#testOnlyCodeAllowed`, an Intent paragraph adding `internal/seatengine.Specs`, `internal/landingshed.FixReviewSeatTable` and `internal/burlerengine.Engine.SeatTables`, and an ImpactSummary.
  - Card 8's Intent deletes these entries along with `internal/seatengine.Prompts`, and its ImpactSummary names them.
- **F3** (MEDIUM): in card 6, `Seat.Specs` now sets the seat's `ModelChoice` to `ModelChoice{Engine: engine}` alone.
  The Intent says why `Model`, `Effort` and `Version` are cleared.
  `TestCompare` gains a `seatsOf` row: a seat whose `ModelChoice` carries all three gives, for each engine, a spec whose `ModelChoice` is that engine alone.
- **F4** (LOW): card 5's `internal/shuttleengines#build` Edit glyph carries the re-sign arrow with the new signature.
- **F5** (LOW): card 6's Intent declares the reference and compared engine names once each as named constants of `internal/shuttleparity`, and says who uses them.
  `Compare`'s text now names the reference and compared engines and no literal.

Self-check changes:

- Card 2 gains `cmd/lym#testOnlyCodeAllowed` in its Edit group and an interim entry for `internal/shuttleengine.MissingItems`, whose one production caller is card 6's `Compare`.
  Its ImpactSummary names the entry, and card 8 deletes it with the others.
- Card 2's `copyManifest` now reads the record through `ReadManifest`, so `ReadManifest` is reachable from card 2 on and needs no entry.
- Card 6's engine-name constants are exported, and card 8's `TestManifestCmd` in `internal/shuttlecli` takes its stub's map keys and expected engine name from them.

## Self-check

- fix-claims: no findings.
- fix-affected-set, MEDIUM: card 2's `MissingItems` is unreached until card 8 and has no interim entry. Fixed with my own change, listed above.
  The same finding raised `ReadManifest`, which I fixed differently: `copyManifest` now reads through it, so it is reached from card 2 and needs no allowlist entry.
- fix-affected-set, fix-consistency and fix-proposal, LOW (one finding, merged): card 8's `TestManifestCmd` in `internal/shuttlecli` had no source for the engine name. Fixed with my own change: the constants are exported and that test uses them.

## Disputed

None.

## Deferred

None.

## Rulings

None.
