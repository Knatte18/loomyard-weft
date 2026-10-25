# Card 6 — Clone records and adopts the marker on main-weft

**Edit:**
- `internal/fabricengine#CloneHub`
- `internal/fabricengine#probeWeftBinding`
- `internal/fabricengine#warpProbeResult`
- `internal/fabriccli#CloneAndWire`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/codebinding.go`
- `internal/fabricengine#Unwire`
- `internal/fabriccli#Command`
- `internal/fabriccli/doc.go`
- `internal/fabricengine#staleFabricAnchorName`
- `cmd/lyx#uncontainedWriteAllowlist`
- `internal/hubforge#assertRealHub`
- `internal/fabricengine_test#TestCloneHub_AnchorCreatePath`
- `internal/fabricengine_test#TestCloneHub_AnchorRootDefaultPath`
- `internal/fabricengine_test#TestCloneHub_AnchorAdoptPath`
- `internal/fabricengine_test#TestCloneHub_AnchorBearingWeftPassesGuard`
- `internal/fabriccli_test#TestRunCLI_CloneEndToEnd`
- `internal/fabricengine_test#TestRemove_TearsDownNestedJunction`
- `internal/fabricengine_test#seedAnchorMarker`
- `internal/fabricengine_test#TestStatus_DetectsWarpPollutionUnderSubpathAnchor`
- `internal/fabricengine_test#TestUnwire_PreservesRepoWideRecords`
- `internal/fabricengine_test#TestReconcile_RefusesEmptyAnchorMarkerInsteadOfWiringAtTheRoot`
- `internal/creelcli#TestHubDelivery_SkipsAWorktreeThatNoLongerResolves`
- `internal/preflight_test#TestHealthyHub`
- `internal/preflight_test#TestResolveMode`
- `internal/fabricengine_test#TestCloneHub_OldOrderInvocationIsRefused`

**Create:**
- `plan:internal/fabricengine#commitAnchorMarker` -> `func commitAnchorMarker(rec *Mutations, weftPath, anchor, subject string) (string, error)`

**Uses:**
- `internal/fabricengine#RecordsBranchName`
- `internal/fabricengine#ensureWeftLockDirAt`
- `internal/fabricengine_test#TestCloneHub_StaleFabricAnchorHardErrors`

**Intent:**
D4.
`commitAnchorMarker` writes `anchor` plus a newline into `<weftPath>/.lyx-anchor` and commits that one path on the branch checked out at `weftPath`, under the weft write lock (`.weft/weft.write.lock`, through `ensureWeftLockDirAt`), recording `KindFileWritten` and `KindCommitCreated`; it pushes nothing, and it is the one writer of a marker commit outside the D7 move (Shared Decisions).
It lives in a new `internal/fabricengine/anchormarker.go` and writes the file through an `os.Root` opened at `weftPath` (`root.WriteFile`), as `writeLaunchers` writes through one at the hub, so it carries no raw write primitive and needs no entry in `cmd/lyx`'s `uncontainedWriteAllowlist`, whose doc comment admits a new raw write only into a git-owned path or a directory a contained minter made in the same call, and a reconcile's or an adopt's weft is neither.
`CloneHub` no longer writes the marker itself, so the allowlist's `clone.go` entry drops the marker from its reason and keeps the hub scratch directory.

`CloneHub`, after the weft primary is on main-weft (`RecordsBranchName` of the warp branch) and the board exists:
it adopts the marker checked out at the root of the prime's weft worktree when present, validated with `ValidateAnchorRel`;
else it adopts the Bolt legacy marker, and only on this fallback runs the stale `.fabric-anchor` guard;
else, on a fresh bind, it validates the requested subpath against the cloned warp as today and records it with `commitAnchorMarker` on main-weft, writing no Bolt marker.
A `--subpath` that disagrees with the adopted value, from either source, is refused as today.
An adopt writes nothing: a hub whose main-weft is markerless keeps resolving through Bolt until `lyx fabric reconcile` migrates it (card 8).
`CloneAndWire`'s Bolt commit subject drops "record anchor", since the board commit no longer carries one.

`probeWeftBinding` also reads the root of main-weft by name: after the HEAD reads it fetches `refs/heads/` + `RecordsBranchName("main")` into the throwaway probe clone, a missing branch being no error, and sets `AnchorFound` when either `HEAD` or main-weft carries `.lyx-anchor`; `warpProbeResult`'s field docs say so.
So the fresh-bind test (`freshBind`) and the bootstrap guard's evidence count a marker on either branch, and the guard's message names `.lyx-anchor` on `weft:main` or main-weft.

`internal/fabricengine/doc.go`'s anchor section says the marker is recorded per worktree at its weft root, first on main-weft by clone, with the Bolt marker read only as the migration fallback.
The text that records the anchor on `weft:main` now places the marker at each worktree's weft root: `codebinding.go`'s file header; `unwire.go`'s file header, `Unwire`'s doc and the unwire help in `Command`'s `Long` text, each of which still says unwire leaves every marker in place; `CloneAndWire`'s doc, whose `weft:main` commit carries config and no anchor; and `internal/fabriccli/doc.go`'s clone paragraphs, which say the re-clone compares `--subpath` with the main-weft marker first and the Bolt marker as the fallback.
`staleFabricAnchorName`'s doc says clone's guard runs on the Bolt fallback read only.

Tests:
`TestCloneHub_AnchorCreatePath` and `TestCloneHub_AnchorRootDefaultPath` assert the marker is committed at the prime weft's `HEAD` (`git show HEAD:.lyx-anchor` in it) and absent from the board.
`TestCloneHub_AnchorAdoptPath` seeds the marker on the weft bare's main-weft and keeps today's Bolt-seeded case as a second row: each re-clone adopts `backend`, refuses `--subpath frontend` and accepts `--subpath backend`, and the main-weft row writes no Bolt marker.
`TestCloneHub_AnchorBearingWeftPassesGuard` gains a row seeding the marker only on main-weft, which passes the bootstrap guard without `ForceBootstrap`.
`TestCloneHub_StaleFabricAnchorHardErrors` runs unchanged, pinning the guard on the Bolt fallback.
`TestCloneHub_OldOrderInvocationIsRefused` also asserts the reworded guard message names `.lyx-anchor` on `weft:main` or main-weft.
The probe's fetch of main-weft failing for any reason other than a missing branch is a propagated git error, which `refusal-spec.md` exempts as transient, so no row reaches it.
`TestRunCLI_CloneEndToEnd` expects `.lyx-anchor` tracked in the prime's weft worktree and not on the board, and `assertRealHub` checks the prime weft's marker.
The hubforge-based tests that re-anchor a hub by writing the Bolt marker (`TestRemove_TearsDownNestedJunction`, `seedAnchorMarker`, `TestStatus_DetectsWarpPollutionUnderSubpathAnchor`, `TestUnwire_PreservesRepoWideRecords`, `TestReconcile_RefusesEmptyAnchorMarkerInsteadOfWiringAtTheRoot`, `TestHubDelivery_SkipsAWorktreeThatNoLongerResolves`, `TestHealthyHub`) write and commit it at the prime weft's root instead, since the prime's own marker now wins over Bolt; `TestResolveMode`'s stale-marker row also removes the prime weft's marker, since the stale guard applies to the Bolt read only.

**ImpactSummary:** clone's adopt-or-create and its pre-hub probe change, every hubforge hub now carries a main-weft marker, and the eight hubforge tests that re-anchored through Bolt move their write to the prime's weft root.
