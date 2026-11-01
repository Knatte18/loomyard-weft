# Card 20 — root-prerun-installs

**Edit:**
- `cmd/lym#newRoot`

**Create:**
- `cmd/lym#TestPersistentPreRunE_InstallsSeams`

**Uses:**
- `cmd/lym#findPersistentPreRunELit`
- `internal/fabricengine#InstallWorktreeSeam`
- `internal/fabricengine#RegisterBoltReplayer`
- `internal/shuttleengine#InstallPassageReader`
- `internal/heddleengine#Seam`
- `internal/heddleengine#Replayer`
- `internal/heddleengine#Renderer.Passages`

**Intent:** Have every lym process install heddle's seams in the root pre-run (D4, D6, D7), beside the push notifier and the board's replayer, so every fabric move, Bolt replay and Claude launch in any lym process, detached or not, reaches heddle.

- Inside the pre-run's `!testing.Testing()` block, after the board replayer's registration: `fabricengine.InstallWorktreeSeam(heddleengine.Seam{})`, `fabricengine.RegisterBoltReplayer(heddleengine.Replayer{})` and `shuttleengine.InstallPassageReader(heddleengine.Renderer{}.Passages)`, each with a one-line comment saying what reaches heddle through it.
  A test binary installs none of them, as today, and a missing seam reads as no event and no passage.
- `TestPersistentPreRunE_InstallsSeams` (tier0, in a new `cmd/lym/prerunseams_test.go` with `//go:build tier0`) parses `cmd/lym/main.go` like `TestPersistentPreRunE_NoInfoOrWarnLoggingAheadOfSeedStencils`, finds the pre-run through `findPersistentPreRunELit`, and asserts its body calls `fabricengine.InstallWorktreeSeam`, `shuttleengine.InstallPassageReader`, and `fabricengine.RegisterBoltReplayer` once with a `boardengine` value and once with a `heddleengine` value, failing with the missing call named.
  A behavioural test cannot pin this, because the installs are skipped under `testing.Testing()`.

**ImpactSummary:** Three install calls in the root pre-run's production-only block; test binaries and every command's own flow are unchanged.
