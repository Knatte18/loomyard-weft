# Card 1 — segmentcolor leaf package

**Create:**
- `internal/segmentcolor/doc.go`
- `internal/segmentcolor/segmentcolor.go`
- `internal/segmentcolor/segmentcolor_test.go`
- `internal/segmentcolor/leaf_enforcement_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`
- `PATTERN.md`
- `pattern/PATTERN-leaf-packages.md`
- `docs/overview.md`

**Intent:**
Add the leaf package D13 names: lyx's own closed vocabulary of loom segments and palette colors, standard library only.

`segmentcolor.go` declares:

- `type Segment string` with one constant per loom segment, whose values are the reed config keys card 2 adds: `Coordinator` (`"coordinator"`), `Discussion` (`"discussion"`), `Plan` (`"plan"`), `Webster` (`"webster"`), `Review` (`"review"`), `Describe` (`"describe"`), `Landing` (`"landing"`), and `Segments() []Segment` returning them in that order.
- `type Color string` with the palette constants `Blue`, `Purple`, `Cyan`, `Green`, `Orange`, `Yellow`, `Pink`, `Red` (values are the lowercase names), and `Palette() []Color` in that order.
- `TmuxColor(c Color) (string, bool)`: blue→`blue`, purple→`magenta`, cyan→`cyan`, green→`green`, orange→`colour208`, yellow→`yellow`, pink→`colour205`, red→`red`; false for anything outside the palette.
- `ParseColor(raw string) (Color, error)`: exact match against the palette; the error names the value and lists the palette, so a caller can show it as the way forward.

No segment's default color lives here: the defaults are reed's template values (card 2), and nothing in Go hard-codes a segment's color.
The package carries no provider name and no provider command (`PATTERN-shuttle-provider-seam`).

`doc.go` carries the package doc: what a segment is (the loom phase a spawned strand belongs to, named by its spawning module on the launch spec), that the palette is lyx's own vocabulary mapped to tmux here and to a provider's names only inside that provider's engine, and the leaf rule.

Tests: `segmentcolor_test.go` is one table over the palette pinning each `TmuxColor` answer plus an unknown color answering false, `ParseColor` rows for every palette name and for a non-palette value whose error names the value and the palette, and a row pinning `Segments()` and `Palette()` as closed sets (their size is the contract).
`leaf_enforcement_test.go` calls `scankit.AssertImportAllowlist(t, "internal/segmentcolor")` with no allowed non-stdlib import, modelled on `internal/standalonestate/leaf_enforcement_test.go`.
`test-budget.yaml` gains an `internal/segmentcolor` row at the package's top-level test count.

`PATTERN.md`'s `PATTERN-leaf-packages` line gains `segmentcolor`, and `pattern/PATTERN-leaf-packages.md` gains an `internal/segmentcolor` section: imports only the standard library.
`docs/overview.md` lists `internal/segmentcolor/` in the repository tree and in the shared-infrastructure leaf list beside the other leaves.

**ImpactSummary:** new package with no callers yet; PATTERN and overview gain one entry each.
