# Card 1 — lymcwd-one-repository

**Create:**
- `plan:internal/lymcwd#OneRepository` -> `func OneRepository(worktreeRoot string) bool`
- `plan:internal/lymcwd_test#TestResolveWithin_OneRepository` -> `func TestResolveWithin_OneRepository(t *testing.T)`

**Edit:**
- `internal/lymcwd#readRecordedAnchor`
- `internal/lymcwd#ResolveWithin`
- `internal/lymcwd#ownerOfRecordsSibling`
- `internal/lymcwd#TestReadRecordedAnchor_MarkerSemantics`

**Intent:** D1 and the `lymcwd` half of D3: the layout is read from the anchor marker at the worktree's own root.

`OneRepository(worktreeRoot)` reports whether `<worktreeRoot>/.lym-anchor` (`AnchorFileName`) is a regular file and the root is neither a records sibling nor the Bolt checkout.
A records sibling is a root whose name ends in `weftname.Suffix` with a directory of the trimmed name beside it, the test `ownerOfRecordsSibling` makes; the Bolt checkout is a root named `BoltDirName`.
Both carry a marker in a two-repository hub, so the exclusion keeps them out.
It reads the working tree only and spawns no git, so `lymcwd`'s import set stays the standard library, `internal/dotgit`, `internal/lymdirs` and `internal/weftname`.
Any stat error, absence included, answers false.
It is the one-repository predicate the part exports.

`readRecordedAnchor` reads `<hub>/<worktreeName>/.lym-anchor` first when `OneRepository(<hub>/<worktreeName>)` holds.
A present root marker is the worktree's anchor: an empty or whitespace-only one is `"."`,
an invalid value is an error naming the file as the records-sibling marker's error does today, and an unreadable one is an error naming the file.
Otherwise the read falls through to the records sibling's marker and then to the Bolt checkout's, as today.
So a one-repository worktree reads its anchor with `_bolt` missing, and a two-repository worktree with its sibling missing still falls back to the Bolt marker.
The doc comment states the new first read.

`ResolveWithin` in a one-repository worktree (`OneRepository(workTreeRoot)`) accepts a cwd at or below the anchored directory only and never looks for a records sibling.
A cwd in a two-repository records sibling still resolves to its owner, since `OneRepository` reads false there.
`ownerOfRecordsSibling` answers "not a records sibling" when the trimmed owner's root satisfies `OneRepository`,
so a checkout named with `weftname.Suffix` beside a one-repository worktree resolves as itself rather than as that worktree's sibling.

Tests:
- `TestReadRecordedAnchor_MarkerSemantics` (tier0) gains a `rootValue` column, written at `<hub>/<worktreeName>/.lym-anchor`,
  and a `wantOneRepository` map from root (`worktree`, `sibling`, `bolt`) to the answer `OneRepository` gives for that root, asserted on every row.
  New rows: `RootMarker_WinsOverSiblingAndBoard` (root `backend`, own `frontend`, board `other`, wants `backend`; worktree true, sibling false, bolt false);
  `RootMarkerWithBoltMissing` (root `backend` only, wants `backend`, worktree true);
  `EmptyRootMarker_IsRootAnchor` (root `"  \n"`, board `backend`, wants `.`);
  `RootMarkerInvalid_IsErrorNamingFile` (root `../out`, wants `ErrInvalidAnchor` naming the root marker's path, which the error-file check reads from a new `wantErrAtRoot` flag).
  Every existing row creates the owner directory `<hub>/<worktreeName>` and asserts worktree false, sibling false and bolt false,
  so the rows with an own or a board marker pin that a records sibling and the Bolt checkout carrying a marker are no one-repository worktree.
- `TestResolveWithin_OneRepository` (tier1, `gitkit.CopyRepo`) commits `.lym-anchor` holding `backend` at the checkout root.
  Rows: `AnchoredDirectory` resolves to `ResolveWorktree(root)`, pinning the root-marker read through the resolver;
  `SuffixedCheckoutBesideIt` adds a linked worktree at `<root>` plus `weftname.Suffix` on a branch made from the commit before the marker commit, so its root holds no marker,
  and resolves a cwd in it to that checkout's own name with anchor `.`.
- `TestResolveWithin` (tier1) keeps its rows unchanged; its `SiblingRecordsDirectory` and `SiblingRoot` rows pin that a two-repository sibling carrying a marker still resolves to its owner.

**ImpactSummary:** Three lymcwd members change; every resolver caller reads the same anchor for a two-repository worktree, sibling and Bolt checkout, and only a code worktree with a root marker reads differently.
