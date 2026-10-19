# Card 5 — retired-artifact-sweep

**Edit:**
- `internal/fabricengine#ReconcileResult`
- `internal/fabricengine#Topology.Reconcile`
- `plan:internal/fabricengine#regenerateHubWorkspace`
- `internal/fabriccli#runReconcile`
- `internal/fabriccli#Command`
- `internal/fabriccli_test#TestRunCLI_HubMutationScenario`

**Create:**
- `plan:internal/fabricengine#retiredPortalTarget` -> `func retiredPortalTarget(l *lyxcwd.Location, name string) string`
- `plan:internal/fabricengine#retiredHubWorkspacePath` -> `func retiredHubWorkspacePath(l *lyxcwd.Location, primeName string) string`
- `plan:internal/fabricengine#retiredHubArtifactsPresent` -> `func retiredHubArtifactsPresent(l *lyxcwd.Location) bool`
- `plan:internal/fabricengine#sweepRetiredHubArtifacts` -> `func sweepRetiredHubArtifacts(rec *Mutations, l *lyxcwd.Location) []string`
- `plan:internal/fabricengine_test#TestReconcile_SweepsRetiredHubArtifacts` -> `func TestReconcile_SweepsRetiredHubArtifacts(t *testing.T)`
- `plan:internal/fabricengine#TestSweepRetiredHubArtifacts_UnresolvablePrime` -> `func TestSweepRetiredHubArtifacts_UnresolvablePrime(t *testing.T)`

**Prosa:**
- `pattern/PATTERN-hub-containment.md`
- `tools/sandbox/SANDBOX-FABRIC-SUITE.md`

**Uses:**
- `internal/fabricengine#portalsDirName`
- `internal/fabricengine#launchersDir`
- `internal/fabricengine#PrimeName`
- `internal/fabricengine#removeLink`
- `internal/fabricengine#removePath`
- `internal/fabricengine#removeContainedPath`
- `internal/fabricengine#ownedWiredJunction`
- `internal/fabricengine#ownedUnderGeometryRoot`
- `internal/fabricengine#dirtinessNA`
- `internal/fabricengine#KindLinkRemoved`
- `internal/fabricengine#KindPathRemoved`
- `internal/fslink#RawTarget`
- `internal/fslink#IsLink`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPair`
- `internal/testkit/logcapture#Capture`
- `internal/testkit/envelope#Decode`
- `internal/testkit/locationkit#Location`

**Intent:**
`lyx fabric reconcile` sweeps the retired portal links and the retired workspace file from an existing hub, and every regeneration warns while one remains (D7).

Geometry, kept beside the sweep in `portals.go` with `portalsDirName`:
- `retiredPortalTarget(l, name)` is `<hub>/<name>/<AnchorRel>/_lyx`, the target a fabric portal link named `name` pointed at.
- `retiredHubWorkspacePath(l, prime)` is `<hub>/_launchers/<AnchorRel>/<prime>.code-workspace`.

`sweepRetiredHubArtifacts(rec, l)` removes only through `destroy.go`'s executors and returns the hub-level details:
- It lists `<hub>/_portals/<AnchorRel>/` through an `os.Root` at the hub.
  A component that is a symlink escaping the hub is refused: nothing is removed, and one detail names the refusal.
- Each entry that is a link whose raw target (`fslink.RawTarget`) is `retiredPortalTarget(l, <entry name>)`, whether or not that target still exists, is removed through `removeLink` with `ownedWiredJunction([]string{link}, target)` and `dirtinessNA`.
  `removeLink` records `KindLinkRemoved`.
- Every other entry under `_portals`, at the anchor dir or anywhere else in the tree, is kept, with one detail naming its path.
- The emptied `_portals/<AnchorRel>` chain is then removed deepest first, up to and including `_portals`, each directory through `removeContainedPath` with `recursive` false.
  Each removal records `KindPathRemoved` with detail `single`; a directory still holding a kept entry stays.
- When `PrimeName` resolves, the retired workspace path is inspected with `Lstat`.
  A regular file or a link there is removed through `removePath` with `ownedUnderGeometryRoot(launchersDir(l))` and `dirtinessNA`; a link is removed as a link and its target is untouched.
  A directory there is kept, with a detail naming it.
  When the prime does not resolve, this step is skipped.
