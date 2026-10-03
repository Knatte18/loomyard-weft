# Card 8 — hubforge-pair-and-open

**Create:**
- `plan:internal/hubforge#AddPairWith` -> `func AddPairWith(tb testing.TB, h *Hub, slug string, opts fabricengine.AddOptions) fabricengine.AddResult`
- `plan:internal/hubforge#OpenFabric` -> `func OpenFabric(tb testing.TB, h *Hub) *fabricengine.Fabric`

**Edit:**
- `internal/hubforge#AddPair`
- `hubforge/hub_test.go`
- `hubforge/doc.go`
- `//CONSTRAINTS.md`

**Uses:**
- `internal/hubforge#Hub`

**Intent:**
Give fabricengine tests the two hub builders they now hand-roll (D5, #325 §2), so a `Topology.Add` or `Open` signature change touches hubforge instead of every site.

`AddPairWith` drives `h.Topology.Add(h.Location, slug, opts)` and fatals on error; `AddPair` becomes `AddPairWith` with zero `AddOptions`, so the two cannot drift.
`OpenFabric` resolves `h.PrimeWorktree()` through `lyxcwd.ResolveWorktree` and opens it with `fabricengine.Open`, fataling on either error, which is the pair 21 fabricengine sites repeat today.
Both live in `hub.go` beside `AddPair`.

`hub_test.go` (integration-tagged) gains tests for both: `AddPairWith` with `SkipPush: true` creates the pair without pushing its branch to the warp bare, and `OpenFabric` returns a handle whose warp and weft paths are the hub's prime pair.

`hubforge/doc.go`'s export summary names the new builders.
`CONSTRAINTS.md`'s hubforge Fabric-Fixture Invariant gains the D6 clause, stated as review discipline: no test package wraps `hubforge` in its own fixture type.
Its import rule (no package in `fabriccli`'s dependency set imports `hubforge`) is unchanged.

**ImpactSummary:** AddPair keeps its behaviour by delegating to the new AddPairWith; hubforge gains two exports and one test file grows.
