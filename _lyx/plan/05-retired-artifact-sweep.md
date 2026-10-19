# Card 5 — retired-artifact-sweep

**Create:**
- `plan:internal/fabricengine#sweepRetiredHubArtifacts` -> `func sweepRetiredHubArtifacts(rec *Mutations, l *lyxcwd.Location) []string`
- `plan:internal/fabricengine#retiredHubArtifactsPresent` -> `func retiredHubArtifactsPresent(l *lyxcwd.Location, primeName string) bool`
- `plan:internal/fabricengine#retiredHubWorkspacePath` -> `func retiredHubWorkspacePath(l *lyxcwd.Location, primeName string) string`
- `plan:internal/fabricengine#retiredPortalTarget` -> `func retiredPortalTarget(l *lyxcwd.Location, name string) string`
- `plan:internal/fabricengine_test#TestReconcile_SweepsRetiredHubArtifacts` -> `func TestReconcile_SweepsRetiredHubArtifacts(t *testing.T)`
- `plan:internal/fabricengine#TestSweepRetiredHubArtifacts_UnresolvablePrime` -> `func TestSweepRetiredHubArtifacts_UnresolvablePrime(t *testing.T)`

**Edit:**
- `internal/fabricengine#ReconcileResult`
- `internal/fabricengine#Topology.Reconcile`
- `plan:internal/fabricengine#RegenerateHubWorkspace`
- `internal/fabricengine#ownedUnderGeometryRoot`
- `internal/fabricengine#resolvePathOwnership`
- `internal/fabricengine#portalsDirName`
- `internal/fabricengine/doc.go`
- `internal/fabriccli#runReconcile`
- `internal/fabriccli#Command`
- `internal/fabriccli_test#TestRunCLI_HubMutationScenario`
- `pattern/PATTERN-hub-containment.md`

**Uses:**
- `internal/fabricengine#removeLink`
- `internal/fabricengine#checkPathRequest`
- `internal/fabricengine#removeContainedPath`
- `internal/fabricengine#ownedWiredJunction`
- `internal/fabricengine#dirtinessNA`
- `internal/fabricengine#launchersDir`
- `internal/fabricengine#PrimeName`
- `internal/fabricengine#HubWorkspacePath`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPairWith`
- `internal/testkit/logcapture#Capture`

**Intent:**
`lyx fabric reconcile` sweeps the retired portal links and the retired workspace file from an existing hub, and every regeneration warns while one is left (D7).

Shapes.
`retiredPortalTarget(l, name)` is `<hub>/<name>/<AnchorRel>/_lyx`, the target a fabric portal link named `name` had.
A fabric-shaped portal link is a link directly in `<hub>/_portals/<AnchorRel>/` whose one-hop raw target (`fslink.RawTarget`) equals `retiredPortalTarget` for its own name, whether or not that target still exists, so a portal of a pair already removed still matches.
`retiredHubWorkspacePath(l, primeName)` is `<hub>/_launchers/<AnchorRel>/<primeName>.code-workspace`.

Sweep.
`sweepRetiredHubArtifacts(rec, l)` returns the hub-level details and never fails Reconcile; every removal goes through `destroy.go`'s gate (PATTERN-fabric-destruction-chokepoint) and is appended to `rec` only after it happened (PATTERN-mutation-record).
The `_portals` tree is listed through an `os.Root` at `l.HubPath`, and every removal uses `l.HubPath` as its container, never `_portals` or `_launchers`, so a symlink planted at either container resolves outside the container's root and is refused rather than opened.
1. Each fabric-shaped portal link is removed through `removeLink` with container `l.HubPath`, ownership `ownedWiredJunction([]string{link}, retiredPortalTarget(l, name))` and `dirtinessNA` (a link holds no content), recording `KindLinkRemoved`.
2. The emptied chain `_portals/<AnchorRel>` up to and including `_portals` is removed bottom-up, each level through `checkPathRequest` with container `l.HubPath` and ownership `ownedUnderGeometryRoot(<hub>/_portals)` and then `removeContainedPath(l.HubPath, dir, false)`, the single-entry removal the OS refuses on a non-empty directory, recording `KindPathRemoved` with detail `single`; the walk stops at the first level still holding something, and touches only levels that `Lstat` through the hub root reports as real directories, so a `_portals` planted as a link is never unlinked by it.
3. Every entry left under `_portals` after steps 1 and 2 other than the chain directories themselves (a regular file, a directory, or a link with any other target) is kept, with one detail per entry naming its path, and so keeps its directory.
4. When `PrimeName(l)` resolves, the entry at `retiredHubWorkspacePath` is read with `Lstat` (never followed): a directory is kept with a detail naming it; a regular file or a link goes through `checkPathRequest` with container `l.HubPath` and `ownedUnderGeometryRoot(launchersDir(l))`, then `removeContainedPath(l.HubPath, path, false)`, which unlinks a link as a link and leaves its target untouched, recording `KindPathRemoved` with detail `single`.
   When the prime does not resolve, this step is skipped with a detail naming the error and the retired path is left alone; Reconcile's own worktree listing fails first on the same cause, so the skip is reached only by calling the sweep directly, which `TestSweepRetiredHubArtifacts_UnresolvablePrime` does.
A refusal or failure at any removal, an `_portals` (or a component of the sweep path) planted as a symlink escaping the hub included, leaves the entry in place and becomes a detail carrying the error; nothing outside the hub is touched because both the gate's containment check and the `os.Root` act refuse the escaping component.
An absent `_portals` or retired file is a no-op, so a second run records nothing and returns no details.

