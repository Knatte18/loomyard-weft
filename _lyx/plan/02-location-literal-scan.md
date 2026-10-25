# Card 2 — No Location literal outside lyxcwd

**Edit:**
- `internal/beamcli#pairLocation` -> `func pairLocation(prime *lyxcwd.Location, slug string) (*lyxcwd.Location, error)`
- `internal/beamcli#dobbyRuns`
- `internal/beamcli#dobbyWatchDirs`
- `internal/pairteardown#Teardown.driverLockBusyError`
- `internal/fabricengine#warpLayoutFor`
- `internal/fabricengine#CloneHub`

**Create:**
- `internal/lyxcwd#TestEnforcement_LocationLiterals`
- `internal/beamcli#TestDobbyRuns_PairAtItsOwnAnchor`

**Uses:**
- `internal/lyxcwd#NamedWorktree`
- `internal/lyxcwd#TestEnforcement`

**Intent:**
D3's scan and the four production `lyxcwd.Location` composite literals it would fail on.
Each literal becomes a `lyxcwd.NamedWorktree(hubPath, name)` call, so the Location carries that worktree's own anchor instead of a copy of the prime's or acting one's:
`pairLocation` returns the error, `dobbyRuns` returns it and `dobbyWatchDirs` skips the slug with a `logger.Warn` naming it, as it skips other unreadable pairs;
`driverLockBusyError` names the lock under the pair's own anchor and returns a `NamedWorktree` error, wrapped naming the pair, in place of the refusal; in production that read never fails there, since its callers reach it only after `driverLockHeldIn`'s `ResolveWorktree` read the same marker;
`warpLayoutFor`'s hub-sibling branch reads the sibling's own anchor, and its doc drops "shares the hub-wide recorded anchor";
`CloneHub`'s post-checkout hook Location comes from `NamedWorktree(hubPath, name)`, which answers `"."` there since no marker exists yet and the hook reads only `WorktreePath()`.

`TestEnforcement_LocationLiterals` (tier0, `enforcement_test.go`, walking through `scankit.Walk` as `TestEnforcement` does) parses every production `.go` file outside `internal/lyxcwd/` and `internal/testkit/` and fails on a composite literal whose type is `lyxcwd.Location`, bare or under `&`; a slice or map literal of `*lyxcwd.Location` is not one.
Its `predicate` subtest pins the matcher on synthetic snippets: `&lyxcwd.Location{}` and `lyxcwd.Location{HubPath: h}` match, `[]*lyxcwd.Location{p}` and `map[string]*lyxcwd.Location{}` do not.
It requires a scan floor through `scankit.RequireFloor`.
A slug-built path joined with another Location's `AnchorRel` stays a review rule.

`TestDobbyRuns_PairAtItsOwnAnchor` (tier1, hubforge, in-package since `dobbyRuns` and `dobbyWatchDirs` are unexported) covers the two `beamcli` callers, since no test drives them today:
a task pair whose records worktree root holds a committed `.lyx-anchor` of `sub` over content moved there (set up by hand with `git mv` and a commit in the pair's records worktree) has its child status path in `dobbyRuns`'s view, and its status directory in `dobbyWatchDirs`' listing, under `<hub>/<slug>/sub/`;
a pair whose root `.lyx-anchor` holds an invalid value (`../out`) makes `dobbyRuns` return an error naming the pair and `ErrInvalidAnchor`, and is left out of `dobbyWatchDirs`' listing while the other pair is listed.

**ImpactSummary:** `pairLocation` gains an error result with its two callers in `beamcli`; the other three sites keep their signatures, and in a one-anchor hub every path they build is unchanged.
