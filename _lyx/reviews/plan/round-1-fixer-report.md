# Plan-Burler round 1 — fixer report

Every finding was validated against the code and fixed; none disputed, none deferred.
`lyx loom validate-plan` returns `ok: true` after the fixes, with informational `create-new-unit` notes only.

## Fixed

### F1 — edit-directive cards miss tests that hand-build stencil text or values (MEDIUM)

Confirmed: `TestBouncer_MarkerCompleteness_BothTemplates` fills the real seed and judge templates from hand-built maps (`internal/shedadapters/bouncer_seed_test.go:394`, `:427`), and the conflict, describe and reflection tests overwrite their seeded stencil with marker-less fixtures.
Following the review's grep, one more case turned up that the review did not name: `internal/shedrecipe/entries_bouncer_test.go` builds its stencils dir by hand (`fixture_test.go` makes an empty `stencils` dir and `minimalBouncerConfig` writes only `bouncer-rubric` and `parent-directive-none`), so once the Bouncer renderers read `edit-directive`, every Bouncer entry test reaching the seed spawn or judge call would fail on a missing stencil.

- Card 18 (formerly 17) lists `internal/shedrecipe/entries_bouncer_test.go`; its Tests name the two hand-built maps in `TestBouncer_MarkerCompleteness_BothTemplates` gaining an `editdirective.MarkerName` value, and `minimalBouncerConfig` writing an `edit-directive` stencil.
- Card 20 (formerly 19) corrects its "against seeded stencils" claim: the three fixtures `conflictStencilFixture`, `describeStencilFixture` and `reflectionStencilFixture` gain a `{{.edit_directive}}` line, and `landingshed/finalize_integration_test.go`'s `finalizeConflictStencilFixture`, which its comment says mirrors mergeresolve's, gains it too and is listed under `**Edit:**`.
- The other renderer cards' tests were checked: webster, treadle, burler, loomengine and the loomcli driver test seed through `stencilkit`, and their hand-built maps were already named.

### F2 — the pair call compares unnormalized worktree paths (MEDIUM)

Confirmed: `parseWorktreePorcelain` stores git's path as printed, while `destroy.go:697` and `remove.go:826` compare `filepath.Clean(filepath.FromSlash(entry.Path))`.

- Card 5 states `CodeWorktree.Path` is normalized that way, so it equals `WorktreePath` for a pair, adds `internal/fabricengine#WorktreePath` to `**Uses:**`, and its `CodeWorktrees` test asserts each pair's `Path` equals `WorktreePath` for its slug.
- Card 7 (formerly 6) matches `filepath.Clean(opts.Pair)` against the listing, and its `Options{Pair: path}` test row builds `path` the way `lyx batten run` does.

### F3 — card 11's fabricengine HeadContains test repeats card 13's rows (LOW)

Card 12 (formerly 11) drops its `warpforward_integration_test.go` test and that file from `**Edit:**`; its Tests say card 14's `StencilSource` table, the only caller, drives `fabricengine.HeadContains`.

### F4 — card 8's new loomcli test file and its duplicate error row (LOW)

Card 9 (formerly 8) no longer creates `startreconcile_integration_test.go`.
`smoke_prebootstrap_integration_test.go` moves to `**Edit:**` and gains one test on a `newWiredPairFixture` pair: `loom status` leaves the retired key and writes no stamp, and `loom resume` removes it, leaves the records worktree clean and writes the stamp.
The `start` gate is pinned by `TestLoomFailedReedUpRefuses`'s `requireStartRefusal` also asserting the stamp exists after the refused start.
The second hub's unparseable-`loom.yaml` row is dropped, as card 7's `ensure_integration_test.go` and card 9's `refusal_integration_test.go` step already cover it.

### F5 — configengine.ScratchDir rides in the hubreconcile card (LOW)

A new card 6, `config-scratch-dir`, creates `configengine.ScratchDir` with its doc comment as the never-tracked mirror of `ConfigDir`.
It also has `StagingFile` join onto `ScratchDir` instead of repeating the segments, with an unchanged result, and adds a `ScratchDir` row to `config_test.go`'s existing `TestConfigPaths` table.
Card 7 moves `ScratchDir` to `**Uses:**`, drops its paragraph, names `configengine.ScratchDir` in `StampPath`/`LockPath`, and its ImpactSummary no longer mentions configengine.

Inserting the card renumbered former cards 6–20 to 7–21: the files were moved with `mv`, and each heading, `**Commit:**` prefix, Card Index row, Shared Decision `Applies to:` list and in-text "card N" reference was updated.

### F6 — decodeLeafValues keeps a parameter it no longer uses (LOW)

Card 3's `decodeLeafValues` glyph carries the re-sign arrow `func decodeLeafValues(data []byte) (map[string]any, error)`, and its Intent says the parameter and the two call-site arguments in `retireHubWideCopy` go.

## Disputed

None.

## Deferred

None.
