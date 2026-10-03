# Card 18 — locationkit

**Create:**
- `testkit/locationkit/locationkit.go`
- `testkit/locationkit/locationkit_test.go`

**Edit:**
- `//cmd/lyx/constructoranchoring_test.go`
- `//cmd/lyx/notransients_test.go`
- `fabricengine/portallauncher_test.go`

**Uses:**
- `testkit/doc.go`

**Intent:**
Replace the three identical synthetic `*lyxcwd.Location` builders with one shared kit (#325 §6): `anchoringFixture` in `cmd/lyx/constructoranchoring_test.go`, `notransientsFixture` in `cmd/lyx/notransients_test.go` and `newPortalLauncherTestLocation` in `fabricengine/portallauncher_test.go`.

They span two packages, so under D1 the builder is a kit; D2 names no kit for it, and `locationkit` fills that gap (see the overview's Shared Decisions).
`locationkit.go` exports `Location(hubPath, worktreeName, anchorRel string) *lyxcwd.Location`, a synthetic, unresolved location for pure path-arithmetic tests, documented as never touching disk and as distinct from a resolved location (the Cwd Resolution Invariant still governs anything that resolves).
It imports only `internal/lyxcwd`.
`locationkit_test.go` asserts the returned fields and that `WorktreePath()`/`AnchorPath()` join them as expected.

The three callers use it and their private builders go; `newPortalLauncherTestLocation` derived the worktree name with `filepath.Base(worktreeRoot)`, which its callers now do inline.
Apply D10 to every touched test.

**ImpactSummary:** Three untagged test files in cmd/lyx and fabricengine drop their private builders for one kit call; no assertion changes.