- An absent `_portals` or retired file is a no-op that records nothing, so a second run changes nothing.

`ReconcileResult` gains `HubDetails []string` (json `hub_details,omitempty`), filled from the sweep.
`Topology.Reconcile` runs the sweep after its pair loop and before the regeneration card 2 placed there, so its own regeneration logs no retired-artifact Warn for anything the sweep removed.
`runReconcile` emits `hub_details` in its envelope when the list is not empty.

`retiredHubArtifactsPresent(l)` reports whether a fabric-shaped portal link (by the same raw-target rule) or a file or link at the retired workspace path exists.
It reads through an `os.Root` at the hub, like the sweep, so an escaping `_portals` component reads as no artifact, and the sweep's detail reports it instead.
`regenerateHubWorkspace` calls it after resolving the prime and, when it reports true, logs one Warn naming the retired artifacts and `lyx fabric reconcile` as the way to remove them.
A hub whose `_portals` holds only a foreign entry, or whose retired workspace path holds only a directory, logs no Warn.

Tests:
- `TestReconcile_SweepsRetiredHubArtifacts` (integration, external package, one `hubforge.CopyHub` hub per row) plants portal links by hand with `os.Symlink`, as no verb creates them any more, at the root anchor and a nested one.
  It has these rows:
  1. A fabric-shaped link of a live pair, one whose pair is gone (dangling), and a file at the retired workspace path are all removed.
     So are the `_portals` chain and the retired file, and Reconcile's mutation record carries one `KindLinkRemoved` per link and a `KindPathRemoved` for the file and each emptied directory.
     A second Reconcile records none of these and returns no `HubDetails`.
     The row runs Reconcile once from the prime's location and once, on a second hub, from a task pair's location.
  2. A foreign file and a link pointing elsewhere under `_portals/<AnchorRel>/` survive, and on the nested-anchor hub so does a foreign file at `_portals/x`, outside the anchor chain.
     So does `_portals`, and `HubDetails` names each foreign entry.
  3. A directory at the retired workspace path survives, with a detail naming it.
  4. A symlink planted at the retired workspace path is removed as a link while its target file stays.
  5. `_portals` replaced by a symlink to a directory outside the hub holding a fabric-shaped-looking link: nothing outside the hub is removed, and a detail names the refusal.
  6. The retired-artifact Warn:
     - With a fabric-shaped link present, a `RegenerateHubWorkspace` call logs the Warn naming `lyx fabric reconcile`.
     - With only a file at the retired workspace path present, a call logs it too.
     - The log of the Reconcile call that sweeps them holds no retired-artifact Warn, and a further call after it logs none.
     - With only a foreign entry under `_portals`, only a directory at the retired path, or `_portals` as a symlink escaping the hub, no Warn is logged.
     These rows capture the logger with `logcapture.Capture` and are not parallel.
- `TestSweepRetiredHubArtifacts_UnresolvablePrime` (unit, internal package) runs the sweep on a `locationkit` location over a temp hub dir with no git checkout.
  The hub holds a fabric-shaped link under `_portals/<AnchorRel>/` and a file at `_launchers/<AnchorRel>/prime.code-workspace`.
  The link is removed and the file is kept, because the prime cannot be resolved.
- `TestRunCLI_HubMutationScenario` gains a row: `lyx fabric reconcile` on a hub with a foreign file under `_portals/<AnchorRel>/` exits 0 and its envelope's `hub_details` names that file.

Docs in this card:
- The `lyx fabric reconcile` help text in `internal/fabriccli#Command` says reconcile removes fabric's retired portal links and the old `_launchers` workspace file, keeps anything else with a `hub_details` entry, and that every regeneration warns until it has run.
- `pattern/PATTERN-hub-containment.md`'s `_portals` bullet adds that it is swept by `lyx fabric reconcile`.
- `SANDBOX-FABRIC-SUITE.md`'s F17 adds the sweep check: a planted portal link and old workspace file are removed by reconcile, a foreign entry is kept and reported.

**ImpactSummary:** Reconcile gains a hub-level sweep and one new result field emitted as `hub_details`; the regeneration gains one Warn; no other caller of these members changes.
