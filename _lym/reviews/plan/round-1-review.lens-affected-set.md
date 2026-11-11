```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens. For each output, key, string, file shape or behaviour a card changes, I found what reads or asserts it and which cards cover it.

**Per change: what reads or asserts it, and which cards cover it**

1. **`boardengine.OwnerRequest` gains `Replace` (card 1).**
   - No reflective seam test or `//lym:seam` marker covers `OwnerRequest` (grep found none).
   - Existing callers leave the field empty, and card 4 is the only new caller.
   - The `OwnerRequest` and `OwnerWrite` field docs are covered by card 1.
2. **New commit subject `board: owner <m> replacing <names>` (card 1).**
   - Readers: `internal/boardengine/ownerset.go:105`, the doc line at `internal/boardengine/doc.go:174`, and the `wantSubject` rows at `ownerset_integration_test.go:184-239`.
   - The old rows keep the `for` subject, since none sets `Replace`.
   - Code, doc and the new test rows are covered by card 1.
3. **Owner-setter takeover clause (card 1).**
   - Readers: `pattern/PATTERN-owner-machine.md:27`, `internal/boardengine/doc.go:183-185`, and the `PATTERN.md:24` line.
   - All covered by card 1.
4. **New module `machine` mounted on the root (card 3).** Readers and pins:
   - `cmd/lym/main.go:135`, the `Available modules` line, which `cmd/lym/clitree_test.go:152` asserts.
   - `cmd/lym/registration_test.go`, which walks the tree and needs no list edit.
   - `cmd/lym/seamsignature_test.go`, both lists.
   - `cmd/lym/sandbox_coverage_test.go` `excludedModules`.
   - `docs/overview.md`, the tree line and module bullet.
   - `pattern/PATTERN-cli-cobra.md` package-naming deviations.
   - All covered by card 3; card 4 updates the overview bullet.
   - No other module list names CLI packages: `hibernatecli` appears outside its package only in `seamsignature_test.go`, `main.go`, `docs/overview.md`, `beamcli` dobby wiring (hibernate-specific) and a benchmark report.
   - `internal/shellguard/rows.go` has no per-operator-verb row, so nothing is forced there.
5. **`machinename.FromThisHost` (card 2).**
   - `TestSoleSource_NoOtherProductionFileReadsTheHostname` and the leaf allowlist still hold, because the hostname read stays in the package.
   - `internal/machinename/doc.go` stays true.
   - Covered by card 2.
6. **`machinename`'s existing way-forward text names `lym machine name <name>` (`internal/machinename/machinename.go:28`).** Cards 3 and 4 make that verb exist, so nothing needs editing.

**Findings**

- None. Every set above is fully covered by the cards that change it.

**Tool calls**

1. Grepped for reflective seam markers on `OwnerRequest`, readers of the `board: owner` commit subject, and the `Available modules` list, because the manifest held no search for them.
2. Grepped for every file naming `hibernatecli`, to find the per-module lists a new CLI joins, because the manifest read only the known pins.
3. Grepped `shellguard` and `contracts` for per-verb rows naming operator verbs, because the manifest did not cover guard tables.
