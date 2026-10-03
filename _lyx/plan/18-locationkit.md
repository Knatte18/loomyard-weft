# Card 18 — locationkit

**Create:**
- `testkit/locationkit/locationkit.go`
- `testkit/locationkit/locationkit_test.go`

**Uses:**
- `testkit/doc.go`

**Intent:**
Provide one shared synthetic `*lyxcwd.Location` builder (#325 §6) in place of the three identical private copies card 19 converts: `anchoringFixture` in `cmd/lyx/constructoranchoring_test.go`, `notransientsFixture` in `cmd/lyx/notransients_test.go` and `newPortalLauncherTestLocation` in `fabricengine/portallauncher_test.go`.

They span two packages, so under D1 the builder is a kit; D2 names no kit for it, and `locationkit` fills that gap (see the overview's Shared Decisions).
`locationkit.go` exports `Location(hubPath, worktreeName, anchorRel string) *lyxcwd.Location`, a synthetic, unresolved location for pure path-arithmetic tests, documented as never touching disk and as distinct from a resolved location (the Cwd Resolution Invariant still governs anything that resolves).
It imports only `internal/lyxcwd`.
Its package doc (D11) says the kit builds synthetic, unresolved locations for path-arithmetic tests, never touches disk, imports only `internal/lyxcwd`, and is no substitute for `lyxcwd.Resolve` where a test needs a real worktree.

`locationkit_test.go` asserts the returned fields and that `WorktreePath()`/`AnchorPath()` join them as expected.
