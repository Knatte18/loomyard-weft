# Card 3 — fabric config dirs

**Create:**
- `internal/fabricengine#ConfigDirs`
- `internal/fabricengine#TestConfigDirs`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/configengine#Dirs`

**Intent:**
Add the one constructor every caller holding a `*lyxcwd.Location` uses to tell a layered load its geometry, beside `BoardDir` in `junctionnames.go`.
`ConfigDirs` returns `configengine.Dirs{Anchor: l.AnchorPath(), Board: BoardDir(l.HubPath)}` when that board dir exists as a directory, and `Board` empty otherwise, so a git checkout that is not a hub loads the standalone stack rather than a repo default that cannot exist.
A stat failure other than absence is treated as absence and logged at Warn, since a loader then degrades or refuses on its own base-layer rule.
It lives in fabricengine because that package already declares `BoardDir` and imports both `lyxcwd` and `configengine`; configengine itself stays told (`PATTERN-told-geometry`).

`TestConfigDirs` is table-driven over a temp hub: a location whose hub holds `_board` yields both dirs, and one whose hub lacks it yields `Board` empty with the same anchor.
The budget row for `internal/fabricengine` rises by one.

**ImpactSummary:** additive: one new fabricengine function and its test; only the budget row changes.