Ownership.
`ownedUnderGeometryRoot`'s geometry-root set grows from `_launchers` to `_launchers` and `_portals`: `resolvePathOwnership` admits a resolved root whose base name is `launchersDirName` or `portalsDirName`, and both docs say so.
`resolvePathOwnership`'s wired-junction comment stops citing "a portal wired at a warp `_lyx`" as live wiring, naming it as the retired portal link the sweep matches by its one-hop target.
Bound: the widened kind admits only paths at or below a resolved `<…>/_portals` root, whose only caller is this sweep, and step 2 pairs it with a non-recursive removal, so it can remove an empty directory under `_portals` and nothing that holds content.

Reconcile.
`ReconcileResult` gains `HubDetails []string` with tag `json:"hub_details,omitempty"`, documented as the sweep's hub-level details.
`Topology.Reconcile` runs `sweepRetiredHubArtifacts` after the warp-binding backfill and before its regeneration (Card 2), so its own regeneration logs no retired-artifact Warn for anything the sweep removed.
`fabriccli`'s `runReconcile` puts `r.HubDetails` into the envelope under `hub_details` when non-empty, and the `reconcile` help text says it sweeps `_portals` links and the old `_launchers` workspace file and reports what it kept under `hub_details`.

Warn.
`RegenerateHubWorkspace` logs one Warn naming `lyx fabric reconcile` when `retiredHubArtifactsPresent(l, primeName)` is true: a fabric-shaped portal link exists, or a regular file or link exists at `retiredHubWorkspacePath`.
A foreign entry under `_portals`, or a directory at the retired path, alone does not raise it, and a read error while probing counts as nothing present.

Docs.
`portalsDirName`'s doc and `internal/fabricengine/doc.go` describe the sweep and the Warn.
`pattern/PATTERN-hub-containment.md`'s `_portals` lines change, its workspace bullet being Card 1's: `_board` and `_launchers` are reachable from the hub only; `_launchers` links point hub-inward only; `_portals` is retired, still reserved, and swept by `lyx fabric reconcile`.

Tests.
`TestReconcile_SweepsRetiredHubArtifacts` (integration, package `fabricengine_test`, `CopyHub` per subtest; the log-capturing subtests are serial) plants the retired artifacts by hand, since no verb creates them any more:
- `sweeps`: a pair from `AddPairWith`, a fabric-shaped link for it and a dangling one for a slug with no pair under `_portals/<AnchorRel>`, a foreign regular file under `_portals`, a foreign link under `_portals/<AnchorRel>` named after the pair but pointing at another directory, and a file at `retiredHubWorkspacePath`; a regeneration before `Reconcile` logs the Warn; `Reconcile`, with the log captured across it, removes both fabric-shaped links and the file, records a `KindLinkRemoved` for each link and a `KindPathRemoved` for the file, keeps the foreign file and the foreign link and their directories, returns a `HubDetails` entry naming each, and logs no retired-artifact Warn during its own regeneration; a regeneration after logs no Warn; a second `Reconcile` records no removal under `_portals` or `_launchers`.
- `empty chain`: on a `CopyHub` anchored at `backend`, fabric-shaped links only under `_portals/backend`; after `Reconcile`, `_portals/backend` and `_portals` are gone and each removed level is recorded `KindPathRemoved`; a second hub of the same shape that also holds a foreign file at `_portals/other` loses `_portals/backend` and keeps `_portals` with a detail for the file, so the walk stops at the first non-empty level.
- `foreign only`: `_portals` holding only a foreign file and a directory at the retired workspace path; a regeneration logs no Warn, and `Reconcile` keeps both with a detail each.
- `unreadable _portals`: `_portals` holding a fabric-shaped link, then `chmod 000` on `_portals` (restored in cleanup); a regeneration logs no Warn, since a probe read error counts as nothing present.
- `removal failure`: a file at `retiredHubWorkspacePath` with its `_launchers` directory made read-only (`chmod 0555`, restored in cleanup); `Reconcile` leaves the file in place and returns a `HubDetails` entry carrying the removal error, reaching the branch where a failed removal becomes a detail.
- `symlinked retired file`: a symlink to a file outside the hub planted at `retiredHubWorkspacePath`; a regeneration logs the Warn, `Reconcile` removes the link and the outside file keeps its bytes.
- `escaping _portals`: `<hub>/_portals` planted as a symlink to an outside directory holding a link shaped like a portal; `Reconcile` returns a detail, and the outside directory and its entry are untouched.
`TestSweepRetiredHubArtifacts_UnresolvablePrime` (integration, package `fabricengine`, since the sweep is unexported) calls `sweepRetiredHubArtifacts` with a Location whose worktree is not a git checkout and a file at `retiredHubWorkspacePath`, and asserts a detail naming the prime-resolution error with the file untouched and nothing recorded.
`TestRunCLI_HubMutationScenario` gains a subtest that plants a foreign file under `_portals` and asserts `fabric reconcile`'s envelope carries `hub_details` naming it; with the file removed, a second `fabric reconcile`'s envelope carries no `hub_details` key.

**ImpactSummary:** Reconcile gains one hub-level step and one result field that only runReconcile reads; the launcher-only geometry-root ownership kind admits `_portals`, used by this sweep alone; every regeneration gains one read-only probe.

**Commit:** 5: fabric reconcile sweeps retired portal links and the old workspace file
